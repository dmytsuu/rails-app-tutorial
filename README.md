# rails app tutorial

Create rails app

`rails new easy-rails-app -T -d postgresql`

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

group :development do
  gem 'rubocop-rails', require: false
end

group :development, :test do
  gem "pry"
  gem 'rspec-rails', '~> 6.1.0'
end
```

`rails action_text:install`

Install gems

`bundle`

Add `/config/database.yml` to `.gitignore`

Commands && scaffold

```
rails g devise:install
rails g devise user
rails g scaffold post title content:rich_text slug:uniq user:references discarded_at:datetime:index
rails g pundit:install
rails g pundit:policy post
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

  friendly_id :title, use: :slugged

  pg_search_scope :by_title,
                against: :title,
                using: { trigram: { word_similarity: true } }
```

Add to application_controller

```
  include Pundit::Authorization
  include Pagy::Backend
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
```

Run

`rubocop -A`

