# [Project Name] Development Context

## Project Overview
[Brief description of what the project does and its main purpose]

## Global Development Guidelines
This project follows comprehensive development standards documented at:
https://github.com/[username]/claude-dev-guidelines

**Key Standards:**
- **5-Step Session Workflow**: Planning → Todo List → Development → Code Management → Documentation Update
- **Testing Requirements**: 90%+ test coverage, comprehensive test types
- **Security First**: Input validation, output sanitization, authentication/authorization
- **Git Standards**: Clear commit messages, atomic commits, regular pushes

## Technical Stack
- **Language/Framework**: [e.g., Ruby on Rails, Node.js, Python Django]
- **Database**: [e.g., PostgreSQL, MySQL, MongoDB]
- **Frontend**: [e.g., React, Vue, HAML/Bootstrap]
- **Testing**: [e.g., RSpec, Jest, Pytest]
- **Authentication**: [e.g., Devise, Auth0, Firebase Auth]
- **Deployment**: [e.g., Heroku, AWS, Docker]

## Current Implementation Status

### Recently Completed Features
1. **[Feature Name]**
   - [Brief description]
   - [Technical implementation notes]
   - [User impact]

2. **[Feature Name]**
   - [Brief description]
   - [Technical implementation notes]
   - [User impact]

### Key Files Structure
```
[project-root]/
├── [main-directory]/
│   ├── [key-file-1]     # Description
│   ├── [key-file-2]     # Description
│   └── [key-file-3]     # Description
├── [config-directory]/
│   ├── [config-file-1]  # Description
│   └── [config-file-2]  # Description
└── [other-directories]/
    └── [important-files] # Description
```

## Development Standards

### Testing Requirements (CRITICAL)
- **Write tests for ALL new functionality**
- **90%+ test coverage** for new code, 100% for critical business logic
- **Manual page testing** to prevent template/syntax errors
- **Required test types**: [unit, integration, feature, etc.]

### Code Quality Standards
- Follow [language/framework] conventions and patterns
- Use proper [template engine] syntax and indentation
- Implement comprehensive error handling
- Progressive enhancement (works without JavaScript)
- Mobile-first responsive design

### Git Workflow
- One feature per commit with descriptive messages
- Test before committing (ensure application loads)
- Push after each completed feature
- Update project documentation with new features

## Session Management Protocol

### Every Session Should Follow:
1. **Planning Phase**: Review project plan, discuss what to build
2. **Todo List Creation**: Break down features into actionable tasks
3. **Development Phase**: Write tests first, then implement features
4. **Code Management**: Commit and push after each feature
5. **Documentation Update**: Update project specs with new features

### Quality Checkpoints
- [ ] All pages load without errors
- [ ] [Template engine] syntax is valid (no template errors)
- [ ] Full test coverage achieved
- [ ] Features work on mobile devices
- [ ] [Framework-specific functionality] tested

## Current Development Priorities

### Next Logical Features
1. **[Priority 1 Feature]**
   - [Description]
   - [Technical requirements]
   - [User story]

2. **[Priority 2 Feature]**
   - [Description]
   - [Technical requirements]
   - [User story]

3. **[Priority 3 Feature]**
   - [Description]
   - [Technical requirements]
   - [User story]

### Technical Debt Items
- [Technical debt item 1]
- [Technical debt item 2]
- [Technical debt item 3]

## Security Considerations
- All user inputs are validated and sanitized
- Authentication required for [protected actions]
- Authorization checks for [resource access]
- No sensitive data in logs or client-side code
- [Project-specific security requirements]

## Database Schema Notes
- **[Model 1]**: [relationships and key fields]
- **[Model 2]**: [relationships and key fields]
- **[Model 3]**: [relationships and key fields]

## Dependencies
- **[Key Dependency 1]**: [Purpose and integration method]
- **[Key Dependency 2]**: [Purpose and integration method]
- **[Key Dependency 3]**: [Purpose and integration method]

## Known Issues & Limitations
- [Known issue 1 and status]
- [Known issue 2 and status]
- [Limitation 1 and potential solutions]

## API Integrations
- **[API Name 1]**: [Purpose, authentication, endpoints used]
- **[API Name 2]**: [Purpose, authentication, endpoints used]

## Environment Configuration
- **Development**: [Key settings and requirements]
- **Test**: [Key settings and requirements]  
- **Production**: [Key settings and requirements]

## Success Metrics
- Features work as specified with comprehensive test coverage
- Pages load quickly without errors
- User experience is intuitive and accessible
- Code is maintainable and follows [framework] conventions
- Mobile experience is seamless
- [Project-specific success criteria]

## Troubleshooting Common Issues
- **Issue 1**: [Common problem and solution]
- **Issue 2**: [Common problem and solution]
- **Issue 3**: [Common problem and solution]

---

**Remember**: Always write tests first, test pages manually for template errors, and maintain our 5-step session workflow for consistent progress.

*Last updated: [Date]*