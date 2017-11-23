# Running Locally

## If running the vagrant install follow these instructions

First go to the volume with the repository

Since I am using it on the ssd remember to change to that volume the command is

    cd /Volumes/Turnstyles
    
Then go to the root folder
    
    cd discourse

to start up vagrant, from that root folder

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
    
    
If you don't have an admin account yet create one before starting the server

    rake admin:create

and finally bring up rails
  
    bundle exec rails s -b 0.0.0.0

now the app should be up at http://localhost:4000

in theory, you should be able to run the tests at http://localhost:3000, but that doesn't work for me yet.

when developing if your plugin changes don't get noticed you may need to delete the tmp directory
  
    rm -rf /tmp

when done
    
    exit
    vagrant halt
    
Admin is at /admin    

Remember from the developer setup that Mailcatcher is used to avoid the whole issue of actually sending emails: https://github.com/sj26/mailcatcher

Mailcatcher is already installed in the vm, and there's an alias to launch it:

```
mailcatcher --http-ip=0.0.0.0
```
   
Here are some more [details on mailcatcher](mail_catcher.md)

And some [notes on testing](testing.md)