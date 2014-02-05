faye_rails4_example
===================

Faye PubSub example in Rails4

This repo is an implementation of the 
[How to Use Faye as a Real-Time Push Server in Rails]
(http://net.tutsplus.com/tutorials/ruby/how-to-use-faye-as-a-real-time-push-server-in-rails/)
tutorial in Rails 4.

Step 1 - Get Everything Ready
-----------------------------

    rails new faye-rails4

Add the following gems to `Gemfile`:

    gem 'faye'
    gem 'thin'
    gem 'private_pub', :git => 'git://github.com/ryanb/private_pub.git'

Install the new gems.

    bundle install

Then install private_pub

    rails g private_pub:install

Private_pub install now generates `faye.ru`, remaned as `private_pub.ru`. 

Run the faye server using Rackup.

    rackup privte_pub.ru -E production -s thin

Step 2 - Some Basic Authentication
----------------------------------

Create the sessions `controller`

    rails g controller sessions new create

Add these routes to your `config/routes.rb` file:

    get  '/login' => 'sessions#new', :as => :login
    post '/login' => 'sessions#create'

Add the following to sessions controller method create

    def create
      session[:username] = params[:username]
      redirect_to chat_path
    end

Modify sessions new view.

    <div style="text-align: center">
      <%= form_tag login_path do |f| %>
        <%= label_tag :username %>
        <%= text_field_tag :username %>
        <%= submit_tag "Enter" %>
      <% end %>
    </div>

Modify the application layout. (stub)

Step 3 - The Chat Room
----------------------
