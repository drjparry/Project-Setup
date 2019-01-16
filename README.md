# Project-Setup


`$ gem install rails`

`rails new call4 -d mysql -T --webpack=react`

or

`'rails new call4 --api -d mysql -T'`

rake db:create

```ruby
group :test do
  gem 'rspec-rails'
  gem 'capybara'
  gem 'factory_bot_rails'
  gem 'database_cleaner'
  gem 'selenium-webdriver'
  gem 'chromedriver-helper'
end

group :development, :test do
  gem 'pry-rails'
  gem 'pry-byebug'
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
  gem 'shoulda-matchers', '~> 3.1'
  gem 'rails-controller-testing'
  gem 'spinach'
end
```

`bundle`

`rails generate rspec:install`

In your `spec/rails_helper.rb` file add the following require statement below the other require statements:

`require 'capybara/rails'`
`require 'support/factory_bot_rails'`
`require_relative 'support/database_cleaner.rb'`
```
Capybara.register_driver :chrome do |app|
  Capybara::Selenium::Driver.new(app, :browser => :chrome)
end


Capybara.register_driver :selenium do |app|
  Capybara::Selenium::Driver.new(app, browser: :chrome)
end

Capybara.javascript_driver = :chrome

Capybara.configure do |config|
  config.default_max_wait_time = 10 # seconds
  config.default_driver        = :selenium
end
```


Add new file

`spec/factories/callfors.rb`

Factory looks like

```
FactoryBot.define do
  factory :user do
    first_name { "John" }
    last_name  { "Doe" }
    admin { false }
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
  config.include FactoryBot::Syntax::Methods
end
```

add new file for Spinach

`features/supportenv.rb`

```
# Help Spinach find the spec directory for autoloading
$LOAD_PATH.unshift(::File.expand_path('../../spec', __dir__))
require './config/environment'
require 'rspec/rails'
# Most of the desired config is already in rspec
require 'rails_helper'
require 'database_cleaner'


Dir[Rails.root.join("features/support/**/*.rb")].each do |f|
  require f
end

DatabaseCleaner.strategy = :truncation

Spinach.hooks.before_scenario do
  ::Capybara.current_driver = :selenium_chrome_headless
  DatabaseCleaner.clean_with(:truncation)
end

Spinach.hooks.after_scenario do
  DatabaseCleaner.clean
end

# Spinach.config.save_and_open_page_on_failure = true

```




To compile webpack before tests
```
config.before :suite do
  # Run webpack compilation before suite, so assets exists in public/packs
  # It would be better to run the webpack compilation only if at least one :js spec
  # should be executed, but `when_first_matching_example_defined`
  # does not work with `config.infer_spec_type_from_file_location!`
  # see https://github.com/rspec/rspec-core/issues/2366
  `RAILS_ENV=test bin/webpack`
end
```

Add Guard to dev
```
gem 'guard'
gem 'guard-rspec' 
gem 'guard-cucumber'

bundle install

guard init 
bundle binstubs guard
cucumber --init 
```
to run guard, type ```guard```

Guardfile should look like
```
# A sample Guardfile
# More info at https://github.com/guard/guard#readme

## Uncomment and set this to only include directories you want to watch
# directories %w(app lib config test spec features) \
#  .select{|d| Dir.exists?(d) ? d : UI.warning("Directory #{d} does not exist")}

## Note: if you are using the `directories` clause above and you are not
## watching the project directory ('.'), then you will want to move
## the Guardfile to a watched dir and symlink it back, e.g.
#
#  $ mkdir config
#  $ mv Guardfile config/
#  $ ln -s config/Guardfile .
#
# and, you'll have to watch "config/Guardfile" instead of "Guardfile"

cucumber_options = {
  # Below are examples overriding defaults

  # cmd: 'bin/cucumber',
  # cmd_additional_args: '--profile guard',

  # all_after_pass: false,
  # all_on_start: false,
  # keep_failed: false,
  # feature_sets: ['features/frontend', 'features/experimental'],

  # run_all: { cmd_additional_args: '--profile guard_all' },
  # focus_on: { 'wip' }, # @wip
  # notification: false
}

guard "cucumber", cucumber_options do
  watch(%r{^features/.+\.feature$})
  watch(%r{^features/support/.+$}) { "features" }

  watch(%r{^features/step_definitions/(.+)_steps\.rb$}) do |m|
    Dir[File.join("**/#{m[1]}.feature")][0] || "features"
  end
end

# Note: The cmd option is now required due to the increasing number of ways
#       rspec may be run, below are examples of the most common uses.
#  * bundler: 'bundle exec rspec'
#  * bundler binstubs: 'bin/rspec'
#  * spring: 'bin/rspec' (This will use spring if running and you have
#                          installed the spring binstubs per the docs)
#  * zeus: 'zeus rspec' (requires the server to be started separately)
#  * 'just' rspec: 'rspec'

guard :rspec, cmd: "bundle exec rspec" do
  require "guard/rspec/dsl"
  dsl = Guard::RSpec::Dsl.new(self)

  # Feel free to open issues for suggestions and improvements

  # RSpec files
  rspec = dsl.rspec
  watch(rspec.spec_helper) { rspec.spec_dir }
  watch(rspec.spec_support) { rspec.spec_dir }
  watch(rspec.spec_files)

  # Ruby files
  ruby = dsl.ruby
  dsl.watch_spec_files_for(ruby.lib_files)

  # Rails files
  rails = dsl.rails(view_extensions: %w(erb haml slim))
  dsl.watch_spec_files_for(rails.app_files)
  dsl.watch_spec_files_for(rails.views)

  watch(%r{^app/controllers/(.+)_(controller)\.rb$})  { "spec/features" }
  watch(%r{^app/models/(.+)\.rb$})  { "spec/features" }
  watch(rails.controllers) do |m|
    [
      rspec.spec.call("routing/#{m[1]}_routing"),
      rspec.spec.call("controllers/#{m[1]}_controller"),
      rspec.spec.call("acceptance/#{m[1]}")
    ]
  end

  # Rails config changes
  watch(rails.spec_helper)     { rspec.spec_dir }
  watch(rails.routes)          { "spec" } # { "#{rspec.spec_dir}/routing" }
  watch(rails.app_controller)  { "#{rspec.spec_dir}/controllers" }

  # Capybara features specs
  watch(rails.view_dirs)     { |m| rspec.spec.call("features/#{m[1]}") }
  watch(rails.layouts)       { |m| rspec.spec.call("features/#{m[1]}") }

  # Turnip features and steps
  watch(%r{^spec/acceptance/(.+)\.feature$})
  watch(rails.view_dirs)     { "spec/features" } # { |m| rspec.spec.call("features/#{m[1]}") }
end
```

add Prefixer
```
gem "autoprefixer-rails"

rake tmp:clear
```

'in config/routes.rb..'

```ruby
get 'callfors' => 'campaigns#index'
```

Add Foundation
`gem "foundation-rails"`

bundle
`rails g foundation:install`
