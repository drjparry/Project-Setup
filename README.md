# Project-Setup


`$ gem install rails`

`rails new call4 -d mysql -T`

or

`'rails new call4 --api -d mysql -T'`

rake db:create

```ruby
group :test do
  gem 'rspec-rails'
  gem 'capybara'
  gem "factory_girl_rails", "~> 4.0"
  gem 'database_cleaner'

end
```

`bundle`

`rails generate rspec:install`

In your `spec/rails_helper.rb` file add the following require statement below the other require statements:

`require 'capybara/rails'`
`require 'support/factory_girl'`
`require_relative 'support/database_cleaner.rb'`


Add new file

`spec/factories/callfors.rb`

Factory looks like

```
FactoryGirl.define do
  factory :callfor do
    title "Dump Trump"
    description "No more Trump please"
  end
end
```

Add new file 

`spec/support/database_cleaner.rb`
 `
 ```
 RSpec.configure do |config|

  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each, :js => true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
end
 ```
 
 add new file 
 
 `spec/support/factory_girl.rb`
 
 ```
 RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
end
```
___

Add React

Add to gemfile

`gem 'react-rails', '~> 1.5' `

Then in terminal

`rails generate react:install`

----
'in config/routes.rb..'

```ruby
get 'callfors' => 'campaigns#index'
```

Add Foundation
`gem 'foundation-rails'`
bundle
`rails g foundation:install`
