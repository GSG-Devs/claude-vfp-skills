---
name: vfp-conventions
description: Visual FoxPro coding conventions and patterns. Use when writing or reviewing VFP code, understanding naming conventions, variable prefixes, or VFP-specific patterns.
allowed-tools: Read, Grep, Glob
---

# VFP Coding Conventions

## Naming Conventions

### Database Field Prefixes (CRITICAL!)

| Prefix | SQL Type | VFP Type | Example |
|--------|----------|----------|---------|
| `id*` | INT IDENTITY/INT | Integer | idperson, idclient |
| `d*` | **DATE** | Date | dbirthdate, ddeadline |
| `t*` | DATETIME | DateTime | tlogon, tcreated |
| `n*` | NUMERIC | Numeric | npercent, nprice |
| `c*` | VARCHAR/CHAR | Character | cname, ccode |
| `l*` | BIT | Logical | lactive, lbillable |

### Variable Prefixes

| Prefix | Scope | Example |
|--------|-------|---------|
| `gl_*` | Global (PUBLIC) | gl_nSQLHandle, gl_cUser |
| `gll_*` | Global saved to .mem file | gll_cExportPath |
| `m.*` | Memory variables (Scatter) | m.cname, m.ddate |
| `lc*` | Local Character | lcSQL, lcWhere |
| `ln*` | Local Numeric | lnTotal, lnCount |
| `ld*` | Local Date | ldStart, ldEnd |
| `ll*` | Local Logical | llSuccess, llFound |
| `la*` | Local Array | laResults, laFiles |
| `lo*` | Local Object | loForm, loException |

### Function Prefixes

| Prefix | Purpose | Example |
|--------|---------|---------|
| `sql_*` | SQL Server helpers | sql_exec(), sql_update() |
| `log_*` | Logging functions | log_error(), log_info() |
| `get_*` | Data retrieval | get_client(), get_settings() |
| `set_*` | Data modification | set_status(), set_config() |

## Code Patterns

### 1. VFP Does NOT Have ELSEIF!

```foxpro
*-- CORRECT - Use DO CASE
DO CASE
CASE nResult > 0
    RETURN .T.
CASE nResult = 0
    RETURN .T.
OTHERWISE
    RETURN .F.
ENDCASE

*-- WRONG - VFP has no ELSEIF!
IF nResult > 0
    RETURN .T.
ELSEIF nResult = 0   && COMPILE ERROR!
    RETURN .T.
ENDIF
```

### 2. SCATTER for Read, REPLACE for Write (NOT GATHER!)

```foxpro
*-- CORRECT - Scatter to read, REPLACE to write specific fields
SELECT clients
Scatter Memvar
m.cname = "New Name"
REPLACE cname WITH m.cname
TABLEUPDATE(.T.)

*-- WRONG - Gather sends ALL fields (risky!)
Scatter Memvar
m.cname = "New Name"
Gather Memvar  && May overwrite fields you didn't intend to change!
```

### 3. File Operations - Use STRTOFILE

```foxpro
*-- CORRECT - STRTOFILE for append
=STRTOFILE(lcText + CHR(13)+CHR(10), lcFile, .T.)

*-- WRONG - Fopen/Fputs/Fclose is fragile
lnHandle = Fopen(lcFile)
=Fputs(lnHandle, lcText)
=Fclose(lnHandle)
```

### 4. NULL Handling

```foxpro
*-- CORRECT - Check type before comparison
IF VARTYPE(ddeadline) = 'D' AND ddeadline < DATE()
    lcStatus = "Overdue"
ENDIF

*-- WRONG - Will crash if field is NULL
IF ddeadline < DATE()  && Error if NULL!
    lcStatus = "Overdue"
ENDIF
```

### 5. Date Comparisons for "Overdue"

```foxpro
*-- CORRECT - < for strictly overdue (excludes today)
IIF(ddeadline < DATE(), "OVERDUE", "OK")

*-- WRONG - <= includes today (not overdue yet!)
IIF(ddeadline <= DATE(), "OVERDUE", "OK")
```

### 6. Error Handling with TRY/CATCH

```foxpro
TRY
    *-- Risky operation
    USE sometable EXCLUSIVE
    *-- Process data
CATCH TO loEx
    *-- Handle error
    log_error(loEx.Message + " in " + loEx.Procedure)
    MESSAGEBOX("Error: " + loEx.Message, 16, "Error")
FINALLY
    *-- Cleanup (always runs)
    USE IN SELECT("sometable")
ENDTRY
```

### 7. Safe Object Property Access

```foxpro
*-- CORRECT - Check object exists first
IF VARTYPE(loObject) = 'O' AND !ISNULL(loObject)
    lcValue = loObject.Property
ENDIF

*-- WRONG - Will crash if object is NULL or doesn't exist
lcValue = loObject.Property
```

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Unrecognized command verb" | Used ELSEIF | Replace with DO CASE |
| "Operator/operand mismatch" | NULL comparison | Use VARTYPE() first |
| "Variable not found" | Missing m. prefix | Use m.varname with Scatter |
| "Property is read-only" | Modifying .Name | Some properties can't change at runtime |

## Best Practices

1. **Always use m. prefix** with SCATTER MEMVAR variables
2. **Prefer local variables** (lc*, ln*) over public (gl_*)
3. **Close cursors explicitly** - don't rely on scope cleanup
4. **Use TABLEUPDATE()** after modifying buffered cursors
5. **Handle NULL values** explicitly in comparisons
6. **Use TRY/CATCH** for operations that might fail
7. **Log errors** to help with debugging

## Quick Reference

```foxpro
*-- String functions
ALLTRIM(string)      && Remove leading/trailing spaces
UPPER(string)        && Convert to uppercase
SUBSTR(string,n,m)   && Extract substring
AT(search, string)   && Find position

*-- Date functions
DATE()               && Current date
DATETIME()           && Current datetime
DTOC(date)           && Date to character
CTOD(string)         && Character to date
DTOS(date)           && Date to string (YYYYMMDD)

*-- Type checking
VARTYPE(var)         && Returns type letter (C,N,D,L,O,U,X)
TYPE("var")          && Returns type (evaluates expression)
ISNULL(var)          && Check if NULL
EMPTY(var)           && Check if empty

*-- Array functions
ALINES(array, string, .T., CHR(13)+CHR(10))  && Split string to array
ALEN(array, 1)       && Number of rows
ASCAN(array, value)  && Find value in array
```

## Resources

- [VFPX Community](https://vfpx.github.io/)
- [VFP Language Reference](https://hackfox.github.io/)
- [FoxBin2Prg](https://github.com/fdbozzo/foxbin2prg) - Version control for VFP
