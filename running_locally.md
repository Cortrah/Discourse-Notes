# Running Locally

If running the vagrant install follow these instructions

to start up vagrant, from the root folder

    vagrant up

then connect to the vm
  
    vagrant ssh

then change to the vagrant directory
  
    cd /vagrant

(first time only or after adding plugins) invoke the rails installer

    bundle install

(also only after bundle installs)

migrate the dev database schema to my local repo

    bundle exec rake db:migrate

and finally bring up rails
  
    bundle exec rails s -b 0.0.0.0

now the app should be up at http://localhost:4000

in theory, you should be able to run the tests at http://localhost:3000, but that doesn't work for me yet.

when developing if your plugin changes don't get noticed you may need to delete the tmp directory
  
    rm -rf /tmp

when done
    
    exit
    vagrant halt

