# rails app tutorial

Create rails app

`rails new easy-rails-app -T -d postgresql -c tailwind`

Add gems to Gemfile

```
gem "devise"
gem "slim-rails"
gem 'pagy'
gem "pundit"
gem "sidekiq"
gem 'pg_search'
gem 'friendly_id'
gem 'discard'
gem "interactor", "~> 3.0"

group :development, :test do
  gem 'brakeman'
  gem 'bullet'
  gem 'bundler-audit'
  gem 'pry'
  gem 'rubocop-rails', require: false
end

group :development do
  gem 'kamal'
  gem 'letter_opener'
  gem 'web-console'
end

group :test do
  gem 'factory_bot_rails'
  gem 'ffaker'
  gem 'rspec-rails', '~> 6.1.0'
end
```

`rails action_text:install`

Install gems

`bundle`

Update `config/environments/development.rb`

```
  config.action_mailer.delivery_method = :letter_opener
  config.action_mailer.perform_deliveries = true
```

Update `config/application.rb`

```
  config.active_job.queue_adapter = :sidekiq
```

Update `config/database.yml`

```
production:
  <<: *default
  host: <%= ENV["DB_HOST"] %>
  database: <%= ENV["POSTGRES_DB"] %>
  username: <%= ENV["POSTGRES_USER"] %>
  password: <%= ENV["POSTGRES_PASSWORD"] %>
```

Commands && scaffold

```
rails g devise:install
rails g devise user
rails g scaffold post title content:rich_text slug:uniq user:references discarded_at:datetime:index
rails g pundit:install
rails g pundit:policy post
rails g bullet:install
```

Generate migration for pg plugin

`rails g migration enable_pg_trgm`

Update #change with

`enable_extension 'pg_trgm'`

Prepare database

`rails db:prepare`

Add to post model

```
  extend FriendlyId
  include Discard::Model
  include PgSearch::Model

  friendly_id :title, use: %i[slugged finders]

  pg_search_scope :by_title,
                against: :title,
                using: { trigram: { word_similarity: true } }
```

Add to application_controller

```
  include Pundit::Authorization
  include Pagy::Backend

  before_action :authenticate_user!
```

Add to application_helper

```
  include Pagy::Frontend
```

Rubocop

`rubocop --init`

Add to file 

```
require: rubocop-rails

Style/Documentation:
  Enabled: false

AllCops:
  Exclude:
   - 'db/**/*'
   - 'bin/*'
   - 'config/**/*'
   - 'vendor/**/*'
```

Run

`rubocop -A`

Add to `application.css` at the top

```
@import "https://unpkg.com/open-props";
@import "https://unpkg.com/open-props/normalize.min.css";
@import "https://unpkg.com/open-props/buttons.min.css";
```

Update routs root

```
  root "posts#index"
```

Generate factories

```
rails generate factory_bot:model user email password
rails g factory_bot:model psot title content:rich_text user:references
```

Remove generated specs and create one simple one, for example

```
  it { expect(true).to be_truthy }
```

Add `.github/workflows/rubyonrails.yml`

```
name: "Ruby on Rails CI"
on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:11-alpine
        ports:
          - "5432:5432"
        env:
          POSTGRES_DB: easy_rails_app_test
          POSTGRES_USER: rails
          POSTGRES_PASSWORD: password
    env:
      RAILS_ENV: test
      DATABASE_URL: "postgres://rails:password@localhost:5432/easy_rails_app_test"
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Install Ruby and gems
        uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
      - name: Set up database schema
        run: bin/rails db:prepare
      - name: Run tests
        run: bundle exec rspec

  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Install Ruby and gems
        uses: ruby/setup-ruby@v1
        with:
          bundler-cache: true
      - name: Security audit dependencies
        run: bundle exec bundler-audit --update
      - name: Security audit application code
        run: bundle exec brakeman -q -w2
      - name: Lint Ruby files
        run: bundle exec rubocop --parallel
```

Run `kamal init`

Having a dockerhub account is required in this setup and two VPS machines too for main/secondary services and db(separately).

Example of `.env`

```
KAMAL_REGISTRY_PASSWORD=dckr_pat_o-dfsd$asd-e-dsadad123
RAILS_MASTER_KEY=644123asdf3423ab4a0a1790182312asd
POSTGRES_DB=easy_rails_app_production
POSTGRES_USER=postgres
POSTGRES_PASSWORD=HxXXXM93Nunq9c
DB_HOST=192.168.0.2
APP_HOST=192.168.0.1
REDIS_URL=redis://192.168.0.1:6379
```

Example of `config/deploy.yml`

```
service: your-app
image: your-name/your-app
servers:
  web:
    hosts:
      - <%= ENV.fetch('APP_HOST') %>
  sidekiq:
    cmd: bundle exec sidekiq
    hosts:
      - <%= ENV.fetch('APP_HOST') %>
registry:
  username: your-name
  password:
    - KAMAL_REGISTRY_PASSWORD
env:
  secret:
    - RAILS_MASTER_KEY
    - DB_HOST
    - POSTGRES_DB
    - POSTGRES_USER
    - POSTGRES_PASSWORD
    - REDIS_URL
builder:
  multiarch: false
accessories:
  db:
    image: postgres:latest
    host: <%= ENV.fetch('DB_HOST') %>
    port: 5432
    env:
      clear:
        POSTGRES_USER: "postgres"
        POSTGRES_DB: "your_app_production"
      secret:
        - POSTGRES_PASSWORD
    files:
      # - config/mysql/production.cnf:/etc/mysql/my.cnf
      - db/production.sql:/docker-entrypoint-initdb.d/setup.sql
    directories:
      - data:/var/lib/postgresql/data
  redis:
    image: redis:7.0
    host: <%= ENV.fetch('APP_HOST') %>
    port: 6379
    directories:
      - redis-data:/data
```

Example of `db/production.sql`

```
CREATE DATABASE your_app_production;
```
