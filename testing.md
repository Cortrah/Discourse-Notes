# Testing Notes

After makeing changes and stopping the server

### possibly remove /tmp

    

### Install any needed gems

    bundle install 

### Doublecheck Mailcatcher

ensure that config/environments/development.rb has these settings before running your migrations and tests.

    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = { address: "localhost", port: 1025 }


### Prepare your database

    rake db:migrate
    rake db:test:prepare
    rake db:seed_fu

## Now, test it out!

    bundle exec rspec

### Resetting the environmnet variables

Reset the environment as a possible solution to failed rspec tests.
These commands assume an empty Discourse database, and an otherwise empty redis environment. CAREFUL HERE

    RAILS_ENV=test bundle exec rake db:drop db:create db:migrate
    redis-cli flushall
    bundle exec rspec # re-running to see if tests pass

If you are running the software locally then 

    http://localhost:3000/qunit
    
Will run the qunit tests

## FakeWeb

Discourse uses the [FakeWeb](https://github.com/chrisk/fakeweb) gem to fake external web 
requests.
For example, check out the specs on `specs/components/oneboxer`.

This has several advantages to making real requests:

* It freezes the expected response from the remote server.
* It doen't need a network connection to run the specs.
* It's faster.

So, if you need to define a spec that makes a web request, you'll have to record 
the real response to a fixture file, and tell FakeWeb to respond with it for the 
URI of your request.

Check out `spec/components/oneboxer/amazon_onebox_spec.rb` for an example on 
this.

### Recording responses

To record the actual response from the remote server, you can use curl and save the response to a file. They use the `-i` option to include headers in the output

    curl -i http://en.m.wikipedia.org/wiki/Ruby > wikipedia.response
