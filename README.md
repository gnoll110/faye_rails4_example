faye_rails4_example
===================

Faye PubSub example in Rails4

rails new faye-rails4

Gemfile:
gem 'faye'
gem 'thin'
gem 'private_pub', :git => 'git://github.com/ryanb/private_pub.git'

rails g private_pub:install

Private_pub install now generates `faye.ru`, remaned as `private_pub.ru`. 

rackup privte_pub.ru -E production -s thin
