---
name: form-editor
description: Guide for editing VFP forms (.scx/.sc2 files). Use when modifying forms, understanding form structure, editing form code, or working with controls and events.
allowed-tools: Read, Grep, Glob, Edit
---

# VFP Form Editor Guide

## Understanding VFP Forms

VFP forms are stored in binary `.scx` files (with `.sct` memo). For version control, many teams convert to text format using [FoxBin2Prg](https://github.com/fdbozzo/foxbin2prg).

| File Type | Format | Use |
|-----------|--------|-----|
| `.scx/.sct` | Binary | VFP IDE uses this |
| `.sc2` | Text | Version control, Claude editing |

## What AI CAN Do

- Edit code in existing procedures/methods
- Modify object properties (Caption, Value, Enabled, etc.)
- Edit .prg files completely
- Add/remove code lines within existing methods

## What AI CANNOT Do

- Add new objects to forms (buttons, textboxes, etc.)
- Delete objects from forms
- Add new methods/events to objects
- Modify form structure (add containers, pages, etc.)

**Reason:** VFP forms have complex binary relationships that require the VFP IDE to manage correctly.

## .sc2 File Structure

```
*-- Header
PLATFORM,UNIQUEID,TIMESTAMP,CLASS,CLASSLOC,BASECLASS,...

*-- Form Definition
CLASS,BASECLASS,OBJNAME,PARENT,PROPERTIES,PROTECTED,METHODS,OBJCODE,NAME

*-- Properties Block
Name = "myform"
Caption = "My Form (ver.1)"
Top = 0
Left = 0
Height = 600
Width = 800
...

*-- Methods Block
PROCEDURE Init
    *-- Initialization code
ENDPROC

PROCEDURE cmdSave.Click
    *-- Button click handler
ENDPROC

*-- Child Object Definition
CLASS,BASECLASS,OBJNAME,PARENT,...
```

## Editing Properties

### Change Caption with Version

```foxpro
*-- Find the Caption line in Properties section
Caption = "Client Editor (ver.15)"

*-- Increment version when making changes
Caption = "Client Editor (ver.16)"
```

### Common Editable Properties

| Property | Type | Example |
|----------|------|---------|
| `Caption` | String | `"Save"` |
| `Value` | Varies | `0`, `""`, `.T.` |
| `Enabled` | Logical | `.T.`, `.F.` |
| `Visible` | Logical | `.T.`, `.F.` |
| `Top`, `Left` | Numeric | `10`, `20` |
| `Height`, `Width` | Numeric | `25`, `100` |
| `FontName` | String | `"Segoe UI"` |
| `FontSize` | Numeric | `9` |
| `BackColor` | Numeric | `RGB(255,255,255)` |
| `ForeColor` | Numeric | `RGB(0,0,0)` |
| `ToolTipText` | String | `"Click to save"` |

## Editing Methods

### Locate Method Block

```foxpro
PROCEDURE methodname
    *-- Method code here
ENDPROC
```

### Common Events to Edit

| Event | Fires When |
|-------|------------|
| `Init` | Object is created |
| `Destroy` | Object is destroyed |
| `Click` | User clicks |
| `DblClick` | User double-clicks |
| `Valid` | Focus leaves (validation) |
| `When` | Focus enters (permission) |
| `InteractiveChange` | Value changes (interactive) |
| `ProgrammaticChange` | Value changes (code) |
| `GotFocus` | Focus received |
| `LostFocus` | Focus lost |
| `KeyPress` | Key pressed |
| `Refresh` | Refresh called |

### Button Click Example

```foxpro
PROCEDURE cmdSave.Click
    IF THISFORM.Validate()
        THISFORM.SaveData()
        THISFORM.Release()
    ELSE
        MESSAGEBOX("Please fix errors before saving.", 48, "Validation")
    ENDIF
ENDPROC
```

### Form Init Example

```foxpro
PROCEDURE Init
    *-- Load data
    IF !THISFORM.LoadData()
        MESSAGEBOX("Failed to load data.", 16, "Error")
        RETURN .F.
    ENDIF

    *-- Set defaults
    THISFORM.txtName.SetFocus()

    RETURN .T.  && Return .F. to prevent form from opening
ENDPROC
```

## Grid DynamicProperties

Grids support dynamic coloring based on data:

```foxpro
*-- In form Init or grid setup
LOCAL lcBackExpr
lcBackExpr = "IIF(nstatus=1, RGB(200,255,200), " + ;
             "IIF(nstatus=2, RGB(255,200,200), " + ;
             "RGB(255,255,255)))"

THIS.grdData.Column1.DynamicBackColor = lcBackExpr
THIS.grdData.Column2.DynamicBackColor = lcBackExpr
```

## Version Tracking Rule

**MANDATORY** when editing .sc2 files:

1. Find the form's Caption property
2. Increment the version number
3. Keep format: `"Form Name (ver.N)"`

```foxpro
*-- Before
Caption = "Client Editor (ver.15)"

*-- After your edit
Caption = "Client Editor (ver.16)"
```

## FoxBin2Prg Workflow

If using text-based version control:

```
1. Developer edits .sc2 file (text)
2. Run conversion: DO foxbin2prg.prg WITH "formname.sc2"
3. Test in VFP with resulting .scx
4. Commit .sc2 to version control
```

## Common Form Patterns

### Modal Form for Data Entry

```foxpro
*-- Parent form calls:
LOCAL loData
loData = CREATEOBJECT("Empty")
ADDPROPERTY(loData, "cName", "")
ADDPROPERTY(loData, "lSaved", .F.)

DO FORM frmEdit WITH loData

IF loData.lSaved
    *-- Save to database
    INSERT INTO clients (cname) VALUES (loData.cName)
ENDIF
```

### Child form stores in passed object:

```foxpro
PROCEDURE cmdOK.Click
    THISFORM.oData.cName = THISFORM.txtName.Value
    THISFORM.oData.lSaved = .T.
    THISFORM.Release()
ENDPROC
```

### Filter/Refresh Pattern

```foxpro
PROCEDURE cmdFilter.Click
    LOCAL lcWhere
    lcWhere = "1=1"

    IF !EMPTY(THISFORM.txtSearch.Value)
        lcWhere = lcWhere + " AND cname LIKE '%" + ;
            sql_quote(ALLTRIM(THISFORM.txtSearch.Value)) + "%'"
    ENDIF

    sql_exec_safe("SELECT * FROM clients WHERE " + lcWhere, "qryClients")
    THISFORM.grdClients.RecordSource = "qryClients"
    THISFORM.grdClients.Refresh()
ENDPROC
```

## Checklist for Form Modifications

- [ ] Read the .sc2 file before editing
- [ ] Locate the correct method/property
- [ ] Make the change
- [ ] Increment version in Caption
- [ ] Run conversion tool (if using text workflow)
- [ ] Test in VFP

## Resources

- [FoxBin2Prg](https://github.com/fdbozzo/foxbin2prg) - Binary to text conversion
- [PEM Editor](https://github.com/VFPX/PEMEditor) - Enhanced property editor
- [VFPX Community](https://vfpx.github.io/)
