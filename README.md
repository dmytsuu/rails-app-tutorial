# rails app tutorial

Create rails app

`rails new easy-rails-app -T -d postgresql`

Add gems to Gemfile

```
gem "slim-rails"

group :development do
  gem 'rubocop-rails', require: false
end

group :development, :test do
  gem "pry"
end
```
