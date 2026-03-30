# Frontal Agent Skills

A comprehensive collection of Agent Skills for Frontal Labs' polyglot platform development. These skills provide specialized expertise for AI/ML, API evolution, code architecture, DevOps, and performance optimization.

## Installation

```bash
npx skills add frontal-labs/agent-skills
```

## Available Skills

### 🤖 AI Specialist
**Category**: AI/ML  
**Use when**: Designing AI systems, optimizing models, or implementing machine learning workflows

Expertise areas:
- AI/ML architecture design
- Model selection and optimization
- Training pipeline development
- MLOps best practices
- Cross-language ML implementation

### 🔌 API Evolution
**Category**: Backend Architecture  
**Use when**: Managing API changes, versioning strategies, or cross-service contract evolution

Expertise areas:
- Protocol buffer evolution
- REST API versioning
- Database schema evolution
- Client code generation
- Breaking change management

### 💻 Code Expert
**Category**: Software Architecture  
**Use when**: Analyzing code architecture, reviewing service integration, or optimizing cross-language systems

Expertise areas:
- Polyglot architecture patterns
- Go, Python, Rust, TypeScript expertise
- Cross-service integration
- Performance optimization
- Security best practices

### 🚀 DevOps Engineer
**Category**: DevOps & Infrastructure  
**Use when**: Designing infrastructure, managing deployments, or optimizing DevOps workflows

Expertise areas:
- Kubernetes orchestration
- CI/CD pipeline management
- Monitoring and observability
- Security and compliance
- Resource optimization

### ⚡ Performance Optimizer
**Category**: Performance Optimization  
**Use when**: Improving performance metrics, optimizing load times, or enhancing user experience

Expertise areas:
- Web performance optimization
- API performance tuning
- Database optimization
- Caching strategies
- Performance monitoring

### 🔧 Monorepo Improver
**Category**: Monorepo Management  
**Use when**: Improving monorepo structure, analyzing code quality, or implementing systematic improvements

Expertise areas:
- LangChain-powered analysis
- Codebase optimization
- Testing enhancement
- Architectural improvements
- Multi-agent orchestration

## Usage

Once installed, skills are automatically available. The agent will use them when relevant tasks are detected:

### Example Usage

```bash
# AI/ML tasks
"Design an AI architecture for user recommendation system"
"Optimize this machine learning model for production"
"Set up MLOps pipeline for model training"

# API evolution tasks
"Evolve our API to support new user features"
"Manage breaking changes in our REST API"
"Generate client code for our protocol buffers"

# Code architecture tasks
"Review our polyglot service architecture"
"Optimize cross-service communication"
"Implement security best practices"

# DevOps tasks
"Set up Kubernetes deployment for our services"
"Design CI/CD pipeline for polyglot application"
"Implement monitoring and observability"

# Performance tasks
"Optimize our web application performance"
"Improve API response times"
"Set up caching strategy for high traffic"

# Monorepo tasks
"Improve our monorepo structure"
"Analyze codebase quality"
"Implement systematic improvements"
```

## Skill Structure

Each skill follows the Agent Skills specification:

```
skills/skill-name/
├── SKILL.md              # Required: metadata + instructions
├── scripts/              # Optional: executable code
├── references/           # Optional: documentation
└── assets/               # Optional: templates, resources
```

## Development

### Prerequisites

- Node.js 16+
- npm or yarn

### Scripts

```bash
# Validate all skills
npm run validate

# List available skills
npm run list

# Install skills locally
npm run install-skills

# Run linting
npm run lint

# Run tests
npm test
```

### Adding New Skills

1. Create a new directory in `skills/` following naming conventions:
   - 1-64 characters
   - Lowercase alphanumeric characters and hyphens
   - Must not start or end with a hyphen
   - Must not contain consecutive hyphens

2. Create `SKILL.md` with proper frontmatter:

```markdown
---
name: skill-name
description: A description of what this skill does and when to use it.
license: MIT
metadata:
  author: frontal-labs
  version: "1.0"
  category: category-name
---

# Skill Content
...
```

### Validation

Skills are automatically validated for:

- Proper frontmatter structure
- Required fields (name, description)
- Naming convention compliance
- File structure requirements

## Categories

- **ai-ml**: Artificial intelligence and machine learning
- **backend-architecture**: API design, database, and backend systems
- **software-architecture**: Code structure, patterns, and design
- **devops-infrastructure**: Deployment, monitoring, and operations
- **performance-optimization**: Performance tuning and optimization
- **monorepo-management**: Monorepo structure and tooling

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

### Quick Start

1. Fork the repository
2. Create a feature branch
3. Add or modify skills following the specification
4. Validate your changes: `npm run validate`
5. Submit a pull request

## Security

If you discover a security vulnerability, please see our [Security Policy](SECURITY.md) for responsible disclosure.

## License

MIT License - see [LICENSE.md](LICENSE.md) file for details.

## Support

- **Issues**: [GitHub Issues](https://github.com/frontal-labs/agent-skills/issues)
- **Discussions**: [GitHub Discussions](https://github.com/frontal-labs/agent-skills/discussions)
- **Documentation**: [Agent Skills Specification](https://agentskills.io/specification)

## Related Projects

- [Frontal Platform](https://github.com/frontal-labs/frontal) - Polyglot cloud platform
- [Agent Skills](https://github.com/agentskills/agentskills) - Open skills format
- [Frontal SDK](https://github.com/frontal-labs/sdk-ts) - TypeScript SDK

---

**Frontal Labs** - Building the future of polyglot development
