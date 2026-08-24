# Contributing to Agentic Review

Thank you for your interest in contributing to Agentic Review! This document provides guidelines and instructions for contributing.

## Code of Conduct

By participating in this project, you agree to abide by our Code of Conduct (implied: be respectful, inclusive, and constructive).

## How to Contribute

### Reporting Bugs

If you find a bug, please create an issue with:
- **Clear title** describing the problem
- **Steps to reproduce** the issue
- **Expected behavior** vs. actual behavior
- **Environment** (platform, OS, MCP client version)
- **Logs/screenshots** if applicable

### Suggesting Features

Feature requests are welcome! Please create an issue with:
- **Clear description** of the feature
- **Use case** - Why is this useful?
- **Proposed implementation** (if you have ideas)
- **Alternatives considered**

### Pull Requests

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Make your changes**
4. **Write/update tests** (if applicable)
5. **Update documentation** (README, SKILL files, etc.)
6. **Commit with clear messages** (`git commit -m 'Add amazing feature'`)
7. **Push to your fork** (`git push origin feature/amazing-feature`)
8. **Open a Pull Request**

#### Pull Request Guidelines

- **One feature per PR** - Keep changes focused
- **Write good commit messages** - Explain what and why
- **Update documentation** - Keep docs in sync with code
- **Test your changes** - Ensure nothing breaks
- **Follow existing code style**

## Development Setup

### Prerequisites

- Python 3.11+
- PostgreSQL 16 + PostGIS
- Google Cloud SDK (for deployment)
- Git

### Local Development

1. **Clone the repository**
   ```bash
   git clone https://github.com/agenticreview/agentic-review.git
   cd agentic-review
   ```

2. **Set up MCP server** (if modifying server code)
   ```bash
   # See main MCP server repository for setup
   # This repo contains only the plugin manifests and SKILL guides
   ```

3. **Test your changes**
   - For plugin changes: Test with Claude Code or Cursor
   - For SKILL changes: Verify workflows still work
   - For documentation: Check formatting and links

## Project Structure

```
agentic-review/
├── marketplace.json          # Marketplace manifest
├── claude-code/              # Claude Code plugin
├── cursor/                   # Cursor plugin
├── README.md                 # Main docs
├── AGENTS.md                 # Agent navigation
├── llms.txt                  # LLM context
└── docs/                     # Additional docs
```

## Documentation Guidelines

### README.md
- Keep concise and scannable
- Include examples and code snippets
- Update badges and links
- Test all links before committing

### SKILL Files
- Follow existing format (Overview, Tools, Workflow, Patterns, Tips)
- Include code examples
- Be specific and actionable
- Think "How would a user DO this?" not "What does this tool do?"

### AGENTS.md
- Update navigation when adding new files
- Keep workflow diagrams current
- Add new pitfalls as discovered

### llms.txt
- Update when architecture changes
- Keep examples current
- Maintain comprehensive context

## Testing Checklist

Before submitting a PR, verify:

- [ ] All links work (no 404s)
- [ ] Code examples are correct
- [ ] SKILL workflows tested end-to-end
- [ ] Documentation is up-to-date
- [ ] Commit messages are clear
- [ ] No sensitive data (API keys, tokens, etc.)

## Coding Standards

### Markdown
- Use ATX-style headers (`#` not `---`)
- Code blocks have language specified
- Lists are consistent (all `-` or all `*`)
- Line length ~80-100 characters (soft limit)

### JSON
- 2-space indentation
- No trailing commas
- Validate with `jsonlint` or similar

### Code Examples
- Use TypeScript for clarity
- Include error handling
- Show real-world usage
- Comment non-obvious parts

## Release Process

(For maintainers)

1. Update version in `marketplace.json` and plugin manifests
2. Update CHANGELOG.md
3. Tag release: `git tag v1.x.x`
4. Push tags: `git push --tags`
5. Create GitHub release with notes

## Questions?

- **General questions:** Open a Discussion
- **Bug reports:** Open an Issue
- **Feature requests:** Open an Issue
- **Security issues:** Email security@agenticreview.com (private)

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for contributing to Agentic Review! 🎉
