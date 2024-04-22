# rails app tutorial

Create rails app

`rails new easy-rails-app -T -d postgresql`

Add gems to Gemfile

```
gem "devise"
gem "slim-rails"

group :development do
  gem 'rubocop-rails', require: false
end

group :development, :test do
  gem "pry"
end
```

Add `/config/database.yml` to `.gitignore`

Initialize devise && add users

`rails g devise:install`
`rails g devise user`
`rails g scaffold post title body user:references`
