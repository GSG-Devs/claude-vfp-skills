# Contributing to Claude VFP Skills

Thank you for your interest in contributing to the Claude VFP Skills project! This project aims to help the Visual FoxPro community leverage AI coding assistants effectively.

## How to Contribute

### Reporting Issues

- Use the [GitHub Issues](https://github.com/GSG-Devs/claude-vfp-skills/issues) page
- Describe the problem clearly
- Include VFP version, SQL Server version if applicable
- Provide code samples if possible

### Suggesting Enhancements

- Open an issue with the "enhancement" label
- Describe the use case
- Explain why it would benefit VFP developers

### Submitting Code

1. **Fork the repository**
   ```bash
   git clone https://github.com/GSG-Devs/claude-vfp-skills.git
   cd claude-vfp-skills
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Follow the existing SKILL.md format
   - Include practical code examples
   - Test with Claude Code before submitting

4. **Commit your changes**
   ```bash
   git add .
   git commit -m "Add: description of your changes"
   ```

5. **Push and create a Pull Request**
   ```bash
   git push origin feature/your-feature-name
   ```
   Then open a PR on GitHub.

## Skill File Guidelines

### Structure

Each skill should be in its own folder:
```
skills/
└── your-skill-name/
    └── SKILL.md
```

### SKILL.md Format

```markdown
---
name: your-skill-name
description: Clear description of when Claude should use this skill
allowed-tools: Read, Grep, Glob
---

# Skill Title

## Section 1
Content with code examples...

## Section 2
More content...
```

### Best Practices for Skills

1. **Be specific** - Include concrete code examples
2. **Show both wrong and right** - Help Claude avoid common mistakes
3. **Include tables** - Quick reference tables are very useful
4. **Link to resources** - Reference VFPX, documentation, etc.
5. **Keep it focused** - One skill per topic area

## Code Style

### VFP Code Examples

```foxpro
*-- Use comments to explain
LOCAL lcVariable  && Use descriptive names

*-- Show wrong way first
*-- WRONG
code_that_fails()

*-- Then show correct way
*-- CORRECT
code_that_works()
```

### Markdown Style

- Use ATX-style headers (`#`, `##`, `###`)
- Use fenced code blocks with language hints
- Use tables for quick reference
- Keep lines under 100 characters when possible

## Ideas for Contributions

Here are some areas where contributions would be especially welcome:

### New Skills

- [ ] Report (.frx) editing guide
- [ ] Class library (.vcx) patterns
- [ ] Menu (.mnx) editing
- [ ] Multi-language/localization
- [ ] VFP to .NET migration helpers
- [ ] MySQL/PostgreSQL integration (alternative to SQL Server)
- [ ] OLE/COM integration patterns
- [ ] Web services/REST API consumption

### Improvements to Existing Skills

- More code examples
- Edge case handling
- Performance tips
- Additional error messages and solutions

### Documentation

- Tutorials for beginners
- Video guides (link to)
- Translations to other languages

## Testing Your Changes

Before submitting, test your skill with Claude Code:

1. Copy your skill to `.claude/skills/` in a VFP project
2. Start Claude Code in that project
3. Ask Claude to help with code in your skill's domain
4. Verify Claude follows your guidelines

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help newcomers to VFP and AI tools
- Remember: VFP developers span many cultures and experience levels

## Questions?

- Open an issue for questions
- Tag it with "question" label
- Or reach out to the maintainers

## Recognition

Contributors will be recognized in the README.md file. Thank you for helping the VFP community!

---

**Maintained by [Geseidl Consulting Group](https://geseidl.ro)**
