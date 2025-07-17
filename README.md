# Claude Development Guidelines

A comprehensive framework for collaborating with Claude Code on software development projects.

## Overview

This repository contains battle-tested guidelines, workflows, and standards for efficient and high-quality development collaboration with Claude Code. These guidelines have been refined through real-world project experience and focus on maintaining consistent quality, comprehensive testing, and clear communication.

## 🎯 Core Philosophy

- **Incremental Progress**: Build features step-by-step with regular commits
- **Test-Driven Development**: Write tests first, maintain 90%+ coverage
- **User-Centric Design**: Always prioritize user experience and accessibility
- **Security First**: Implement defensive security practices throughout
- **Documentation**: Keep project context current and accessible

## 📋 Repository Structure

```
claude-dev-guidelines/
├── README.md                      # This overview
├── COLLABORATION_GUIDE.md         # Comprehensive working relationship guide
├── TESTING_STANDARDS.md           # Testing requirements and examples
├── GIT_WORKFLOW.md                # Git practices and commit standards
├── SECURITY_CHECKLIST.md          # Security considerations
└── templates/
    ├── CLAUDE.md                  # Template for project context files
    └── project_spec_template.md   # Project specification template
```

## 🚀 Quick Start

### For New Projects

1. **Copy the CLAUDE.md template** to your project root
2. **Customize** project-specific sections (tech stack, current status, priorities)
3. **Reference these guidelines** in your project's CLAUDE.md
4. **Follow the 5-step session workflow** for consistent progress

### For Existing Projects

1. **Create a CLAUDE.md file** using our template
2. **Document current state** and implementation status
3. **Update project specifications** to match current reality
4. **Adopt testing standards** for future development

## 📖 Documentation Guide

### Essential Files

- **[COLLABORATION_GUIDE.md](COLLABORATION_GUIDE.md)** - Complete working relationship framework
- **[TESTING_STANDARDS.md](TESTING_STANDARDS.md)** - Test requirements and examples
- **[GIT_WORKFLOW.md](GIT_WORKFLOW.md)** - Git practices and commit standards
- **[SECURITY_CHECKLIST.md](SECURITY_CHECKLIST.md)** - Security considerations

### Templates

- **[templates/CLAUDE.md](templates/CLAUDE.md)** - Project context template
- **[templates/project_spec_template.md](templates/project_spec_template.md)** - Project specification template

### Examples

- **[HeartQuest CLAUDE.md](https://github.com/nickyrasch/heartquest/blob/main/CLAUDE.md)** - Real-world implementation
- **[HeartQuest project_spec.md](https://github.com/nickyrasch/heartquest/blob/main/project_spec.md)** - Example project specification

## 🔄 Standard Session Workflow

Every development session should follow this 5-step process:

1. **Planning Phase** 🎯 - Review project plan, discuss what to build
2. **Todo List Creation** 📋 - Break down features into actionable tasks
3. **Development Phase** 🛠️ - Write tests first, implement features
4. **Code Management** 💾 - Commit and push after each feature
5. **Documentation Update** 📝 - Update project specs with new features

## ✅ Quality Standards

### Testing Requirements
- **90%+ test coverage** for all new code
- **100% coverage** for critical business logic
- **Manual page testing** to prevent template errors
- **All edge cases** and error conditions tested

### Code Quality
- Follow language/framework conventions
- Implement comprehensive error handling
- Progressive enhancement (works without JavaScript)
- Mobile-first responsive design
- Security best practices

### Documentation
- Keep project context current
- Document technical decisions
- Clear commit messages
- Update specifications with new features

## 🛡️ Security Principles

- **Input Validation**: All user inputs validated and sanitized
- **Authentication**: Proper user authentication and session management
- **Authorization**: Users can only access allowed resources
- **No Secrets**: No sensitive data in logs or client-side code
- **Defensive Coding**: Assume all inputs are malicious

## 🔧 Technology Preferences

### General Principles
- **Free/Open Source**: Prefer free solutions over paid APIs
- **Established Libraries**: Use well-maintained, popular libraries
- **Progressive Enhancement**: Features work without JavaScript
- **Mobile-First**: Design for mobile devices first
- **Accessibility**: Ensure features work with assistive technologies

### Common Stacks
- **Ruby on Rails**: Use Rails conventions and patterns
- **JavaScript**: ES6+ syntax, modular design
- **CSS**: Bootstrap first, custom CSS sparingly
- **Testing**: RSpec, Capybara, Factory Bot
- **Database**: PostgreSQL with proper indexing

## 📊 Success Metrics

### Session Quality
- Clear objectives achieved
- All features tested and working
- Code committed and pushed
- Documentation updated
- Next steps identified

### Code Quality
- Comprehensive test coverage
- No broken functionality
- Follows established patterns
- Maintainable and readable
- Secure and performant

## 🔄 Continuous Improvement

This framework is designed to evolve. Regular review points:

- **Monthly**: Review guidelines and update based on experience
- **After Complex Features**: Discuss what worked well and what could improve
- **Before Major Changes**: Align on approach for significant changes
- **When Blocked**: Step back and reassess if progress stalls

## 🤝 Contributing

These guidelines are living documents. Improvements should be:

- **Tested**: Tried in real development scenarios
- **Documented**: Clear explanations and examples
- **Consistent**: Aligned with existing principles
- **Practical**: Focused on real-world implementation

## 📞 Support

For questions or suggestions about these guidelines:
- Review the comprehensive guides in this repository
- Check the examples for real-world implementations
- Suggest improvements through clear, specific feedback

---

**Version**: 1.0.0  
**Last Updated**: July 17, 2025  
**Based on**: HeartQuest project development experience  
**Next Review**: August 17, 2025