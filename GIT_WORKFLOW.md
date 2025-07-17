# Git Workflow Standards

Standardized Git practices, commit conventions, and branching strategies for consistent development.

## Overview

This document establishes Git workflow standards that promote clear history, collaborative development, and maintainable codebases. These practices ensure that every commit tells a story and the repository history remains clean and useful.

## Commit Message Standards

### Format Structure
```
Brief descriptive title (50 characters max)

Detailed description:
- Feature 1: What it does and why
- Feature 2: Technical implementation details
- Feature 3: User experience improvements

Technical implementation:
- File changes made
- Dependencies added
- Configuration updates

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Title Guidelines
- **50 characters maximum**
- **Start with imperative verb** (Add, Fix, Update, Remove, etc.)
- **No period at the end**
- **Be specific and descriptive**

#### Good Examples:
- `Add interactive map-based location selection`
- `Fix HAML syntax error in admin dashboard`
- `Update user authentication to support 2FA`
- `Remove deprecated payment gateway integration`

#### Poor Examples:
- `Fixed stuff` (too vague)
- `Updated the system to handle user authentication better` (too long)
- `fix bug` (not descriptive, wrong capitalization)

### Description Guidelines
- **Explain the why, not just the what**
- **Use bullet points for multiple changes**
- **Include technical details**
- **Mention breaking changes**
- **Reference issues when applicable**

### Example Commit Messages

#### Feature Implementation
```
Add quest creation wizard for better UX

Features added:
- 4-step wizard interface: Basic Info → Rewards → Locations → Review
- Visual progress bar showing completion status
- Step indicators with clickable navigation
- Real-time form validation with error feedback
- Smart navigation (can only advance when step is valid)

Technical implementation:
- QuestWizard JavaScript class for wizard logic
- Bootstrap toast integration for notifications
- Form validation with detailed error messages
- CSS animations for smooth transitions

User experience improvements:
- Reduces cognitive load by breaking complex form into steps
- Provides clear visual feedback on progress
- Prevents errors with validation at each step
- Mobile-responsive design for all devices

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

#### Bug Fix
```
Fix HAML syntax error causing admin dashboard 500 error

Issue:
- Admin dashboard was returning 500 errors due to HAML syntax
- stylesheet_link_tag was using incorrect hash syntax
- Asset pipeline couldn't resolve the malformed stylesheet reference

Solution:
- Updated stylesheet_link_tag to use proper HAML syntax
- Changed from Rails helper to direct link tag
- Added proper importmap configuration for admin layout

Technical changes:
- Updated app/views/layouts/admin.html.haml
- Fixed asset pipeline integration
- Tested page loading to ensure no template errors

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

#### Testing Addition
```
Add comprehensive test coverage for quest creation

Test coverage added:
- Model tests for Quest validation and associations
- Controller tests for QuestsController CRUD operations
- Feature tests for quest creation wizard workflow
- JavaScript tests for map interaction functionality

Technical implementation:
- RSpec tests with Factory Bot for test data
- Capybara feature tests with JavaScript support
- SimpleCov configuration for coverage reporting
- Database cleaner setup for test isolation

Coverage achieved:
- 95% overall test coverage
- 100% coverage for critical quest creation logic
- All edge cases and error conditions tested
- Manual testing protocol documented

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Branching Strategy

### Main Branch Strategy
- **Main branch**: Always deployable, production-ready code
- **Direct commits**: For straightforward features and bug fixes
- **Feature branches**: For complex features or experimental work

### Branch Naming Conventions
When using feature branches:
- `feature/quest-creation-wizard`
- `bugfix/admin-dashboard-haml-error`
- `enhancement/map-performance-optimization`
- `refactor/user-authentication-cleanup`

### Branch Management
```bash
# Create feature branch
git checkout -b feature/quest-participation-workflow

# Work on feature with regular commits
git add .
git commit -m "Add quest participation model and associations"

# Push branch
git push -u origin feature/quest-participation-workflow

# Merge back to main (after testing)
git checkout main
git merge feature/quest-participation-workflow
git push origin main

# Clean up feature branch
git branch -d feature/quest-participation-workflow
git push origin --delete feature/quest-participation-workflow
```

## Commit Frequency and Size

### Commit Guidelines
- **One feature per commit**: Each commit should represent a complete, working feature
- **Atomic commits**: Each commit should be self-contained and not break functionality
- **Regular commits**: Commit frequently to avoid losing work
- **Meaningful commits**: Every commit should add value and be deployable

### What Makes a Good Commit
- **Single responsibility**: Addresses one feature or bug fix
- **Complete functionality**: Feature works as intended
- **Tests included**: Comprehensive test coverage
- **Documentation updated**: Relevant docs reflect changes
- **No broken state**: Code compiles and runs without errors

### Commit Size Examples

#### Good Commit Size:
```
Add address search functionality to quest creation

- OpenStreetMap Nominatim API integration
- Real-time search with clickable results
- Error handling for API failures
- Loading states and user feedback
- Tests for search functionality
```

#### Too Large (should be split):
```
Add complete quest management system

- Quest creation with wizard interface
- Quest editing and deletion
- Quest participation workflow
- Admin dashboard for quest management
- User authentication and authorization
- Payment integration for quest rewards
- Email notifications for quest events
```

#### Too Small (should be combined):
```
Fix typo in quest model
```

## Code Quality Before Commit

### Pre-Commit Checklist
- [ ] **Code compiles without errors**
- [ ] **All tests pass**
- [ ] **Test coverage meets requirements (90%+)**
- [ ] **Code follows style guidelines**
- [ ] **No debugging code left in**
- [ ] **Documentation updated**
- [ ] **Security review completed**

### Testing Before Commit
```bash
# Run all tests
bundle exec rspec

# Check test coverage
bundle exec rspec --coverage

# Run linter (if configured)
bundle exec rubocop

# Test that Rails application loads
bundle exec rails runner "puts 'App loads successfully'"
```

## Collaboration Workflow

### Working with Others
- **Pull before pushing**: Always pull latest changes before pushing
- **Communicate changes**: Let team know about significant changes
- **Review commits**: Review your commits before pushing
- **Consistent style**: Follow established commit message format

### Merge Conflicts
```bash
# Pull latest changes
git pull origin main

# Resolve conflicts in files
# Edit conflicted files and resolve conflicts

# Mark conflicts as resolved
git add .

# Complete merge
git commit -m "Resolve merge conflicts with main branch"

# Push resolved changes
git push origin main
```

## Release and Deployment

### Release Preparation
- **Tag releases**: Use semantic versioning for releases
- **Release notes**: Document changes for each release
- **Deployment checklist**: Ensure all requirements are met
- **Rollback plan**: Have a plan for reverting if needed

### Tagging Releases
```bash
# Create annotated tag
git tag -a v1.0.0 -m "Release version 1.0.0: Quest creation wizard and map integration"

# Push tag
git push origin v1.0.0

# List tags
git tag -l
```

### Semantic Versioning
- **Major (1.0.0)**: Breaking changes, major new features
- **Minor (1.1.0)**: New features, backward compatible
- **Patch (1.1.1)**: Bug fixes, no new features

## Git Configuration

### Recommended Git Settings
```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"

# Set default branch name
git config --global init.defaultBranch main

# Enable helpful colorization
git config --global color.ui auto

# Set pull strategy
git config --global pull.rebase false
```

### Git Aliases
```bash
# Useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## Advanced Git Techniques

### Interactive Rebase
```bash
# Clean up commits before pushing
git rebase -i HEAD~3

# Squash commits, reorder, or edit messages
# Use with caution - only for unpushed commits
```

### Cherry Picking
```bash
# Apply specific commit to current branch
git cherry-pick <commit-hash>

# Apply multiple commits
git cherry-pick <commit1> <commit2>
```

### Stashing Work
```bash
# Save work in progress
git stash

# List stashes
git stash list

# Apply stash
git stash apply

# Drop stash
git stash drop
```

## Troubleshooting Common Issues

### Undoing Changes
```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Undo changes to specific file
git checkout HEAD -- filename

# Revert published commit
git revert <commit-hash>
```

### Fixing Commit Messages
```bash
# Fix last commit message
git commit --amend -m "New commit message"

# Fix author information
git commit --amend --author="New Author <new@email.com>"
```

### Removing Files
```bash
# Remove file from git but keep locally
git rm --cached filename

# Remove file completely
git rm filename

# Remove directory
git rm -r directory/
```

## Security Considerations

### What Not to Commit
- **Passwords and API keys**
- **Database credentials**
- **Private keys and certificates**
- **Local configuration files**
- **Temporary files and logs**
- **IDE-specific files**

### .gitignore Best Practices
```gitignore
# Secrets
.env
.env.local
config/database.yml
config/secrets.yml

# Dependencies
node_modules/
vendor/bundle/

# Logs
*.log
log/

# Temporary files
tmp/
*.tmp
*.swp

# OS files
.DS_Store
Thumbs.db

# IDE files
.vscode/
.idea/
*.sublime-*
```

### Removing Sensitive Data
```bash
# Remove file from entire history (use with caution)
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch path/to/file' --prune-empty --tag-name-filter cat -- --all

# Alternative using BFG Repo-Cleaner
java -jar bfg.jar --delete-files filename repo.git
```

## Continuous Integration

### GitHub Actions Integration
```yaml
# .github/workflows/test.yml
name: Test and Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Ruby
      uses: ruby/setup-ruby@v1
      with:
        bundler-cache: true
    
    - name: Run tests
      run: bundle exec rspec
    
    - name: Check commit format
      run: |
        git log --oneline -1 | grep -E "^[A-Z][a-z]+"
```

## Best Practices Summary

### Do's
- **Write clear, descriptive commit messages**
- **Commit frequently with complete features**
- **Test before every commit**
- **Keep commits atomic and focused**
- **Use consistent formatting**
- **Pull before pushing**
- **Tag releases properly**

### Don'ts
- **Don't commit broken code**
- **Don't commit sensitive information**
- **Don't rewrite published history**
- **Don't commit generated files**
- **Don't use vague commit messages**
- **Don't mix unrelated changes**
- **Don't commit without testing**

---

**Remember**: Every commit should tell a story and leave the codebase in a better state than you found it.