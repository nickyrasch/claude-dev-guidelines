# Testing Standards

Comprehensive testing requirements and best practices for all Claude Code development projects.

## Overview

Testing is mandatory for all features and must achieve 90%+ coverage for new code, with 100% coverage for critical business logic. This document provides standards, examples, and tools to ensure consistent testing quality.

## Test-Driven Development Approach

All features must have comprehensive test coverage before being considered complete. Tests should be written first, then implementation follows.

## Required Test Types

### 1. Model Tests (`spec/models/`)
- **Validations**: Test all model validations
- **Associations**: Verify relationships between models
- **Scopes**: Test query scopes and custom methods
- **Business Logic**: Test all instance and class methods
- **Edge Cases**: Test boundary conditions and error states

### 2. Controller Tests (`spec/controllers/` or `spec/requests/`)
- **HTTP Status Codes**: Verify correct response codes
- **Redirects**: Test redirect logic and destinations
- **Authentication**: Test login requirements
- **Authorization**: Test user permissions and access controls
- **Parameter Handling**: Test strong parameters and validation

### 3. View Tests (`spec/views/`)
- **Template Rendering**: Ensure templates render without errors
- **Form Elements**: Test presence of required form fields
- **Conditional Display**: Test conditional content rendering
- **Helper Methods**: Test view helper functionality
- **Accessibility**: Test ARIA labels and semantic HTML

### 4. Feature Tests (`spec/features/`)
- **User Workflows**: Test complete user journeys
- **JavaScript Interactions**: Test dynamic functionality
- **Form Submissions**: Test form validation and submission
- **Navigation**: Test page navigation and links
- **Error Handling**: Test error states and recovery

### 5. JavaScript Tests (when applicable)
- **Interactive Components**: Test map functionality, wizards, etc.
- **AJAX Requests**: Test asynchronous operations
- **User Interactions**: Test click handlers, form interactions
- **Error Handling**: Test client-side error scenarios
- **Mobile Responsiveness**: Test on different screen sizes

## Test Coverage Requirements

### Coverage Thresholds
- **Minimum 90% coverage** for all new code
- **100% coverage** for critical business logic (payments, security, data integrity)
- **All edge cases** must be tested
- **Error conditions** must be tested

### Coverage Tools
```bash
# Run tests with coverage report
bundle exec rspec --coverage

# Generate detailed coverage report
bundle exec rspec spec/ --coverage --format html
```

## Testing Tools and Commands

### Running Tests
```bash
# Run all tests
bundle exec rspec

# Run specific test file
bundle exec rspec spec/models/quest_spec.rb

# Run tests with coverage
bundle exec rspec --coverage

# Run feature tests (including JavaScript)
bundle exec rspec spec/features/

# Run tests in parallel (for large test suites)
bundle exec parallel_rspec spec/
```

### Test Database Management
```bash
# Reset test database
bundle exec rake db:test:prepare

# Run migrations on test database
bundle exec rake db:migrate RAILS_ENV=test

# Seed test database
bundle exec rake db:seed RAILS_ENV=test
```

## Test Writing Guidelines

### Model Testing Example
```ruby
# spec/models/quest_spec.rb
RSpec.describe Quest, type: :model do
  let(:user) { create(:user) }
  let(:quest) { build(:quest, creator: user) }

  describe 'validations' do
    it 'is valid with valid attributes' do
      expect(quest).to be_valid
    end

    it 'requires a title' do
      quest.title = nil
      expect(quest).not_to be_valid
      expect(quest.errors[:title]).to include("can't be blank")
    end

    it 'requires title to be under 255 characters' do
      quest.title = 'a' * 256
      expect(quest).not_to be_valid
      expect(quest.errors[:title]).to include('is too long (maximum is 255 characters)')
    end

    it 'requires start coordinates' do
      quest.start_latitude = nil
      quest.start_longitude = nil
      expect(quest).not_to be_valid
      expect(quest.errors[:start_latitude]).to include("can't be blank")
      expect(quest.errors[:start_longitude]).to include("can't be blank")
    end
  end

  describe 'associations' do
    it 'belongs to a creator' do
      association = Quest.reflect_on_association(:creator)
      expect(association.macro).to eq(:belongs_to)
      expect(association.class_name).to eq('User')
    end

    it 'has many checkpoints' do
      association = Quest.reflect_on_association(:checkpoints)
      expect(association.macro).to eq(:has_many)
      expect(association.options[:dependent]).to eq(:destroy)
    end
  end

  describe 'scopes' do
    let!(:private_quest) { create(:quest, visibility: 'private') }
    let!(:public_quest) { create(:quest, visibility: 'public') }

    it 'returns only public quests' do
      expect(Quest.public_quests).to include(public_quest)
      expect(Quest.public_quests).not_to include(private_quest)
    end
  end

  describe 'instance methods' do
    let(:quest) { create(:quest) }
    let(:user) { create(:user) }

    describe '#completed_by?' do
      it 'returns false when user has not completed quest' do
        expect(quest.completed_by?(user)).to be false
      end

      it 'returns true when user has completed quest' do
        create(:participation, quest: quest, user: user, completed_at: Time.current)
        expect(quest.completed_by?(user)).to be true
      end
    end
  end
end
```

### Controller Testing Example
```ruby
# spec/controllers/quests_controller_spec.rb
RSpec.describe QuestsController, type: :controller do
  let(:user) { create(:user) }
  let(:quest) { create(:quest, creator: user) }

  describe 'GET #index' do
    it 'returns success status' do
      get :index
      expect(response).to have_http_status(:success)
    end

    it 'assigns public quests' do
      public_quest = create(:quest, visibility: 'public')
      private_quest = create(:quest, visibility: 'private')
      
      get :index
      expect(assigns(:quests)).to include(public_quest)
      expect(assigns(:quests)).not_to include(private_quest)
    end
  end

  describe 'GET #show' do
    context 'when quest is public' do
      let(:quest) { create(:quest, visibility: 'public') }

      it 'returns success status' do
        get :show, params: { id: quest.id }
        expect(response).to have_http_status(:success)
      end
    end

    context 'when quest is private' do
      let(:quest) { create(:quest, visibility: 'private') }

      it 'returns not found for unauthorized user' do
        get :show, params: { id: quest.id }
        expect(response).to have_http_status(:not_found)
      end
    end
  end

  describe 'GET #new' do
    context 'when user is authenticated' do
      before { sign_in user }

      it 'returns success status' do
        get :new
        expect(response).to have_http_status(:success)
      end

      it 'assigns a new quest' do
        get :new
        expect(assigns(:quest)).to be_a_new(Quest)
      end
    end

    context 'when user is not authenticated' do
      it 'redirects to login' do
        get :new
        expect(response).to redirect_to(new_user_session_path)
      end
    end
  end

  describe 'POST #create' do
    before { sign_in user }

    context 'with valid parameters' do
      let(:valid_params) do
        {
          quest: {
            title: 'Test Quest',
            description: 'Test description',
            start_latitude: 40.7128,
            start_longitude: -74.0060,
            end_latitude: 40.7589,
            end_longitude: -73.9851
          }
        }
      end

      it 'creates a new quest' do
        expect {
          post :create, params: valid_params
        }.to change(Quest, :count).by(1)
      end

      it 'redirects to quest show page' do
        post :create, params: valid_params
        expect(response).to redirect_to(quest_path(Quest.last))
      end

      it 'sets current user as creator' do
        post :create, params: valid_params
        expect(Quest.last.creator).to eq(user)
      end
    end

    context 'with invalid parameters' do
      let(:invalid_params) do
        {
          quest: {
            title: '',
            description: 'Test description'
          }
        }
      end

      it 'does not create a quest' do
        expect {
          post :create, params: invalid_params
        }.not_to change(Quest, :count)
      end

      it 'renders new template' do
        post :create, params: invalid_params
        expect(response).to render_template(:new)
      end
    end
  end
end
```

### Feature Testing Example
```ruby
# spec/features/quest_creation_spec.rb
RSpec.describe 'Quest Creation', type: :feature, js: true do
  let(:user) { create(:user) }

  before do
    sign_in user
  end

  scenario 'User creates a quest using the wizard' do
    visit new_quest_path

    # Step 1: Basic Information
    expect(page).to have_content('Basic Information')
    expect(page).to have_css('.wizard-step.active', text: 'Basic Info')

    fill_in 'Title', with: 'My Amazing Quest'
    fill_in 'Description', with: 'This is a test quest with a detailed description'
    select 'Public', from: 'Visibility'

    click_button 'Next'

    # Step 2: Rewards
    expect(page).to have_content('Reward Settings')
    expect(page).to have_css('.wizard-step.active', text: 'Rewards')

    check 'Add a reward for completing this quest'
    fill_in 'Reward amount', with: '50.00'
    select 'USD', from: 'Currency'

    click_button 'Next'

    # Step 3: Locations
    expect(page).to have_content('Quest Locations')
    expect(page).to have_css('.wizard-step.active', text: 'Locations')

    # Test address search
    fill_in 'Search for an address', with: 'Times Square, New York'
    click_button 'Search'

    # Wait for search results
    expect(page).to have_css('#search-results')

    # Test map interaction
    click_button 'Set Start Location'
    expect(page).to have_css('.btn.active', text: 'Set Start Location')

    # Simulate clicking on map (would require more complex Capybara setup)
    fill_in 'start_lat', with: '40.7128'
    fill_in 'start_lng', with: '-74.0060'
    fill_in 'end_lat', with: '40.7589'
    fill_in 'end_lng', with: '-73.9851'

    click_button 'Next'

    # Step 4: Review
    expect(page).to have_content('Review Your Quest')
    expect(page).to have_css('.wizard-step.active', text: 'Review')

    # Verify quest details are displayed
    expect(page).to have_content('My Amazing Quest')
    expect(page).to have_content('This is a test quest')
    expect(page).to have_content('Public')
    expect(page).to have_content('USD 50.00')

    # Create the quest
    click_button 'Create Quest'

    # Verify success
    expect(page).to have_content('Quest was successfully created')
    expect(page).to have_content('My Amazing Quest')
    expect(current_path).to eq(quest_path(Quest.last))
  end

  scenario 'User cannot advance without completing required fields' do
    visit new_quest_path

    # Try to advance without filling required fields
    click_button 'Next'

    # Should show validation error
    expect(page).to have_content('Please fill in both the title and description')
    expect(page).to have_css('.wizard-step.active', text: 'Basic Info')
  end

  scenario 'User can navigate back through wizard steps' do
    visit new_quest_path

    # Fill step 1
    fill_in 'Title', with: 'Test Quest'
    fill_in 'Description', with: 'Test description'
    click_button 'Next'

    # Go to step 2
    expect(page).to have_css('.wizard-step.active', text: 'Rewards')
    
    # Navigate back
    click_button 'Previous'
    expect(page).to have_css('.wizard-step.active', text: 'Basic Info')
    
    # Verify form data is preserved
    expect(page).to have_field('Title', with: 'Test Quest')
    expect(page).to have_field('Description', with: 'Test description')
  end
end
```

## Page Testing Protocol

### Manual Testing Checklist
For every page/view created or modified:

- [ ] **Page loads without 500 errors**
- [ ] **HAML syntax is valid** (no template errors)
- [ ] **All forms render correctly**
- [ ] **JavaScript functionality works**
- [ ] **CSS styles are applied correctly**
- [ ] **Mobile responsiveness works**
- [ ] **Accessibility features work** (keyboard navigation, screen readers)
- [ ] **Error states display properly**
- [ ] **Loading states work correctly**
- [ ] **Links and navigation work**

### Common Issues to Test For

#### HAML Syntax Issues
- Proper indentation (2 spaces)
- Correct attribute syntax `{attribute: "value"}`
- Proper Ruby expression syntax `= variable`
- Correct conditional syntax `- if condition`

#### Asset Loading
- CSS files load correctly
- JavaScript files load without errors
- Images display properly
- Fonts load correctly

#### Form Validation
- Client-side validation works
- Server-side validation works
- Error messages display clearly
- Success messages appear

#### Authentication/Authorization
- Login/logout functionality works
- Protected pages require authentication
- User permissions are enforced
- Session management works correctly

## Factory Bot Usage

### Basic Factory Example
```ruby
# spec/factories/users.rb
FactoryBot.define do
  factory :user do
    sequence(:email) { |n| "user#{n}@example.com" }
    password { 'password123' }
    first_name { 'John' }
    last_name { 'Doe' }
    role { 'user' }

    trait :admin do
      role { 'admin' }
    end

    trait :with_quests do
      after(:create) do |user|
        create_list(:quest, 3, creator: user)
      end
    end
  end
end

# spec/factories/quests.rb
FactoryBot.define do
  factory :quest do
    association :creator, factory: :user
    sequence(:title) { |n| "Quest #{n}" }
    description { 'An exciting quest awaits!' }
    start_latitude { 40.7128 }
    start_longitude { -74.0060 }
    end_latitude { 40.7589 }
    end_longitude { -73.9851 }
    visibility { 'public' }

    trait :private do
      visibility { 'private' }
    end

    trait :with_reward do
      reward_amount { 50.00 }
      reward_currency { 'USD' }
    end

    trait :with_checkpoints do
      after(:create) do |quest|
        create_list(:checkpoint, 3, quest: quest)
      end
    end
  end
end
```

## Test Configuration

### RSpec Configuration
```ruby
# spec/rails_helper.rb
RSpec.configure do |config|
  # Use Factory Bot
  config.include FactoryBot::Syntax::Methods

  # Use Devise helpers
  config.include Devise::Test::ControllerHelpers, type: :controller
  config.include Devise::Test::IntegrationHelpers, type: :feature

  # Database cleaner
  config.before(:suite) do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean_with(:truncation)
  end

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end

  # Capybara configuration for JavaScript tests
  config.before(:each, js: true) do
    DatabaseCleaner.strategy = :truncation
  end

  # SimpleCov configuration
  config.before(:all) do
    SimpleCov.start 'rails' do
      add_filter '/spec/'
      add_filter '/config/'
      add_filter '/vendor/'
      
      minimum_coverage 90
      refuse_coverage_drop
    end
  end
end
```

## Continuous Integration

### GitHub Actions Example
```yaml
# .github/workflows/test.yml
name: Test Suite

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
          
      - name: Set up database
        run: |
          bundle exec rails db:create
          bundle exec rails db:migrate
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
          
      - name: Run tests
        run: bundle exec rspec --coverage
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test
          
      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
```

## Best Practices

### Test Organization
- Group related tests with `describe` blocks
- Use `context` for different scenarios
- Use `let` for test data setup
- Use `before` hooks for common setup

### Test Data
- Use factories instead of fixtures
- Create minimal test data needed for each test
- Use traits for variations
- Clean up test data between tests

### Test Naming
- Use descriptive test names
- Start with "it" for behavior descriptions
- Use "should" sparingly, focus on behavior
- Group related tests logically

### Test Maintenance
- Keep tests simple and focused
- Update tests when requirements change
- Remove obsolete tests
- Refactor tests to reduce duplication

---

**Remember**: Testing is not optional. Every feature must have comprehensive test coverage before being considered complete.