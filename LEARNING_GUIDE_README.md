# Label Studio Source Code Learning Guides

This repository contains comprehensive learning guides for understanding the Label Studio codebase.

## Available Guides

### 📚 5-Day Source Code Learning Guide

Structured learning plans for developers who want to understand Label Studio's architecture and codebase:

- **[Chinese Version (中文版)](./docs/source/guide/学习Label-Studio源码指南.md)** - 为后端开发实习生准备的5天学习计划
- **[English Version](./docs/source/guide/source-code-learning-guide.md)** - 5-day structured learning plan for developers

## What's Covered

Both guides cover:

1. **Day 1: Project Structure & Environment Setup**
   - Understanding the codebase organization
   - Setting up development environment
   - Running the application locally

2. **Day 2: Core Modules & Data Models**
   - Data model relationships (Organization, Project, Task, Annotation)
   - Label configuration system
   - Django signals and hooks

3. **Day 3: API Layer & Business Logic**
   - Django REST Framework patterns
   - ViewSets, Serializers, and Permissions
   - API testing and exploration

4. **Day 4: Storage, Import/Export & Async Tasks**
   - Storage backends (S3, GCS, Azure, Local)
   - Data import/export flows
   - Redis Queue (RQ) for background tasks

5. **Day 5: ML Integration & Advanced Features**
   - Machine Learning backend integration
   - Feature flags and webhooks
   - Testing framework and code quality tools

## Key Features

- ✅ **Structured 5-day plan** with clear daily goals
- ✅ **Hands-on exercises** to reinforce learning
- ✅ **Architecture diagrams** and data flow explanations
- ✅ **Code navigation tips** and debugging strategies
- ✅ **Best practices** for contributing to the project
- ✅ **Comprehensive resource links** for further learning

## Target Audience

These guides are designed for:

- Backend developers learning the Label Studio codebase
- Developers preparing to contribute to the project
- Teams customizing Label Studio for their organization
- Interns and junior developers studying data annotation platforms

## Technology Stack Covered

**Backend:**
- Python 3.10+
- Django 5.1+
- Django REST Framework
- PostgreSQL/SQLite
- Redis & RQ (Redis Queue)

**Frontend:**
- React
- MobX State Tree
- TypeScript/JavaScript

## Quick Start

1. Choose your preferred language version
2. Follow the day-by-day structure
3. Complete the hands-on exercises
4. Reference the architecture sections as needed
5. Use the debugging tips when exploring code

## Contributing

If you find issues or want to improve these guides:

1. Open an issue describing the improvement
2. Submit a pull request with your changes
3. Follow the [Contributing Guidelines](./CONTRIBUTING.md)

## Additional Resources

- [Label Studio Documentation](https://labelstud.io/guide/)
- [API Documentation](https://labelstud.io/api/)
- [GitHub Repository](https://github.com/HumanSignal/label-studio)
- [Slack Community](https://slack.labelstud.io/)

## License

These guides are part of the Label Studio project and are licensed under the Apache License 2.0.

---

**Happy Learning! 🚀**
