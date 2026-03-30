# Contributing to Frontal Agent Skills

We welcome contributions to the Frontal Agent Skills collection! This document provides guidelines for contributing.

## Getting Started

1. Fork the repository
2. Clone your fork locally
3. Install dependencies: `npm install`
4. Create a new branch for your feature

## Adding New Skills

### Skill Structure

Each skill must follow this structure:

```
skills/skill-name/
├── SKILL.md              # Required: metadata + instructions
├── scripts/              # Optional: executable code
├── references/           # Optional: documentation
└── assets/               # Optional: templates, resources
```

### Skill Requirements

1. **Naming Convention**:
   - 1-64 characters
   - Lowercase alphanumeric characters and hyphens only
   - Must not start or end with a hyphen
   - Must not contain consecutive hyphens

2. **Frontmatter Requirements**:
   ```yaml
   ---
   name: skill-name
   description: A description of what this skill does and when to use it.
   license: MIT
   metadata:
     author: frontal-labs
     version: "1.0"
     category: category-name
   ---
   ```

3. **Content Guidelines**:
   - Clear, concise descriptions
   - Specific activation triggers
   - Practical examples and code snippets
   - Well-structured sections

### Validation

Run the validation script before submitting:

```bash
npm run validate
```

## Development Process

1. **Create Issue**: Discuss your idea in an issue first
2. **Branch**: Create a feature branch from `main`
3. **Develop**: Implement your changes following the guidelines
4. **Test**: Run validation and tests
5. **Submit**: Create a pull request with clear description

## Pull Request Guidelines

- Use descriptive titles and descriptions
- Reference any related issues
- Ensure all validation checks pass
- Follow the existing code style and structure

## Code Style

- Use markdown for documentation
- Follow the Agent Skills specification
- Keep descriptions clear and concise
- Use proper formatting and syntax highlighting

## Categories

Skills should be categorized appropriately:

- `ai-ml`: Artificial intelligence and machine learning
- `backend-architecture`: API design, database, and backend systems
- `software-architecture`: Code structure, patterns, and design
- `devops-infrastructure`: Deployment, monitoring, and operations
- `performance-optimization`: Performance tuning and optimization
- `monorepo-management`: Monorepo structure and tooling

## Questions?

Feel free to open an issue for any questions about contributing!
