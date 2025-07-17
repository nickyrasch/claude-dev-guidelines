# Collaboration Guidelines for HeartQuest Development

## Overview
This document establishes our working relationship patterns, expectations, and best practices for collaborative development on the HeartQuest project.

## Development Philosophy

### Core Principles
1. **Incremental Progress**: Build features step-by-step with regular commits
2. **User-Centric Design**: Always prioritize user experience and accessibility
3. **Code Quality**: Maintain clean, readable, and well-documented code
4. **Security First**: Implement defensive security practices throughout
5. **Performance Awareness**: Consider performance implications of all changes

### Technical Preferences
- **Ruby on Rails**: Leverage Rails conventions and best practices
- **Progressive Enhancement**: Features should work without JavaScript, enhanced with it
- **Mobile-First**: Design for mobile devices first, then enhance for desktop
- **Free/Open Source**: Prefer free solutions over paid APIs when possible
- **Accessibility**: Ensure features work with screen readers and keyboard navigation

## Communication Style

### My Preferred Working Style
- **Direct and Concise**: I appreciate clear, actionable communication
- **Context-Aware**: Reference specific files, line numbers, and existing patterns
- **Solution-Oriented**: Focus on implementation details rather than abstract concepts
- **Realistic Scope**: Break large features into manageable, testable chunks

### Request Format Preferences
✅ **Good**: "Add user search functionality to the admin dashboard with filters by role and export to CSV"
✅ **Good**: "Fix the HAML syntax error in admin layout - the stylesheet_link_tag is causing a 500 error"
❌ **Less Helpful**: "Make the admin better"
❌ **Less Helpful**: "Add some improvements to the user interface"

## Project Management

### Task Organization
- **Use TodoWrite Tool**: Always create and maintain todo lists for multi-step tasks
- **Commit After Each Feature**: One feature per commit with descriptive messages
- **Push Regularly**: Push to remote after each completed feature
- **Test Before Committing**: Ensure Rails app loads without errors before commits

### Feature Development Process
1. **Planning**: Break down feature into specific, testable components
2. **Implementation**: Write code following Rails conventions
3. **Testing**: Verify functionality works as expected
4. **Documentation**: Update project specs and add comments where needed
5. **Commit**: Clear commit message describing what was implemented
6. **Push**: Push to remote repository

## Testing Requirements

### Test-Driven Development Approach
All features must have comprehensive test coverage before being considered complete.

### RSpec Testing Standards

#### Required Test Types
1. **Model Tests** (`spec/models/`)
   - Validations and associations
   - Instance methods and scopes
   - Business logic and edge cases

2. **Controller Tests** (`spec/controllers/` or `spec/requests/`)
   - HTTP status codes and redirects
   - Authentication and authorization
   - Parameter handling and validation

3. **View Tests** (`spec/views/`)
   - Template rendering without errors
   - Presence of key elements and forms
   - Conditional display logic

4. **Feature Tests** (`spec/features/`)
   - End-to-end user workflows
   - JavaScript interactions
   - Form submissions and validations

5. **JavaScript Tests** (when applicable)
   - Interactive map functionality
   - Wizard step navigation
   - AJAX requests and responses

#### Test Coverage Requirements
- **Minimum 90% coverage** for all new code
- **100% coverage** for critical business logic
- **All edge cases** must be tested
- **Error conditions** must be tested

### Page Testing Protocol

#### Manual Page Testing Checklist
For every page/view created or modified:
- [ ] Page loads without 500 errors
- [ ] HAML syntax is valid (no template errors)
- [ ] All forms render correctly
- [ ] JavaScript functionality works
- [ ] Mobile responsiveness works
- [ ] Accessibility features work (keyboard navigation, screen readers)

#### Common Issues to Test For
- **HAML Syntax**: Check for proper indentation and syntax
- **Asset Loading**: Ensure CSS/JS files load correctly
- **Form Validation**: Test both client-side and server-side validation
- **Authentication**: Verify login/logout functionality
- **Authorization**: Test user permissions and access controls
- **Mobile Layout**: Check responsive design on different screen sizes

### Testing Tools and Commands

#### Running Tests
```bash
# Run all tests
bundle exec rspec

# Run specific test file
bundle exec rspec spec/models/quest_spec.rb

# Run tests with coverage
bundle exec rspec --coverage

# Run feature tests (including JavaScript)
bundle exec rspec spec/features/
```

#### Test Database Management
```bash
# Reset test database
bundle exec rake db:test:prepare

# Run migrations on test database
bundle exec rake db:migrate RAILS_ENV=test
```

### Test Writing Guidelines

#### Model Testing Example
```ruby
# spec/models/quest_spec.rb
RSpec.describe Quest, type: :model do
  describe 'validations' do
    it 'requires a title' do
      quest = build(:quest, title: nil)
      expect(quest).not_to be_valid
      expect(quest.errors[:title]).to include("can't be blank")
    end
  end

  describe 'associations' do
    it 'belongs to a creator' do
      association = Quest.reflect_on_association(:creator)
      expect(association.macro).to eq(:belongs_to)
    end
  end
end
```

#### Controller Testing Example
```ruby
# spec/controllers/quests_controller_spec.rb
RSpec.describe QuestsController, type: :controller do
  describe 'GET #new' do
    context 'when user is authenticated' do
      before { sign_in user }
      
      it 'returns success status' do
        get :new
        expect(response).to have_http_status(:success)
      end
      
      it 'renders the new template' do
        get :new
        expect(response).to render_template(:new)
      end
    end
  end
end
```

#### Feature Testing Example
```ruby
# spec/features/quest_creation_spec.rb
RSpec.describe 'Quest Creation', type: :feature, js: true do
  scenario 'User creates a quest with map interface' do
    sign_in user
    visit new_quest_path
    
    fill_in 'Title', with: 'My Test Quest'
    fill_in 'Description', with: 'A test quest description'
    
    # Test map interaction
    find('#set-start-mode').click
    # Additional map testing would go here
    
    click_button 'Create Quest'
    expect(page).to have_text('Quest was successfully created')
  end
end
```

## CI/CD and Code Quality

### GitHub Actions Integration
All code changes must pass automated quality checks before being considered complete.

#### Required CI/CD Checks
1. **RuboCop Linting** (`bin/rubocop -f github`)
   - Zero style offenses allowed
   - Consistent code formatting enforced
   - Auto-fixable issues should be corrected with `bundle exec rubocop --autocorrect`
   
2. **Brakeman Security Scan** (`bin/brakeman --no-pager`)
   - Zero security vulnerabilities allowed
   - Special attention to mass assignment vulnerabilities
   - Review any parameter whitelisting carefully
   
3. **JavaScript Dependency Audit** (`bin/importmap audit`)
   - No vulnerable JavaScript packages
   - Keep dependencies up to date
   - Use free, well-maintained libraries

#### Pre-Commit Quality Checklist
Before committing any code:
- [ ] Run `bundle exec rubocop` and fix all offenses
- [ ] Run `bundle exec brakeman` and address security warnings
- [ ] Run `bin/importmap audit` to verify JS dependencies
- [ ] Ensure all tests pass (`bundle exec rspec`)
- [ ] Manually test affected pages for HAML/syntax errors

#### Common Quality Issues to Avoid
1. **Mass Assignment Security**: Never allow sensitive parameters (like `:role`) in mass assignment
   ```ruby
   # ❌ DANGEROUS - allows privilege escalation
   params.require(:user).permit(:first_name, :last_name, :email, :role)
   
   # ✅ SECURE - use dedicated action for role changes
   params.require(:user).permit(:first_name, :last_name, :email)
   ```

2. **Code Style Consistency**: 
   - Use double quotes for strings (RuboCop enforced)
   - Proper spacing in array literals: `[ item1, item2 ]`
   - Remove trailing whitespace
   - Add final newlines to all files

3. **Security Best Practices**:
   - Use strong parameters for all form inputs
   - Implement proper authorization checks with CanCanCan
   - Validate all user inputs
   - Use HTTPS and secure cookies in production

### Code Quality Standards
All code must meet these quality standards:

#### Immediate Failure Conditions
These issues will cause CI to fail and must be fixed:
- Any RuboCop style violations
- Any Brakeman security warnings
- Any vulnerable JavaScript dependencies
- Test failures
- HAML syntax errors causing 500 responses

#### Quality Maintenance Commands
```bash
# Fix auto-correctable style issues
bundle exec rubocop --autocorrect

# Check for security vulnerabilities
bundle exec brakeman --quiet

# Audit JavaScript dependencies
bin/importmap audit

# Run full test suite
bundle exec rspec

# Test app loads without errors
bundle exec rails runner "puts 'App loads successfully'"
```

## Code Standards

### Rails Conventions
- **Follow Rails Way**: Use Rails generators, conventions, and patterns
- **HAML Templates**: Use proper HAML syntax and indentation
- **Stimulus Controllers**: Prefer Stimulus over vanilla JavaScript for interactions
- **CSS Classes**: Use Bootstrap classes first, custom CSS sparingly
- **Database**: Use PostgreSQL-specific features when beneficial

### JavaScript Patterns
- **ES6+ Syntax**: Use modern JavaScript features (classes, async/await, etc.)
- **Modular Design**: Create reusable classes and functions
- **Error Handling**: Implement comprehensive error handling with user feedback
- **Performance**: Minimize DOM manipulation and use efficient event handling

### File Organization
- **Logical Structure**: Group related functionality in appropriate directories
- **Naming Conventions**: Use clear, descriptive file and class names
- **Partial Views**: Break complex views into manageable partials
- **Shared Components**: Extract reusable components when appropriate

## Git Workflow

### Commit Message Format
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

### Branch Strategy
- **Main Branch**: Always keep main branch deployable
- **Feature Branches**: Create feature branches for complex changes (optional)
- **Direct Commits**: For straightforward features, commit directly to main
- **Regular Pushes**: Push after each completed feature

## Problem-Solving Approach

### When I Encounter Issues
1. **Read Error Messages**: Carefully analyze error logs and stack traces
2. **Check Existing Code**: Look for similar patterns in the codebase
3. **Test Incrementally**: Verify each change works before moving to the next
4. **Document Solutions**: Add comments explaining complex logic
5. **Ask for Context**: Request clarification on unclear requirements

### Error Handling Priorities
1. **Fix Immediately**: Syntax errors, missing dependencies, broken functionality
2. **Security Issues**: Any potential security vulnerabilities
3. **User Experience**: Confusing interfaces, poor error messages
4. **Performance**: Slow loading, inefficient queries
5. **Code Quality**: Refactoring, documentation improvements

## Session Management

### Standard Session Workflow
Every conversation should follow this structured 5-step process:

#### 1. **Planning Phase** 🎯
- **Project Discussion**: Review project plan and identify what we want to build
- **Feature Clarification**: Discuss requirements, user stories, and technical approach
- **Scope Definition**: Define clear boundaries for the session
- **Priority Setting**: Determine what's most important to accomplish

#### 2. **Todo List Creation** 📋
- **Break Down Features**: Convert discussion into specific, actionable tasks
- **Use TodoWrite Tool**: Create structured todo list with priorities
- **Estimate Complexity**: Identify which tasks might be challenging
- **Sequence Planning**: Order tasks logically for efficient development

#### 3. **Development Phase** 🛠️
- **Follow Todo List**: Work through tasks systematically
- **Mark Progress**: Update todo status as work progresses (pending → in_progress → completed)
- **Write Tests First**: Create RSpec tests for all new functionality
- **Test Incrementally**: Verify each feature works before moving to next
- **Page Testing**: Actually load and test pages in browser to catch HAML/syntax issues
- **Handle Blockers**: Address issues immediately, update todos if needed

#### 4. **Code Management** 💾
- **Quality Checks First**: Run RuboCop, Brakeman, and tests before committing
- **Commit After Each Feature**: One feature per commit with descriptive messages
- **CI/CD Compliance**: Ensure all automated checks pass
- **Push Regularly**: Push to remote after each completed feature
- **Quality Check**: Ensure Rails app loads without errors before commits
- **Clean History**: Maintain clear, readable commit history

#### 5. **Documentation Update** 📝
- **Update Project Plan**: Add completed features to project_spec.md
- **Reflect New Architecture**: Document any new files, dependencies, or patterns
- **Note Technical Decisions**: Record important implementation choices
- **Prepare for Next Session**: Identify logical next steps

### Session Quality Checkpoints

#### Before Starting Development
- [ ] Clear understanding of what we're building
- [ ] Realistic scope for the session
- [ ] Todo list with specific, actionable items
- [ ] Technical approach agreed upon

#### During Development
- [ ] Regular progress updates
- [ ] Todos marked as in_progress/completed
- [ ] RSpec tests written for all new functionality
- [ ] Features tested as they're built (both automated and manual)
- [ ] Pages loaded and tested for HAML syntax errors
- [ ] Code committed after each completed feature

#### Before Ending Session
- [ ] All features working as expected
- [ ] Full test coverage achieved (90%+ for new code)
- [ ] All pages load without 500 errors
- [ ] Manual testing checklist completed
- [ ] All CI/CD checks passing (RuboCop, Brakeman, importmap audit)
- [ ] Code committed and pushed
- [ ] GitHub Actions pipeline passing (green checkmarks)
- [ ] Project plan updated with new features
- [ ] Clear handoff notes for next session

### Error Handling Protocol

#### If Development Gets Blocked
1. **Immediate Assessment**: Clearly identify the blocker
2. **Scope Adjustment**: Modify todos to work around the issue
3. **Alternative Approach**: Propose different technical solutions
4. **Transparent Communication**: Keep you informed of all challenges

#### If Session Scope Is Too Large
1. **Priority Triage**: Identify most important features
2. **Phase Planning**: Break into multiple sessions
3. **Minimum Viable Features**: Focus on core functionality first
4. **Future Session Setup**: Plan logical continuation points

### Handoff Protocol

#### Information I'll Provide at Session End
- **Completed Features**: What was accomplished
- **Technical Changes**: New files, dependencies, or patterns added
- **Known Issues**: Any bugs or limitations discovered
- **Next Steps**: Recommended priorities for future sessions
- **Context Notes**: Important decisions or learnings for next time

#### What Helps Future Sessions
- **Updated Project Plan**: Reflects current state accurately
- **Clean Commit History**: Easy to understand what was done
- **Working Application**: No broken functionality left behind
- **Clear Documentation**: Future context is preserved

## Feedback and Iteration

### What Helps Me Work Better
- **Specific Examples**: Show me exactly what you want to change
- **Context About Users**: Explain how features will be used
- **Technical Constraints**: Let me know about any limitations or requirements
- **Priority Guidance**: Help me understand what's most important

### How to Provide Feedback
- **Reference Specific Files**: Point to exact locations that need changes
- **Explain the "Why"**: Help me understand the reasoning behind requests
- **Suggest Improvements**: Share ideas for better approaches
- **Acknowledge Good Work**: Let me know when something works well

## Tools and Environment

### Expected Tools
- **Rails Environment**: Ruby on Rails with PostgreSQL
- **Git**: Version control with regular commits
- **Text Editor**: Any editor with HAML and Ruby support
- **Browser**: For testing web interface
- **Terminal**: For running Rails commands and git operations

### Development Commands I Use
- `bundle exec rails runner "puts 'Rails application loaded successfully'"` - Test app loads
- `bundle exec rubocop --autocorrect` - Fix code style issues
- `bundle exec brakeman --quiet` - Check for security vulnerabilities
- `bin/importmap audit` - Verify JavaScript dependencies are secure
- `bundle exec rspec` - Run full test suite
- `git add . && git commit -m "message" && git push` - Commit and push changes
- `tail -f log/development.log` - Monitor Rails logs
- `bundle install` - Install gem dependencies

## Security Considerations

### What I Always Check
- **Input Validation**: All user inputs are properly validated
- **Authentication**: Proper user authentication and session management
- **Authorization**: Users can only access what they're allowed to
- **SQL Injection**: Use parameterized queries and Rails conventions
- **XSS Prevention**: Proper output escaping in templates

### What I Won't Do
- **Malicious Code**: I won't create or improve code for malicious purposes
- **Security Vulnerabilities**: I won't intentionally introduce security flaws
- **Data Exposure**: I won't create code that exposes sensitive information
- **Bypass Security**: I won't help circumvent security measures

## Success Metrics

### How I Measure Success
- **Functionality**: Features work as specified
- **User Experience**: Interface is intuitive and accessible
- **Code Quality**: Code is clean, maintainable, and follows conventions
- **Performance**: Features load quickly and efficiently
- **Security**: No security vulnerabilities introduced

### What Makes a Good Session
- **Clear Objectives**: We both understand what we're building
- **Steady Progress**: Regular commits showing incremental improvement
- **Working Code**: Each feature is tested and functional
- **Good Documentation**: Changes are reflected in project specs
- **Future-Ready**: Code is maintainable and extensible

## Emergency Procedures

### If Something Breaks
1. **Immediate Assessment**: Check error logs and identify the issue
2. **Rollback Option**: Be prepared to revert to last working commit
3. **Isolated Testing**: Test fixes in isolation before applying
4. **Documentation**: Document what went wrong and how it was fixed

### Communication During Issues
- **Immediate Notification**: Tell you right away if something breaks
- **Clear Error Reports**: Share exact error messages and stack traces
- **Solution Options**: Propose multiple approaches to fix the issue
- **Prevention Measures**: Suggest ways to avoid similar problems

## Continuous Improvement

### Regular Review Points
- **Monthly**: Review this guide and update based on our experience
- **After Complex Features**: Discuss what worked well and what could improve
- **Before Major Changes**: Align on approach for significant architectural changes
- **When Stuck**: If we're not making progress, step back and reassess

### Adaptation Guidelines
- **Flexibility**: This guide should evolve with our working relationship
- **Open Communication**: Always feel free to suggest improvements
- **Experimentation**: Try new approaches and evaluate their effectiveness
- **Learning**: Both of us should learn from each session

---

*This collaboration guide is a living document. Please suggest updates as our working relationship develops.*

**Recent Updates (July 17, 2025)**:
- Added comprehensive CI/CD and code quality standards
- Documented GitHub Actions integration requirements
- Added security best practices based on real-world fixes
- Updated session workflow to include quality checks
- Added pre-commit checklist for consistent code quality

**Last Updated**: July 17, 2025
**Next Review**: August 17, 2025