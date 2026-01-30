# Contributing to Kilo-Kit

Thank you for your interest in contributing to Kilo-Kit! This document provides guidelines and instructions for contributing.

---

## 📋 Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Setup](#development-setup)
4. [Contribution Types](#contribution-types)
5. [Skill Development Guidelines](#skill-development-guidelines)
6. [Pull Request Process](#pull-request-process)
7. [Coding Standards](#coding-standards)
8. [Commit Message Format](#commit-message-format)

---

## 📜 Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inclusive environment. All contributors are expected to:

- Be respectful and considerate
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment, discrimination, or offensive language
- Personal attacks or trolling
- Publishing others' private information
- Any conduct inappropriate in a professional setting

---

## 🚀 Getting Started

### Prerequisites

- Git
- Python 3.11+ (for tools)
- Understanding of AI agent systems
- Familiarity with Markdown

### Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR-USERNAME/kilo-kit.git
cd kilo-kit
git remote add upstream https://github.com/kilo-kit/kilo-kit.git
```

### Stay Updated

```bash
git fetch upstream
git checkout main
git merge upstream/main
```

---

## 🔧 Development Setup

### Directory Structure

```
kilo-kit/
├── src/
│   ├── core/           # Core components (careful!)
│   ├── skills/         # Skill modules
│   ├── behaviors/      # Behavior units
│   └── tools/          # CLI utilities
├── docs/               # Documentation
├── examples/           # Usage examples
└── tests/              # Test cases
```

### Running Tests

```bash
# Run all tests
python -m pytest tests/

# Run specific test
python -m pytest tests/unit/test-intent-parser.py
```

### Validating Skills

```bash
# Validate a skill structure
python src/tools/validate-skill.py src/skills/my-skill/
```

---

## 🎁 Contribution Types

### 1. Documentation Improvements

- Fix typos or clarify language
- Add examples or use cases
- Improve guides or tutorials
- Translate documentation

**Low barrier, high impact!**

### 2. New Skills

- Create skills for new domains
- Improve existing skills
- Add skill references/scripts

**See [Skill Development Guidelines](#skill-development-guidelines)**

### 3. New Behaviors

- Create atomic behaviors
- Create compound behaviors
- Improve behavior composition

### 4. Core Improvements

- Bug fixes
- Performance optimizations
- New features

**Requires discussion first — open an issue!**

### 5. Tools and Utilities

- CLI improvements
- New analysis tools
- Automation scripts

---

## 📝 Skill Development Guidelines

### Skill Structure

Every skill MUST have:

```
skill-name/
├── SKILL.md           # Required - Main instructions
├── references/        # Optional - Detailed docs
├── scripts/           # Optional - Helper scripts
└── assets/            # Optional - Templates, images
```

### SKILL.md Format

```yaml
---
name: skill-name
description: >-
  Clear description including trigger keywords.
  Keywords: debug, fix, error
version: 1.0.0
behaviors: []
dependencies: []
token_estimate:
  min: 500
  typical: 1500
  max: 5000
---

# Skill Name

## When to Use
- Trigger condition 1
- Trigger condition 2

## Prerequisites
- What must be true

## Process
1. Step 1
2. Step 2
3. Step 3

## Guidelines
- Guideline 1
- Guideline 2

## Common Patterns
- Pattern 1: When X, do Y

## Anti-Patterns (AVOID)
- Don't do X

## Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2

## References
- `references/guide.md` - Detailed guide
```

### Skill Quality Checklist

- [ ] Clear, descriptive name
- [ ] Keywords in description for dispatch
- [ ] Token estimates provided
- [ ] Process steps are actionable
- [ ] Success criteria are verifiable
- [ ] Anti-patterns documented
- [ ] Tested with real tasks

---

## 🔄 Pull Request Process

### 1. Create Feature Branch

```bash
git checkout -b feature/issue-123-add-new-skill
```

**Branch naming:**
- `feature/issue-{n}-{description}` - New features
- `fix/issue-{n}-{description}` - Bug fixes
- `docs/{description}` - Documentation only

### 2. Make Changes

- Follow coding standards
- Write clear commit messages
- Keep changes focused (one logical change per PR)

### 3. Test Your Changes

```bash
# Run tests
python -m pytest tests/

# Validate skills
python src/tools/validate-skill.py src/skills/your-skill/
```

### 4. Submit Pull Request

- Fill out the PR template
- Link related issues
- Request review from maintainers

### 5. Address Review Feedback

- Respond to all comments
- Make requested changes
- Re-request review when ready

### PR Template

```markdown
## Description
Brief description of changes.

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation
- [ ] Breaking change

## Related Issues
Closes #123

## Checklist
- [ ] Tests pass
- [ ] Documentation updated
- [ ] Follows coding standards
- [ ] Commit messages follow format
```

---

## 💻 Coding Standards

### Markdown Files

- Use proper heading hierarchy (# > ## > ###)
- Use code blocks with language hints
- Keep lines under 100 characters when possible
- Use tables for structured data
- Include table of contents for long docs

### Python Scripts

```python
"""
Module docstring describing purpose.
"""

def function_name(param: str) -> bool:
    """
    Function docstring.
    
    Args:
        param: Description of parameter
        
    Returns:
        Description of return value
    """
    pass
```

**Standards:**
- Python 3.11+ syntax
- Type hints required
- Docstrings for all public functions
- Ruff for linting
- Black for formatting

### YAML in Markdown

```yaml
# Use consistent indentation (2 spaces)
key:
  nested_key: value
  list:
    - item1
    - item2
```

---

## 📝 Commit Message Format

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `refactor` | Code change, no new feature |
| `test` | Adding tests |
| `chore` | Maintenance tasks |

### Examples

```
feat(skills): add systematic-debugging skill

- Implements 4-phase debugging process
- Includes root cause analysis
- Adds verification checklist

Closes #42
```

```
fix(token-economy): correct budget calculation

Budget was not accounting for verification phase.
Now correctly reserves 10% for post-execution checks.

Fixes #57
```

```
docs: update quickstart guide

- Clarify setup steps
- Add troubleshooting section
- Fix typos
```

---

## 🏷️ Issue Labels

| Label | Description |
|-------|-------------|
| `good first issue` | Good for newcomers |
| `help wanted` | Extra attention needed |
| `bug` | Something isn't working |
| `enhancement` | New feature request |
| `documentation` | Documentation improvements |
| `skill` | Skill-related |
| `core` | Core system changes |

---

## 🤝 Getting Help

- **Questions:** Open a Discussion
- **Bugs:** Open an Issue
- **Feature Ideas:** Open an Issue with `enhancement` label
- **Security Issues:** Email security@kilo-kit.dev (private)

---

## 📄 License

By contributing, you agree that your contributions will be licensed under the Apache 2.0 License.

---

*Thank you for contributing to Kilo-Kit! Together we build better AI agents.* 🚀
