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

Create the sessions `controller`:

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

Create a chat `controller`:

    rails generate controller chats room new_massage

Add chat route to `config/routes.rb`

    get  '/chatroom' => 'chats#room', :as => :chat
    post '/new_message' => 'chats#new_message', :as => :new_message

Add code to room method of chat controller.

    def room
      redirect_to login_path unless session[:username]
    end

Modify chats room view

    <%= subscribe_to "/messages/public" %>
    <%= subscribe_to "/messages/private/#{session[:username]}" %>
      
    <script type="text/javascript" charset="utf-8">
      PrivatePub.subscribe("/messages/public", function(data) {
        $('<p></p>').html(data.username + ": " + data.msg).appendTo('#chat_room');
      });
      
      PrivatePub.subscribe("/messages/private/<%= session[:username] %>", function(data) {
        $('<p></p>').addClass('private').html(data.username + ": " + data.msg).appendTo('#chat_room');
      });
    </script>
      
    <div class="chat_container">
      <div id="chat_room">
        <p class="alert"> Welcome to the chat room <%= session[:username] %>! </p>
      </div>
      
      <%= form_tag new_message_path, :remote => true do %>
        <%= text_field_tag :message %>
        <%= submit_tag "Send" %>
      <% end %>
    </div>

Add code to new_message method of chat controller.

    def new_message
      # faye = URI.parse 'http://localhost:9292/faye'
      
      # Check if the message is private
      if recipient = params[:message].match(/@(.+) (.+)/)
        
        # It is private, send it to the recipient's channel
        # message = {:username => session[:username], :msg => recipient.captures.second}.to_json
        # Net::HTTP.post_form(faye, :message => {:channel => "/messages/private/#{recipient.captures.first}", :data => message}.to_json)
        @channel = "/messages/private/#{recipient.captures.first}"
        @message = { :username => session[:username], :msg => recipient.captures.second }
      else
        # message = {:username => session[:username], :msg => params[:message]}.to_json
        # Net::HTTP.post_form(faye, :message => {:channel => '/messages/public', :data => message}.to_json)
        @channel = "/messages/public"
        @message = { :username => session[:username], :msg => params[:message] }
      end
      
      respond_to do |f|
        f.js
      end
    end

Rename chats view template from `new_message.html.erb` to `new_message.html.erb`. Then modify new_message view

    // Clear message input
    $('#message').val('');
    
    // Send the message
    <% publish_to @channel, @message %>
    