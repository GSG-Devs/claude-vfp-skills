---
name: debug-helper
description: Debugging guide for Visual FoxPro applications. Use when troubleshooting errors, analyzing logs, investigating SQL blocking issues, or diagnosing runtime problems.
allowed-tools: Read, Grep, Glob, Bash
---

# VFP Debug Helper

## Common Errors and Solutions

### "Conversion failed when converting date and/or time"

**Cause:** Using `Datetime()` for a DATE field (prefix `d*`)

```foxpro
*-- WRONG
m.dbirthdate = Datetime()  && DATE field expects Date()!

*-- CORRECT
m.dbirthdate = Date()      && DATE field
m.tcreated = Datetime()    && DATETIME field
```

### "Unrecognized command verb"

**Cause:** Using `ELSEIF` which doesn't exist in VFP

```foxpro
*-- WRONG - VFP has no ELSEIF
IF x > 10
    y = 1
ELSEIF x > 5   && ERROR!
    y = 2
ENDIF

*-- CORRECT - Use DO CASE
DO CASE
CASE x > 10
    y = 1
CASE x > 5
    y = 2
ENDCASE
```

### "Operator/operand type mismatch"

**Common causes:**

1. **Comparing NULL values:**
```foxpro
*-- WRONG
IF ddeadline < DATE()  && Fails if NULL

*-- CORRECT
IF VARTYPE(ddeadline) = 'D' AND ddeadline < DATE()
```

2. **Using ! on non-logical:**
```foxpro
*-- WRONG - sql_exec returns numeric, not logical
IF !sql_exec(lcSQL)

*-- CORRECT
IF sql_exec(lcSQL) < 0
```

### Error 1492 "No key columns"

**Cause:** TABLEUPDATE() can't identify which record to update

**Solution:** Set cursor properties correctly:

```foxpro
=CURSORSETPROP("KeyFieldList", "idclient", "qryClients")
=CURSORSETPROP("UpdateNameList", "idclient dbo.clients.idclient, cname dbo.clients.cname", "qryClients")
=CURSORSETPROP("Tables", "dbo.clients", "qryClients")
=CURSORSETPROP("SendUpdates", .T., "qryClients")
```

### "Lock request time out period exceeded"

**Cause:** Another session is blocking the table/row

**Diagnosis:**
```sql
-- Find blocking sessions
SELECT
    r.session_id,
    r.blocking_session_id,
    r.wait_type,
    r.wait_time,
    s.login_name,
    s.host_name,
    s.program_name
FROM sys.dm_exec_requests r
JOIN sys.dm_exec_sessions s ON r.session_id = s.session_id
WHERE r.blocking_session_id > 0
```

**Solutions:**
1. Use ROWLOCK hint: `UPDATE table WITH (ROWLOCK) SET ...`
2. Use single T-SQL batch instead of multiple calls
3. Keep transactions short
4. Implement connection cleanup for abandoned sessions

### "Variable not found"

**Common causes:**

1. **Missing m. prefix with SCATTER:**
```foxpro
SCATTER MEMVAR
*-- WRONG
? cname

*-- CORRECT
? m.cname
```

2. **Scope issue:**
```foxpro
*-- LOCAL variable not visible in called function
LOCAL lcValue
lcValue = "test"
=SomeFunction()  && Can't see lcValue

*-- Solution: pass as parameter or use PRIVATE
```

### "File is in use"

**Cause:** Table or file already open

```foxpro
*-- Check if already open
IF USED("mytable")
    SELECT mytable
ELSE
    USE mytable
ENDIF

*-- Or use alias safely
USE mytable IN 0 ALIAS newalias
```

## Debugging Techniques

### Check Variable Type

```foxpro
? VARTYPE(myvar)
&& Returns: C=Char, N=Numeric, D=Date, T=DateTime, L=Logical, O=Object, U=Undefined, X=NULL
```

### List Open Tables/Cursors

```foxpro
LOCAL laUsed[1], i
=AUSED(laUsed)
FOR i = 1 TO ALEN(laUsed, 1)
    ? laUsed[i, 1], laUsed[i, 2]  && Alias, WorkArea
ENDFOR
```

### Check Cursor Properties

```foxpro
? CURSORGETPROP("Buffering", "mycursor")
? CURSORGETPROP("KeyFieldList", "mycursor")
? CURSORGETPROP("UpdateNameList", "mycursor")
? CURSORGETPROP("Tables", "mycursor")
? CURSORGETPROP("SendUpdates", "mycursor")
```

### List Open Forms

```foxpro
LOCAL i
FOR i = 1 TO _SCREEN.FormCount
    ? _SCREEN.Forms[i].Name, _SCREEN.Forms[i].Caption
ENDFOR
```

### Error Object Details

```foxpro
TRY
    *-- Code that might fail
CATCH TO loEx
    ? "Error:", loEx.ErrorNo
    ? "Message:", loEx.Message
    ? "Procedure:", loEx.Procedure
    ? "Line:", loEx.LineNo
    ? "LineContents:", loEx.LineContents
    ? "Details:", loEx.Details
ENDTRY
```

## SQL Session Debugging

### Check Current SPID

```foxpro
sql_exec_safe("SELECT @@SPID AS spid", "qrySpid")
? "Current SPID:", qrySpid.spid
```

### Find Active Sessions

```sql
SELECT
    session_id,
    login_name,
    host_name,
    program_name,
    status,
    open_transaction_count,
    last_request_start_time
FROM sys.dm_exec_sessions
WHERE program_name LIKE '%YourAppName%'
ORDER BY last_request_start_time DESC
```

### Find Blocking

```sql
SELECT
    t1.session_id AS blocked_session,
    t1.blocking_session_id AS blocking_session,
    t1.wait_type,
    t1.wait_time / 1000.0 AS wait_seconds,
    DB_NAME(t1.database_id) AS database_name
FROM sys.dm_exec_requests t1
WHERE t1.blocking_session_id > 0
```

### Kill Stuck Session (Use Carefully!)

```sql
-- First verify it's safe to kill
SELECT * FROM sys.dm_exec_sessions WHERE session_id = 123

-- Kill if safe
KILL 123
```

## Logging Best Practices

### Simple Log Function

```foxpro
FUNCTION log_error(tcMessage)
    LOCAL lcFile, lcLine
    lcFile = "Logs\" + DTOS(DATE()) + "_error.log"
    lcLine = TRANSFORM(DATETIME()) + " | " + tcMessage
    =STRTOFILE(lcLine + CHR(13)+CHR(10), lcFile, .T.)
ENDFUNC
```

### Log with Context

```foxpro
FUNCTION log_debug(tcMessage, tcProcedure)
    LOCAL lcFile, lcLine
    lcFile = "Logs\" + DTOS(DATE()) + "_debug.log"
    lcLine = TRANSFORM(DATETIME()) + " | " + ;
        IIF(EMPTY(tcProcedure), "", "[" + tcProcedure + "] ") + tcMessage
    =STRTOFILE(lcLine + CHR(13)+CHR(10), lcFile, .T.)
ENDFUNC
```

### Search Logs

```bash
# Find errors in log files
grep -r "ERROR" Logs/

# Find specific error
grep -r "Conversion failed" Logs/

# Recent entries
tail -50 Logs/20240115_error.log
```

## Performance Debugging

### Check Index Usage

```sql
-- Show missing indexes
SELECT
    OBJECT_NAME(d.object_id) AS TableName,
    d.equality_columns,
    d.inequality_columns,
    d.included_columns,
    s.user_seeks,
    s.user_scans
FROM sys.dm_db_missing_index_details d
JOIN sys.dm_db_missing_index_groups g ON d.index_handle = g.index_handle
JOIN sys.dm_db_missing_index_group_stats s ON g.index_group_handle = s.group_handle
WHERE d.database_id = DB_ID()
ORDER BY s.user_seeks DESC
```

### Identify Slow Queries

```sql
SELECT TOP 10
    SUBSTRING(t.text, (s.statement_start_offset/2)+1,
        ((CASE s.statement_end_offset
            WHEN -1 THEN DATALENGTH(t.text)
            ELSE s.statement_end_offset
        END - s.statement_start_offset)/2) + 1) AS query_text,
    s.execution_count,
    s.total_elapsed_time / 1000000.0 AS total_seconds,
    s.total_elapsed_time / s.execution_count / 1000.0 AS avg_ms
FROM sys.dm_exec_query_stats s
CROSS APPLY sys.dm_exec_sql_text(s.sql_handle) t
ORDER BY s.total_elapsed_time DESC
```

## Quick Diagnostic Checklist

- [ ] Check error number and message
- [ ] Verify variable types with VARTYPE()
- [ ] Check if tables/cursors are open with USED()
- [ ] Verify cursor buffering properties
- [ ] Check SQL connection with SQLGETPROP()
- [ ] Look for blocking sessions
- [ ] Review recent log entries
- [ ] Test with simplified code

## Resources

- [VFPX Community](https://vfpx.github.io/)
- [VFP Error Messages](https://hackfox.github.io/section4/s4c9.html)
- [SQL Server DMVs](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/)
