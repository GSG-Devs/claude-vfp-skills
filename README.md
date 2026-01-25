# Claude VFP Skills

[![Visual FoxPro](https://img.shields.io/badge/Visual%20FoxPro-9.0-blue.svg)](https://en.wikipedia.org/wiki/Visual_FoxPro)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-green.svg)](https://claude.ai/code)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![VFPX Community](https://img.shields.io/badge/VFPX-Community-orange.svg)](https://vfpx.github.io/)

**AI-powered coding assistance for Visual FoxPro developers using Claude Code.**

This is the **first** Claude Code skills package specifically designed for Visual FoxPro (VFP) development. It provides conventions, patterns, and best practices that help Claude understand and generate quality VFP code.

---

## Table of Contents

- [Why This Exists](#why-this-exists)
- [Features](#features)
- [Installation](#installation)
- [Available Skills](#available-skills)
- [Usage](#usage)
- [CLAUDE.md Template](#claudemd-template)
- [VFP Community Resources](#vfp-community-resources)
- [Contributing](#contributing)
- [License](#license)

---

## Why This Exists

Visual FoxPro is a powerful database programming language with an active community, but AI coding assistants like Claude weren't specifically trained on VFP patterns. This project bridges that gap by providing:

- **Coding conventions** that Claude can follow
- **SQL Server integration patterns** (SPT, ODBC, buffered cursors)
- **Form editing guidelines** for .scx/.sc2 files
- **Debugging helpers** for common VFP errors

## Features

| Skill | Description |
|-------|-------------|
| `vfp-conventions` | Naming conventions, variable prefixes, code patterns |
| `sql-helper` | SQL Server integration, sql_exec_safe(), transactions |
| `form-editor` | Guide for editing VFP forms (.scx/.sc2 files) |
| `debug-helper` | Troubleshooting common VFP and SQL errors |

## Installation

### Option 1: Project-Level (Recommended)

Copy the `skills/` folder to your VFP project:

```bash
# Clone this repo
git clone https://github.com/GSG-Devs/claude-vfp-skills.git

# Copy skills to your project
cp -r claude-vfp-skills/skills/ your-vfp-project/.claude/skills/
```

### Option 2: Global Installation

Install skills for all your projects:

```bash
# Copy to global Claude config
cp -r claude-vfp-skills/skills/* ~/.claude/skills/
```

### Option 3: Manual

1. Create `.claude/skills/` folder in your project
2. Copy the desired `SKILL.md` files from this repo
3. Restart Claude Code

## Available Skills

### `/vfp-conventions`

Automatically loaded when writing VFP code. Includes:

- Variable prefixes (`lc*`, `ln*`, `ld*`, `ll*`, `gl_*`, `m.*`)
- Database field prefixes (`id*`, `d*`, `t*`, `n*`, `c*`, `l*`)
- Function naming (`sql_*`, `log_*`)
- DO CASE instead of ELSEIF (VFP doesn't have ELSEIF!)
- SCATTER/REPLACE patterns (not GATHER)

### `/sql-helper`

For SQL Server integration via SPT (SQL Pass-Through):

- `sql_exec_safe()` patterns with retry logic
- Date vs Datetime handling (`d*` fields = `Date()`, `t*` fields = `Datetime()`)
- Transaction patterns (single T-SQL batch for multi-step operations)
- ROWLOCK hints for multi-user environments
- NULL handling in comparisons

### `/form-editor`

Guidelines for editing VFP forms:

- .sc2 file structure (text representation of .scx)
- What Claude CAN do (edit code in methods, modify properties)
- What Claude CANNOT do (add/delete objects, add methods)
- Version tracking in Caption
- prg2bin workflow for text-to-binary conversion

### `/debug-helper`

Troubleshooting common issues:

- "Conversion failed date/time" errors
- "Lock request time out" in multi-user scenarios
- Error 1492 "No key columns"
- Data type mismatch with `sql_exec_safe()` return values
- Session blocking diagnosis with `sys.dm_exec_sessions`

## Usage

### Automatic (Recommended)

Claude automatically loads relevant skills based on context. Just start coding!

```
You: "Write a function to save client data to SQL Server"
Claude: [Loads sql-helper skill, uses sql_exec_safe(), proper date handling]
```

### Manual Invocation

Invoke a skill directly:

```
/vfp-conventions
/sql-helper SELECT * FROM dbo.clients
/debug-helper "Conversion failed"
```

## CLAUDE.md Template

For the best experience, add a `CLAUDE.md` file to your VFP project root. See [examples/CLAUDE.md.template](examples/CLAUDE.md.template) for a complete starting point.

Basic structure:

```markdown
# Project Context

## Overview
- Language: Visual FoxPro 9.0
- Database: SQL Server (version)
- Architecture: SQL Pass-Through with buffered cursors

## Naming Conventions
[Link to vfp-conventions skill or inline rules]

## Key Functions
- `sql_exec_safe(cSQL, cCursor)` - Execute SQL with retry
- `foloseste(cTable, cCursor)` - Open table with buffering
- `inchide(cCursor)` - Close cursor (auto-saves)

## Project-Specific Rules
[Your custom rules here]
```

## VFP Community Resources

This project is proudly connected to the Visual FoxPro community:

### Official Resources

| Resource | Description |
|----------|-------------|
| [VFPX](https://vfpx.github.io/) | Open Source Add-ons for Visual FoxPro |
| [VFPX GitHub](https://github.com/VFPX) | 117+ community projects |
| [VFP Wiki](https://github.com/VFPX/VFPWiki) | Community documentation |

### Essential VFPX Tools

| Tool | Description |
|------|-------------|
| [FoxBin2Prg](https://github.com/fdbozzo/foxbin2prg) | Binary to text conversion for version control |
| [GoFish](https://github.com/VFPX/GoFish) | Advanced code search tool |
| [Thor](https://github.com/VFPX/Thor) | Tool manager and IDE enhancements |
| [PEM Editor](https://github.com/VFPX/PEMEditor) | Property/Event/Method editor |
| [FoxCharts](https://github.com/VFPX/FoxCharts) | Charting library |
| [DPI-Aware Manager](https://github.com/VFPX/DPIAwareManager) | High-DPI support |

### Community

| Platform | Link |
|----------|------|
| VFP Facebook Group | [Visual FoxPro Developers](https://www.facebook.com/groups/visualfoxprodevs/) |
| Stack Overflow | [VFP Tag](https://stackoverflow.com/questions/tagged/visual-foxpro) |
| Google Groups | [VFP Discussion](https://groups.google.com/g/comp.databases.xbase.fox) |
| Foxite (Archive) | [foxite.com](http://www.foxite.com/) |

### Learning Resources

- [VFP Language Reference](https://hackfox.github.io/section5/s5c1.html) - HackFox documentation
- [Visual FoxPro Wiki](http://fox.wikis.com/) - Community wiki
- [Tamar Granor's Site](http://www.tomorrowssolutionsllc.com/) - Books and articles

## Contributing

Contributions are welcome! This project aims to help the VFP community leverage AI coding assistants.

### How to Contribute

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-skill`)
3. Add or improve skills
4. Test with Claude Code
5. Submit a Pull Request

### Ideas for Contributions

- Additional VFP patterns and idioms
- Integration with other databases (MySQL, PostgreSQL)
- Report (.frx) editing guidelines
- Class library (.vcx) patterns
- Migration guides (VFP to .NET, etc.)

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## About the Maintainers

This project is developed and maintained by **[Geseidl Consulting Group](https://geseidl.ro)** (GSG), a family-run accounting and consulting firm based in Ploiești, Romania, with over 20 years of experience.

We use Visual FoxPro for our internal time tracking and management systems, and created these skills to improve our development workflow with Claude Code.

| | |
|---|---|
| **Website** | [geseidl.ro](https://geseidl.ro) |
| **Services** | Audit, Tax Consulting, IT Services, Accounting |
| **Location** | Ploiești, Romania |
| **Contact** | office@geseidl.ro |

---

## Acknowledgments

- The [VFPX](https://github.com/VFPX) community for keeping VFP alive
- [Anthropic](https://www.anthropic.com/) for Claude and Claude Code
- All VFP developers who continue to maintain and improve legacy systems

---

**Made with care for the VFP community by [Geseidl Consulting Group](https://geseidl.ro)**

*If this project helps you, consider starring it and sharing with other VFP developers!*
