# Security Checklist

Comprehensive security guidelines and checklist for all development projects.

## Overview

Security is a fundamental requirement, not an optional feature. This checklist ensures that all code follows security best practices and defensive programming principles. Every feature must pass security review before being considered complete.

## Core Security Principles

### 1. Defensive Programming
- **Assume all input is malicious** until proven otherwise
- **Validate everything** - never trust user input
- **Fail securely** - errors should not expose sensitive information
- **Use least privilege** - grant minimum necessary permissions
- **Defense in depth** - multiple layers of security

### 2. Data Protection
- **Encrypt sensitive data** at rest and in transit
- **Hash passwords** using strong algorithms (bcrypt, scrypt, Argon2)
- **Sanitize output** to prevent XSS attacks
- **Protect against injection** attacks (SQL, NoSQL, Command)
- **Secure file uploads** and downloads

### 3. Authentication & Authorization
- **Strong authentication** mechanisms
- **Proper session management**
- **Role-based access control**
- **Multi-factor authentication** where appropriate
- **Secure password policies**

## Input Validation

### Always Validate
- [ ] **All user inputs** (forms, URLs, headers, cookies)
- [ ] **File uploads** (type, size, content)
- [ ] **API parameters** (required, format, bounds)
- [ ] **Database queries** (parameterized queries only)
- [ ] **Configuration values** (environment variables, settings)

### Validation Techniques
```ruby
# Ruby on Rails examples

# Model validation
class User < ApplicationRecord
  validates :email, presence: true, 
                   format: { with: URI::MailTo::EMAIL_REGEXP },
                   uniqueness: { case_sensitive: false }
  validates :password, presence: true, 
                      length: { minimum: 8 },
                      format: { with: /\A(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/ }
end

# Controller parameter validation
class QuestsController < ApplicationController
  private

  def quest_params
    params.require(:quest).permit(:title, :description, :visibility, 
                                  :start_latitude, :start_longitude, 
                                  :end_latitude, :end_longitude)
  end

  def validate_coordinates
    %w[start_latitude start_longitude end_latitude end_longitude].each do |coord|
      value = params.dig(:quest, coord)
      next if value.blank?
      
      unless value.to_f.between?(-180, 180)
        flash[:error] = "Invalid coordinate value"
        redirect_to new_quest_path and return
      end
    end
  end
end

# Custom validation
class Quest < ApplicationRecord
  validate :coordinates_within_bounds
  
  private
  
  def coordinates_within_bounds
    if start_latitude && (start_latitude < -90 || start_latitude > 90)
      errors.add(:start_latitude, "must be between -90 and 90")
    end
    
    if start_longitude && (start_longitude < -180 || start_longitude > 180)
      errors.add(:start_longitude, "must be between -180 and 180")
    end
  end
end
```

## Output Sanitization

### Prevent XSS Attacks
```ruby
# HAML automatically escapes by default
%p= user.name  # Automatically escaped

# When you need raw HTML (be very careful)
%p!= sanitize(user.bio, tags: %w[b i em strong])

# JavaScript context
:javascript
  var userName = #{user.name.to_json};  # Properly escaped for JS

# URL context
%a{href: url_for(user)}= user.name
```

### Content Security Policy
```ruby
# config/application.rb
config.content_security_policy do |policy|
  policy.default_src :self
  policy.script_src :self, :unsafe_inline, 'https://unpkg.com'
  policy.style_src :self, :unsafe_inline, 'https://cdnjs.cloudflare.com'
  policy.img_src :self, :data, 'https:'
  policy.connect_src :self, 'https://nominatim.openstreetmap.org'
end
```

## Authentication Security

### Password Security
- [ ] **Use bcrypt** or similar strong hashing algorithm
- [ ] **Enforce password complexity** (length, character types)
- [ ] **Implement password history** (prevent reuse)
- [ ] **Use secure password reset** mechanisms
- [ ] **Implement account lockout** after failed attempts

```ruby
# Using Devise with proper configuration
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable,
         :lockable, :timeoutable

  # Password complexity validation
  validates :password, format: { 
    with: /\A(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/,
    message: 'must include at least one lowercase letter, one uppercase letter, one digit, and one special character'
  }
end

# config/initializers/devise.rb
Devise.setup do |config|
  config.password_length = 12..128
  config.lock_strategy = :failed_attempts
  config.maximum_attempts = 5
  config.unlock_strategy = :time
  config.unlock_in = 30.minutes
  config.timeout_in = 2.hours
end
```

### Session Security
- [ ] **Use secure session cookies**
- [ ] **Implement session timeout**
- [ ] **Regenerate session IDs** after login
- [ ] **Use HTTPS only** cookies
- [ ] **Implement proper logout**

```ruby
# config/application.rb
config.session_store :cookie_store, 
                     key: '_app_session',
                     secure: Rails.env.production?,
                     httponly: true,
                     same_site: :strict

# config/initializers/session_store.rb
Rails.application.config.session_store :cookie_store, 
  expire_after: 2.hours,
  secure: Rails.env.production?
```

## Authorization

### Role-Based Access Control
```ruby
# Using CanCanCan
class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new

    if user.admin?
      can :manage, :all
    elsif user.user?
      can :read, Quest, visibility: 'public'
      can :manage, Quest, creator: user
      can :create, Quest
      can :participate, Quest do |quest|
        quest.visibility == 'public' || quest.creator == user
      end
    else
      can :read, Quest, visibility: 'public'
    end
  end
end

# Controller usage
class QuestsController < ApplicationController
  authorize_resource
  
  def show
    # Authorization automatically handled by CanCanCan
  end
  
  def create
    @quest = current_user.created_quests.build(quest_params)
    authorize! :create, @quest
    
    if @quest.save
      redirect_to @quest, notice: 'Quest created successfully'
    else
      render :new
    end
  end
end
```

### Admin Access Protection
```ruby
# Admin namespace protection
class Admin::BaseController < ApplicationController
  before_action :authenticate_admin!
  
  private
  
  def authenticate_admin!
    redirect_to root_path unless current_user&.admin?
  end
end

# Strong parameter validation for admin actions
class Admin::UsersController < Admin::BaseController
  def update
    @user = User.find(params[:id])
    
    # Prevent privilege escalation
    if user_params[:role] == 'admin' && !current_user.super_admin?
      flash[:error] = "Insufficient permissions to grant admin access"
      redirect_to admin_user_path(@user) and return
    end
    
    if @user.update(user_params)
      redirect_to admin_user_path(@user), notice: 'User updated successfully'
    else
      render :edit
    end
  end
  
  private
  
  def user_params
    params.require(:user).permit(:first_name, :last_name, :email, :role)
  end
end
```

## Database Security

### SQL Injection Prevention
```ruby
# NEVER do this (vulnerable to SQL injection)
User.where("name = '#{params[:name]}'")

# Always use parameterized queries
User.where("name = ?", params[:name])
User.where(name: params[:name])
User.find_by(name: params[:name])

# Complex queries with parameters
User.where("created_at > ? AND role = ?", 1.week.ago, params[:role])

# Using ActiveRecord query methods (preferred)
User.where(created_at: 1.week.ago.., role: params[:role])
```

### Database Configuration Security
```ruby
# config/database.yml - use environment variables
production:
  adapter: postgresql
  database: <%= ENV['DATABASE_NAME'] %>
  username: <%= ENV['DATABASE_USER'] %>
  password: <%= ENV['DATABASE_PASSWORD'] %>
  host: <%= ENV['DATABASE_HOST'] %>
  port: <%= ENV['DATABASE_PORT'] %>
  sslmode: require
  sslcert: <%= ENV['DATABASE_SSL_CERT'] %>
  sslkey: <%= ENV['DATABASE_SSL_KEY'] %>
  sslrootcert: <%= ENV['DATABASE_SSL_ROOT_CERT'] %>
```

## File Upload Security

### Secure File Handling
```ruby
class MediaUploadController < ApplicationController
  before_action :authenticate_user!
  
  ALLOWED_TYPES = %w[image/jpeg image/png image/gif].freeze
  MAX_FILE_SIZE = 5.megabytes
  
  def create
    uploaded_file = params[:file]
    
    # Validate file presence
    unless uploaded_file.present?
      render json: { error: 'No file provided' }, status: :bad_request
      return
    end
    
    # Validate file type
    unless ALLOWED_TYPES.include?(uploaded_file.content_type)
      render json: { error: 'Invalid file type' }, status: :bad_request
      return
    end
    
    # Validate file size
    if uploaded_file.size > MAX_FILE_SIZE
      render json: { error: 'File too large' }, status: :bad_request
      return
    end
    
    # Validate file content (not just extension)
    unless valid_image?(uploaded_file)
      render json: { error: 'Invalid file content' }, status: :bad_request
      return
    end
    
    # Generate secure filename
    filename = SecureRandom.uuid + File.extname(uploaded_file.original_filename)
    
    # Store file securely
    # ... implementation
  end
  
  private
  
  def valid_image?(file)
    # Use ImageMagick or similar to validate actual image content
    begin
      image = MiniMagick::Image.read(file.tempfile)
      image.valid?
    rescue
      false
    end
  end
end
```

## API Security

### API Authentication
```ruby
class ApiController < ApplicationController
  before_action :authenticate_api_user!
  
  private
  
  def authenticate_api_user!
    token = request.headers['Authorization']&.split(' ')&.last
    
    unless token.present?
      render json: { error: 'Missing authentication token' }, status: :unauthorized
      return
    end
    
    # Validate JWT token
    begin
      decoded_token = JWT.decode(token, Rails.application.secrets.secret_key_base, true, { algorithm: 'HS256' })
      @current_user = User.find(decoded_token[0]['user_id'])
    rescue JWT::DecodeError
      render json: { error: 'Invalid token' }, status: :unauthorized
    end
  end
end
```

### Rate Limiting
```ruby
# Using rack-attack
class Rack::Attack
  # Throttle login attempts
  throttle('logins/ip', limit: 5, period: 20.seconds) do |req|
    req.ip if req.path == '/users/sign_in' && req.post?
  end
  
  # Throttle API requests
  throttle('api/ip', limit: 100, period: 1.hour) do |req|
    req.ip if req.path.starts_with?('/api/')
  end
  
  # Block suspicious IPs
  blocklist('suspicious_ips') do |req|
    # Check against known malicious IPs
    SuspiciousIp.exists?(ip: req.ip)
  end
end
```

## Environment Security

### Environment Variables
```ruby
# .env file (never commit to version control)
DATABASE_URL=postgresql://user:pass@localhost/db
SECRET_KEY_BASE=very_long_random_string
API_KEY=secret_api_key

# config/application.rb
class Application < Rails::Application
  # Ensure required environment variables are set
  config.before_configuration do
    required_env_vars = %w[DATABASE_URL SECRET_KEY_BASE]
    missing_vars = required_env_vars.select { |var| ENV[var].blank? }
    
    if missing_vars.any?
      raise "Missing required environment variables: #{missing_vars.join(', ')}"
    end
  end
end
```

### Secrets Management
```ruby
# config/credentials.yml.enc (encrypted)
secret_key_base: abc123...
database:
  password: secret_password
stripe:
  publishable_key: pk_test_...
  secret_key: sk_test_...

# Usage in application
Rails.application.credentials.dig(:stripe, :secret_key)
```

## Error Handling Security

### Secure Error Messages
```ruby
class ApplicationController < ActionController::Base
  rescue_from StandardError, with: :handle_error if Rails.env.production?
  
  private
  
  def handle_error(exception)
    # Log detailed error for debugging
    Rails.logger.error "Error: #{exception.message}"
    Rails.logger.error exception.backtrace.join("\n")
    
    # Show generic error to user
    respond_to do |format|
      format.html { render 'errors/500', status: :internal_server_error }
      format.json { render json: { error: 'Internal server error' }, status: :internal_server_error }
    end
  end
end
```

## Logging Security

### Secure Logging
```ruby
# config/application.rb
config.filter_parameters += [:password, :password_confirmation, :secret_key, :token, :api_key]

# Custom logging
class ApplicationController < ActionController::Base
  after_action :log_user_action
  
  private
  
  def log_user_action
    return unless current_user
    
    UserAction.create!(
      user: current_user,
      action: "#{controller_name}##{action_name}",
      ip_address: request.remote_ip,
      user_agent: request.user_agent,
      parameters: filtered_params
    )
  end
  
  def filtered_params
    params.except(:password, :password_confirmation, :secret_key, :token)
  end
end
```

## Third-Party Integration Security

### API Integration Security
```ruby
class ExternalApiService
  def initialize
    @base_url = ENV['API_BASE_URL']
    @api_key = ENV['API_KEY']
  end
  
  def fetch_data(params)
    # Validate and sanitize parameters
    sanitized_params = sanitize_params(params)
    
    # Make secure HTTP request
    response = HTTParty.get(
      "#{@base_url}/data",
      query: sanitized_params,
      headers: {
        'Authorization' => "Bearer #{@api_key}",
        'Content-Type' => 'application/json',
        'User-Agent' => 'MyApp/1.0'
      },
      timeout: 10,
      verify: true  # Verify SSL certificates
    )
    
    handle_response(response)
  end
  
  private
  
  def sanitize_params(params)
    # Remove potentially dangerous parameters
    params.except(:system, :eval, :exec, :command)
  end
  
  def handle_response(response)
    case response.code
    when 200
      JSON.parse(response.body)
    when 401
      Rails.logger.error "API authentication failed"
      nil
    when 429
      Rails.logger.warn "API rate limit exceeded"
      nil
    else
      Rails.logger.error "API request failed: #{response.code}"
      nil
    end
  rescue JSON::ParserError
    Rails.logger.error "Invalid JSON response from API"
    nil
  end
end
```

## Security Testing

### Security Test Cases
```ruby
# spec/security/authentication_spec.rb
RSpec.describe 'Authentication Security', type: :request do
  describe 'login attempts' do
    it 'locks account after 5 failed attempts' do
      user = create(:user)
      
      5.times do
        post '/users/sign_in', params: { 
          user: { email: user.email, password: 'wrong' } 
        }
      end
      
      expect(user.reload).to be_locked_at
    end
  end
  
  describe 'session management' do
    it 'invalidates session after timeout' do
      user = create(:user)
      sign_in user
      
      travel_to 3.hours.from_now do
        get '/dashboard'
        expect(response).to redirect_to(new_user_session_path)
      end
    end
  end
end

# spec/security/authorization_spec.rb
RSpec.describe 'Authorization Security', type: :request do
  describe 'admin access' do
    it 'prevents regular users from accessing admin panel' do
      user = create(:user, role: 'user')
      sign_in user
      
      get '/admin/dashboard'
      expect(response).to redirect_to(root_path)
    end
  end
  
  describe 'quest access' do
    it 'prevents access to private quests by non-creators' do
      creator = create(:user)
      other_user = create(:user)
      quest = create(:quest, creator: creator, visibility: 'private')
      
      sign_in other_user
      get "/quests/#{quest.id}"
      expect(response).to have_http_status(:not_found)
    end
  end
end
```

## Security Review Checklist

### Pre-Deployment Security Review
- [ ] **Input validation** implemented for all user inputs
- [ ] **Output sanitization** prevents XSS attacks
- [ ] **SQL injection** prevention using parameterized queries
- [ ] **Authentication** properly implemented and tested
- [ ] **Authorization** controls access to all resources
- [ ] **Session management** is secure and times out appropriately
- [ ] **Error handling** doesn't expose sensitive information
- [ ] **File uploads** are validated and stored securely
- [ ] **API endpoints** are authenticated and rate-limited
- [ ] **Environment variables** are used for sensitive configuration
- [ ] **Logging** doesn't include sensitive information
- [ ] **Third-party integrations** are secure and validated
- [ ] **HTTPS** is enforced in production
- [ ] **Security headers** are properly configured
- [ ] **Dependencies** are up-to-date and vulnerability-free

### Ongoing Security Monitoring
- [ ] **Regular security audits** of dependencies
- [ ] **Penetration testing** for critical applications
- [ ] **Code reviews** with security focus
- [ ] **Security training** for development team
- [ ] **Incident response** plan in place
- [ ] **Backup and recovery** procedures tested
- [ ] **Compliance** with relevant standards (GDPR, HIPAA, etc.)

## Security Tools and Resources

### Recommended Tools
```bash
# Ruby security scanner
gem install brakeman
brakeman

# Dependency vulnerability scanner
gem install bundler-audit
bundle audit

# License compliance
gem install license_finder
license_finder

# Static analysis
gem install reek
reek

# Code quality
gem install rubocop
rubocop
```

### Security Headers
```ruby
# config/application.rb
config.force_ssl = true if Rails.env.production?

# Security headers middleware
class SecurityHeadersMiddleware
  def initialize(app)
    @app = app
  end
  
  def call(env)
    status, headers, response = @app.call(env)
    
    headers['X-Frame-Options'] = 'DENY'
    headers['X-Content-Type-Options'] = 'nosniff'
    headers['X-XSS-Protection'] = '1; mode=block'
    headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
    
    [status, headers, response]
  end
end
```

---

**Remember**: Security is everyone's responsibility. When in doubt, choose the more secure option and ask for security review.