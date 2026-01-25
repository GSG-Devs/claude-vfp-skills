---
name: sql-helper
description: SQL Server integration patterns for Visual FoxPro applications. Use when writing SQL queries, working with SQL Pass-Through (SPT), troubleshooting database operations, or optimizing SQL Server calls.
allowed-tools: Read, Grep, Glob
---

# SQL Helper for VFP + SQL Server

## Core Rules

### 1. Use Safe SQL Execution

Always wrap SQL execution with error handling and retry logic:

```foxpro
*-- CORRECT - With error handling
FUNCTION sql_exec_safe(tcSQL, tcCursor)
    LOCAL lnResult, lnRetry
    lnRetry = 0
    DO WHILE lnRetry < 3
        lnResult = SQLEXEC(gl_nSQLHandle, tcSQL, tcCursor)
        IF lnResult >= 0
            RETURN lnResult
        ENDIF
        lnRetry = lnRetry + 1
        =INKEY(0.5)  && Wait before retry
    ENDDO
    log_error("SQL failed after 3 retries: " + tcSQL)
    RETURN -1
ENDFUNC

*-- WRONG - Raw SQLEXEC without handling
SQLEXEC(gl_nSQLHandle, "SELECT * FROM clients")  && No error handling!
```

### 2. Date vs Datetime (CRITICAL!)

| Prefix | SQL Type | VFP Function | Example |
|--------|----------|--------------|---------|
| `d*` | DATE | `Date()` | dbirthdate, ddeadline |
| `t*` | DATETIME | `Datetime()` | tlogon, tcreated |

```foxpro
*-- CORRECT
m.dmodified = Date()      && For DATE field
m.tlogon = Datetime()     && For DATETIME field

*-- WRONG - Will cause "Conversion failed"
m.dmodified = Datetime()  && ERROR! DATE field expects Date()
```

### 3. Single T-SQL Batch for Multi-Step Operations

```foxpro
*-- CORRECT - One batch, one roundtrip
lcSQL = "SET NOCOUNT ON; " + ;
    "DECLARE @newid INT; " + ;
    "BEGIN TRY " + ;
    "  BEGIN TRANSACTION; " + ;
    "  INSERT INTO dbo.orders (cclient) VALUES ('ABC'); " + ;
    "  SET @newid = SCOPE_IDENTITY(); " + ;
    "  INSERT INTO dbo.orderitems (idorder) VALUES (@newid); " + ;
    "  COMMIT TRANSACTION; " + ;
    "  SELECT @newid AS newid; " + ;
    "END TRY " + ;
    "BEGIN CATCH " + ;
    "  IF @@TRANCOUNT > 0 ROLLBACK; " + ;
    "  SELECT -1 AS newid, ERROR_MESSAGE() AS errmsg; " + ;
    "END CATCH"
sql_exec_safe(lcSQL, "result")

*-- WRONG - Multiple roundtrips, risk of blocking
sql_begin_transaction()
sql_exec_safe("INSERT INTO dbo.orders ...")    && Roundtrip 1
sql_exec_safe("INSERT INTO dbo.orderitems ...") && Roundtrip 2 - may block!
sql_commit_transaction()                        && Roundtrip 3
```

### 4. SET NOCOUNT ON - Required!

```foxpro
*-- CORRECT - NOCOUNT ensures you get the SELECT result
lcSQL = "SET NOCOUNT ON; UPDATE ...; INSERT ...; SELECT @id AS newid"

*-- WRONG - Without NOCOUNT, UPDATE returns empty result set
lcSQL = "UPDATE ...; INSERT ...; SELECT @id AS newid"  && Result is empty!
```

### 5. ROWLOCK for Multi-User

```foxpro
*-- CORRECT - Lock only the row, not the table
sql_exec_safe("UPDATE dbo.sessions WITH (ROWLOCK) SET lactive = 0 WHERE idsession = 123")

*-- WRONG - May lock entire table
sql_exec_safe("UPDATE dbo.sessions SET lactive = 0 WHERE idsession = 123")
```

### 6. NULL Handling in Queries

```foxpro
*-- CORRECT - Use ISNULL or IS NULL
sql_exec_safe("SELECT * FROM orders WHERE ISNULL(dshipped, '1900-01-01') < GETDATE()")

*-- Also correct - IS NULL check
sql_exec_safe("SELECT * FROM orders WHERE dshipped IS NULL OR dshipped < GETDATE()")

*-- WRONG - NULL comparison always returns false
sql_exec_safe("SELECT * FROM orders WHERE dshipped < GETDATE()")  && Misses NULLs!
```

### 7. Report Date Fields - CAST Required

ODBC returns SQL DATE as STRING in VFP. For reports, cast to DATETIME:

```foxpro
*-- CORRECT - For VFP reports
sql_exec_safe("SELECT CAST(dbirthdate AS DATETIME) AS dbirthdate FROM dbo.persons", "rpt")

*-- WRONG - ODBC returns DATE as STRING, report formatting fails
sql_exec_safe("SELECT dbirthdate FROM dbo.persons", "rpt")
```

### 8. INSERT with IDENTITY Return (Multi-User Safe)

```foxpro
*-- CORRECT - SCOPE_IDENTITY is session-safe
FUNCTION sql_insert_get_identity(tcTable, tcFields, tcValues)
    LOCAL lcSQL
    lcSQL = "SET NOCOUNT ON; " + ;
        "INSERT INTO dbo." + tcTable + " (" + tcFields + ") VALUES (" + tcValues + "); " + ;
        "SELECT SCOPE_IDENTITY() AS newid"
    sql_exec_safe(lcSQL, "qryId")
    RETURN qryId.newid
ENDFUNC

*-- WRONG - @@IDENTITY may return ID from trigger insert!
sql_exec_safe("INSERT INTO dbo.orders ...")
sql_exec_safe("SELECT @@IDENTITY AS id", "result")  && May be wrong ID!
```

### 9. sql_exec Return Value is Numeric

```foxpro
*-- CORRECT
IF sql_exec_safe(lcSql, "result") < 0
    log_error("Query failed")
ENDIF

*-- WRONG - Can't use ! operator on numeric
IF !sql_exec_safe(lcSql)  && ERROR! "!" expects logical
```

### 10. String Escaping

```foxpro
*-- CORRECT - Escape single quotes
FUNCTION sql_quote(tcValue)
    RETURN STRTRAN(tcValue, "'", "''")
ENDFUNC

lcSQL = "INSERT INTO notes (ctext) VALUES ('" + sql_quote(m.ctext) + "')"

*-- WRONG - SQL injection risk
lcSQL = "INSERT INTO notes (ctext) VALUES ('" + m.ctext + "')"  && O'Brien crashes!
```

### 11. Date Formatting for SQL

```foxpro
*-- CORRECT - ISO format works universally
FUNCTION sql_format_date(tdDate)
    RETURN "'" + TRANSFORM(YEAR(tdDate)) + "-" + ;
        PADL(MONTH(tdDate), 2, "0") + "-" + ;
        PADL(DAY(tdDate), 2, "0") + "'"
ENDFUNC

*-- Usage
lcSQL = "SELECT * FROM orders WHERE dorder >= " + sql_format_date(DATE())

*-- WRONG - Locale-dependent, may fail
lcSQL = "SELECT * FROM orders WHERE dorder >= '" + DTOC(DATE()) + "'"
```

## Buffered Cursor Pattern

```foxpro
*-- Open with buffering
FUNCTION foloseste(tcTable, tcCursor, tlReadWrite)
    LOCAL lcSQL
    lcSQL = "SELECT * FROM dbo." + tcTable
    IF sql_exec_safe(lcSQL, tcCursor) < 0
        RETURN .F.
    ENDIF

    IF tlReadWrite
        =CURSORSETPROP("Buffering", 5, tcCursor)  && Optimistic table buffering
        =CURSORSETPROP("Tables", "dbo." + tcTable, tcCursor)
        =CURSORSETPROP("KeyFieldList", "id" + tcTable, tcCursor)
        =CURSORSETPROP("UpdatableFieldList", "*", tcCursor)
        =CURSORSETPROP("UpdateNameList", ..., tcCursor)
        =CURSORSETPROP("SendUpdates", .T., tcCursor)
    ENDIF
    RETURN .T.
ENDFUNC

*-- Save changes
FUNCTION sql_tableupdate(tcCursor)
    IF !TABLEUPDATE(.T., .F., tcCursor)
        =TABLEREVERT(.T., tcCursor)
        RETURN .F.
    ENDIF
    RETURN .T.
ENDFUNC

*-- Close (auto-saves if dirty)
FUNCTION inchide(tcCursor)
    IF USED(tcCursor)
        IF CURSORGETPROP("Buffering", tcCursor) > 1
            =sql_tableupdate(tcCursor)
        ENDIF
        USE IN (tcCursor)
    ENDIF
ENDFUNC
```

## Quick Reference

| Function | Purpose | Example |
|----------|---------|---------|
| `SQLCONNECT()` | Connect to SQL Server | `gl_nHandle = SQLCONNECT("DSN")` |
| `SQLEXEC()` | Execute SQL | `SQLEXEC(handle, sql, cursor)` |
| `SQLDISCONNECT()` | Close connection | `SQLDISCONNECT(handle)` |
| `SQLSETPROP()` | Set connection property | `SQLSETPROP(handle, "QueryTimeOut", 30)` |
| `SQLGETPROP()` | Get connection property | `SQLGETPROP(handle, "ConnectString")` |
| `CURSORSETPROP()` | Set cursor property | `CURSORSETPROP("Buffering", 5, cursor)` |
| `TABLEUPDATE()` | Save buffered changes | `TABLEUPDATE(.T., .F., cursor)` |
| `TABLEREVERT()` | Discard changes | `TABLEREVERT(.T., cursor)` |

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Conversion failed date/time" | Datetime() for DATE field | Use Date() for d* fields |
| "Lock request time out" | Another session blocking | Use ROWLOCK, single batch |
| Error 1492 "No key columns" | Missing KeyFieldList | Set primary key in cursor props |
| Empty result after UPDATE | Missing SET NOCOUNT ON | Add SET NOCOUNT ON at start |

## Resources

- [VFPX Community](https://vfpx.github.io/)
- [SQL Server Documentation](https://docs.microsoft.com/en-us/sql/)
