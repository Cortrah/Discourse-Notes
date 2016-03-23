# Development Setup

There are 3 levels of instructions to choose from
1. Discourse with vagrant
2. Discourse with a custom vagrant setup
3. Installation from script (osx only)

## 1. Discourse with Vagrant

The following instructions will automatically download and provision a virtual machine for you to begin hacking
on Discourse with:

### Getting Started

1. Install Git: http://git-scm.com/downloads 
2. Install VirtualBox: https://www.virtualbox.org/wiki/Downloads
3. Install Vagrant: http://www.vagrantup.com/ (We require Vagrant 1.7.2 or later)
4. Open a terminal
5. Clone the project: `git clone https://github.com/discourse/discourse.git`
6. Enter the project directory: `cd discourse`

### Using Vagrant

When you're ready to start working, boot the VM:
```
vagrant up
```

Vagrant will prompt you for your admin password. This is so it can mount your local files inside the VM for an easy workflow.

(The first time you do this, it will take a while as it downloads the VM image and installs it. Go grab a coffee.)

Once the machine has booted up, you can shell into it by typing:

```
vagrant ssh
```

The discourse code is found in the /vagrant directory in the image.

### Keeping your VM up to date (and first install)

Now you're in a virtual machine is almost ready to start developing. It's a good idea to perform the following instructions
*every time* you pull from master to ensure your environment is still up to date.

```
cd /vagrant
bundle install
bundle exec rake db:migrate
```

### Starting Rails

Once your VM is up to date, you can start a rails instance using the following command from the /vagrant directory:

```
bundle exec rails s -b 0.0.0.0
```

In a few seconds, rails will start serving pages. To access them, open a web browser to [http://localhost:4000](http://localhost:4000) - if it all worked you should see discourse! Congratulations, you are ready to start working!

If you want to log in as a user, a shortcut you can use in development mode is to follow this link to log in as `eviltrout`:

[http://localhost:4000/session/eviltrout/become](http://localhost:4000/session/eviltrout/become)

You can now edit files on your local file system, using your favorite text editor or IDE. When you reload your web browser, it should have the latest changes.

### Tests

If you're actively working on Discourse, we recommend that you run rake autospec, which will run the specs.  It’s very, very smart. It’ll abort very long test runs. So if it starts running all of the specs and then you just start editing a spec file and save it, it knows that it’s time to interrupt the spec suite, run this one spec for you, then it’ll keep running these specs until they pass as well. If you fail a spec by saving it and then go and start editing around the project to try and fix that spec, it’ll detect that and run that one failing spec, not a hundred of them.

To use it, follow all the above steps. Once rails is running, open a new terminal window or tab, and then do this:

```
vagrant ssh
cd /vagrant
RAILS_ENV=test bundle exec rake db:migrate
bundle exec rake autospec p l=5
```

For more insight into testing Discourse, see [this discussion](http://rubyrogues.com/117-rr-discourse-part-2-with-sam-saffron-and-robin-ward/) with the Ruby Rogues.

### Sending Email

Mail is sent asynchronously by Sidekiq, so you'll need to have sidekiq running to process jobs. Run it with this command in the /vagrant directory:

```
bundle exec sidekiq
```

Mailcatcher is used to avoid the whole issue of actually sending emails: https://github.com/sj26/mailcatcher

Mailcatcher is already installed in the vm, and there's an alias to launch it:

```
mailcatcher --http-ip=0.0.0.0
```

Then in a browser, go to [http://localhost:4080](http://localhost:4080). Sent emails will be received by mailcatcher and shown in its web ui.

### Shutting down the VM

When you're done working on Discourse, you can shut down Vagrant with:

```
vagrant halt
```

# 2. Discourse with Customized Vagrant

## First Steps

1. Install and configure PostgreSQL 9.3+.
  1. Run `postgres -V` to see if you already have it.
  1. Make sure that the server's messages language is English; this is [required](https://github.com/rails/rails/blob/3006c59bc7a50c925f6b744447f1d94533a64241/activerecord/lib/active_record/connection_adapters/postgresql_adapter.rb#L1140) by the ActiveRecord Postgres adapter.
2. Install and configure Redis 2.8.12+.
  1. Run `redis-server -v` to see if you already have it.
3. Install ImageMagick
4. Install libxml2, libpq-dev, g++, gifsicle, libjpeg-progs and make.
5. Install Ruby 2.1.3 and Bundler.
6. Clone the project and bundle.

## Before you start Rails

1. `bundle install`
2. Start up Redis by running `redis-server`
3. `bundle exec rake db:create db:migrate db:test:prepare`
4. Try running the specs: `bundle exec rake autospec`
5. `bundle exec rails server`

You should now be able to connect to rails on [http://localhost:3000](http://localhost:3000) - try it out! The seed data includes a pinned topic that explains how to get an admin account, so start there! Happy hacking!


## Building your own Vagrant VM

Here are the steps we used to create the **[Vagrant Virtual Machine](VAGRANT.md)**. They might be useful if you plan on setting up an environment from scratch on Linux:


### Base box

Vagrant version 1.1.2. With this Vagrantfile:

    Vagrant::Config.run do |config|
      config.vm.box     = 'precise32'
      config.vm.box_url = 'http://files.vagrantup.com/precise32.box'
      config.vm.network :hostonly, '192.168.10.200'

      if RUBY_PLATFORM =~ /darwin/
        config.vm.share_folder("v-root", "/vagrant", ".", :nfs => true)
      end
    end

    vagrant up
    vagrant ssh

### Some basic setup:

    sudo su -
    ln -sf /usr/share/zoneinfo/Canada/Eastern /etc/localtime
    apt-get -yqq update
    apt-get -yqq install python-software-properties
    apt-get -yqq install vim curl expect debconf-utils git-core build-essential zlib1g-dev libssl-dev openssl libcurl4-openssl-dev libreadline6-dev libpcre3 libpcre3-dev imagemagick

### Unicode

    echo "export LANGUAGE=en_US.UTF-8" >> /etc/bash.bashrc
    echo "export LANG=en_US.UTF-8" >> /etc/bash.bashrc
    echo "export LC_ALL=en_US.UTF-8" >> /etc/bash.bashrc
    export LANGUAGE=en_US.UTF-8
    export LANG=en_US.UTF-8
    export LC_ALL=en_US.UTF-8

    locale-gen en_US.UTF-8
    dpkg-reconfigure locales

### RVM and Ruby

    apt-get -yqq install libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config curl build-essential git

    su - vagrant -c "sudo bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)"
    adduser vagrant rvm
    source /etc/profile.d/rvm.sh
    su - vagrant -c "rvm pkg install libyaml"
    su - vagrant -c "rvm install 2.0.0-turbo"
    su - vagrant -c "rvm use 2.0.0-turbo --default"

    echo "gem: --no-document" >> /etc/gemrc
    su - vagrant -c "echo 'gem: --no-document' >> ~/.gemrc"

### Postgres

Configure so that the vagrant user doesn't need to provide username and password.

    apt-get -yqq install postgresql postgresql-contrib-9.3 libpq-dev postgresql-server-dev-9.3
    su - postgres
    createuser --createdb --superuser -Upostgres vagrant
    psql -c "ALTER USER vagrant WITH PASSWORD 'password';"
    psql -c "create database discourse_development owner vagrant encoding 'UTF8' TEMPLATE template0;"
    psql -c "create database discourse_test        owner vagrant encoding 'UTF8' TEMPLATE template0;"
    psql -d discourse_development -c "CREATE EXTENSION hstore;"
    psql -d discourse_development -c "CREATE EXTENSION pg_trgm;"


Edit /etc/postgresql/9.3/main/pg_hba.conf to have this:

    local all all trust
    host all all 127.0.0.1/32 trust
    host all all ::1/128 trust
    host all all 0.0.0.0/0 trust # wide-open

### Redis

    sudo su -
    mkdir /tmp/redis_install
    cd /tmp/redis_install
    wget http://redis.googlecode.com/files/redis-2.6.7.tar.gz
    tar xvf redis-2.6.7.tar.gz
    cd redis-2.6.7
    make
    make install
    cd utils
    ./install_server.sh
    # Press enter to accept all the defaults
    /etc/init.d/redis_6379 start

### Sending email (SMTP)

By default, development.rb will attempt to connect locally to send email.

```rb
config.action_mailer.smtp_settings = { address: "localhost", port: 1025 }
```

Set up [MailCatcher](https://github.com/sj26/mailcatcher) so the app can intercept
outbound email and you can verify what is being sent.

Note also that mail is sent asynchronously by Sidekiq, so you'll need to have it running to process jobs. Run it with this command:

```
bundle exec sidekiq
```

## Phantomjs

Needed to run javascript tests.

    cd /usr/local/share
    wget https://phantomjs.googlecode.com/files/phantomjs-1.8.2-linux-i686.tar.bz2
    tar xvf phantomjs-1.8.2-linux-i686.tar.bz2
    rm phantomjs-1.8.2-linux-i686.tar.bz2
    ln -s /usr/local/share/phantomjs-1.8.2-linux-i686/bin/phantomjs /usr/local/bin/phantomjs


# 3. Installation from script (osx only)

Obviously, if you **already** develop Ruby on OS X, a lot of this will be redundant, because you'll have already done it, or something like it. If that's the case, you may well be able to just install Ruby 2.0 using RVM and get started! Discourse has enough dependencies, however (note: not a criticism!) that there's a good chance you'll find **something** else in this document that's useful for getting your Discourse development started!

## Quick Setup

If you don't already have a Ruby environment that's tuned to your liking, you can do most of this set up in just a few steps:

Note that if you have postres installed in
usr/local/var/postgres

You'll need to remove or change that directory names or the script will fail 

if you already have phantom js in usr/local/bin you'll neeed to either
rm 'usr/local/bin/phantomjs' or just overwrite the symlink with
brew link --overwrite phantomjs

1. Install XCode and/or the XCode Command Line Tools from [Apple's developer site](https://developer.apple.com/downloads/index.action). This should also install Git.
2. Clone the Discourse repo and cd into it.
3. Run `script/osx_dev`.
4. Review `log/osx_dev.log` to make sure everything finished successfully.

Of course, it is good to understand what the script is doing and why. The rest of this guide goes through what's happening.

After running the script run 

    bundle install
  
From the root of the discourse directory and continue on from there below

## UTF-8

OS X 10.8 uses UTF-8 by default. You can, of course, double-check this by examining LANG, which appears to be the only relevant environment variable.

You should see this:

```sh
$ echo $LANG
en_US.UTF-8
```

## OS X Development Tools

As the [RVM website](http://rvm.io) makes clear, there are some serious issues between MRI Ruby and the modern Xcode command line tools, which are based on CLANG and LLVM, rather than classic GCC.

This means that you need to do a little bit of groundwork if you do not already have an environment that you know for certain yields working rubies and gems.

You will want to install XCode Command Line Tools. If you already have XCode installed, you can do this from within XCode's preferences. You can also install just the command line tools, without the rest of XCode, at [Apple's developer site](https://developer.apple.com/downloads/index.action). You will need these more for some of the headers they include than the compilers themselves.

You will then need the old GCC-4.2 compilers, which leads us to...

## Homebrew

**[Homebrew](http://mxcl.github.com/homebrew)** is a package manager for ports of various Open Source packages that Apple doesn't already include (or newer versions of the ones they do), and competes in that space with MacPorts and a few others. Brew is very different from Apt, in that it often installs from source, and almost always installs development files as well as binaries, especially for libraries, so there are no special "-dev" packages.

RVM (below) can automatically install homebrew for you with the autolibs setting, but doesn't install the GCC-4.2 compiler package when it does so, possibly because that package is not part of the mainstream homebrew repository.

So, you will need to install Homebrew separately, based on the instructions at the website above, and then run the following from the command line:

    brew tap homebrew/dupes # roughly the same to adding a repo to apt/sources.list
    brew install apple-gcc42
    gcc-4.2 -v # Test that it's installed and available

(You may note the Homebrew installation script requires ruby. This is not a chicken-and-egg problem; OS X 10.8 comes with ruby 1.8.7)

## RVM and Ruby

While some people dislike magic, I recommend letting RVM do most of the dirty work for you.

### RVM from scratch

If you don't have RVM installed, the "official" install command line on rvm.io will take care of just about everything you need, including installing Homebrew if you don't already have it installed. If you do, it will bring things up to date and use it to install the packages it needs.

    curl -L https://get.rvm.io | bash -s stable --rails --autolibs=enabled

### Updating RVM

If you do already have RVM installed, this should make sure everything is up to date for what you'll need.

    rvm get stable

    # Tell RVM to install anything its missing. Use '4' if homebrew isn't installed either.
    rvm autolibs 3

    # This will install baseline requirements that might be missing, including homebrew.
    # If autolibs is set to 0-2, it will give an error for things that are missing, instead.
    rvm requirements

Either way, you'll now want to install the 'turbo' version of Ruby 2.0.

    # Now, install Ruby
    rvm install 2.0.0-turbo
    rvm use 2.0.0-turbo --default # Careful with this if you're already developing Ruby

## Git

### Command line

OS X comes with Git (which is why the LibXML2 dance above will work before this step!), but I recommend you update to Homebrew's version:

    brew install git

You should now be able to check out a clone of Discourse.

### SourceTree

Atlassian has a free Git client for OS X called [SourceTree](http://www.sourcetreeapp.com/download/) which can be extremely useful for keeping visual track of what's going on in Git-land. While it's arguably not a full substitute for command-line git (especially if you know the command line well), it's extremely powerful for a GUI version-control client.

## Postgres 9.3

OS X ships with Postgres 9.1.5, but you're better off going with the latest from Homebrew or [Postgres.app](http://postgresapp.com).

### Using Postgres.app

After installing [Postgres.app](http://postgresapp.com/), there are some additional setup steps that are necessary for discourse to create a database on your machine.

Open this file:
```
~/Library/Application Support/Postgres/var-9.3/postgresql.conf
```
And change these two lines so that postgres will create a socket in the folder discourse expects it to:
```
unix_socket_directories = '/var/pgsql_socket'   # comma-separated list of directories
#and
unix_socket_permissions = 0777  # begin with 0 to use octal notation
```
Then create the '/var/pgsql/' folder and set up the appropriate permission in your bash (this requires admin access)
```
sudo mkdir /var/pgsql_socket
sudo chmod 770 /var/pgsql_socket
sudo chown root:staff /var/pgsql_socket
```
Now you can restart Postgres.app and it will use this socket. Make sure you not only restart the app but kill any processes that may be left behind. You can view these processes with this bash command:
```
netstat -ln | grep PGSQL
```
And you should be good to go!

#### Troubleshooting

If you get this error when starting `psql` from the command line:

    psql: could not connect to server: No such file or directory
    Is the server running locally and accepting
    connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
    
it is because it is still looking in the `/tmp` directory and not in `/var/pgsql_socket`.

If running `psql -h /var/pgsql_socket` works then you need to configure the host in your `.bash_profile`:

```
export PGHOST="/var/pgsql_socket"
````

Then make sure to reload your config with: `source ~/.bash_profile`. Now `psql` should work.


### Using Homebrew:

Whereas Ubuntu installs postgres with 'postgres' as the default superuser, Homebrew installs it with the user who installed it... but with 'postgres' as the default database. Go figure.

However, the seed data currently has some dependencies on their being a 'postgres' user, so we create one below.

In theory, you're not setting up with vagrant, either, and shouldn't need a vagrant user; however, again, all the seed data assumes 'vagrant'. To avoid headaches, it's probably best to go with this flow, so again, we create a 'vagrant' user.

    brew install postgresql # Installs 9.2
    ln -sfv /usr/local/opt/postgresql/*.plist ~/Library/LaunchAgents

    export PATH=/usr/local/opt/postgresql/bin:$PATH # You may want to put this in your default path!
    initdb /usr/local/var/postgres -E utf8
    launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.postgresql.plist

### Seed data relies on both 'postgres' and 'vagrant'
    
    createuser --createdb --superuser postgres
    createuser --createdb --superuser vagrant

    psql -d postgres -c "ALTER USER vagrant WITH PASSWORD 'password';"
    psql -d postgres -c "create database discourse_development owner vagrant encoding 'UTF8' TEMPLATE template0;"
    psql -d postgres -c "create database discourse_test        owner vagrant encoding 'UTF8' TEMPLATE template0;"
    psql -d discourse_development -c "CREATE EXTENSION hstore;"
    psql -d discourse_development -c "CREATE EXTENSION pg_trgm;"

You should not need to alter `/usr/local/var/postgres/pg_hba.conf`

## Redis

    brew install redis
    ln -sfv /usr/local/opt/redis/*.plist ~/Library/LaunchAgents
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist

That's about it.

## PhantomJS

Homebrew loves you.

    brew install phantomjs

## ImageMagick

ImageMagick is used for generating avatars (including for test fixtures).

    brew install imagemagick

## Sending email (SMTP)

By default, development.rb will attempt to connect locally to send email.

```rb
config.action_mailer.smtp_settings = { address: "localhost", port: 1025 }
```

Set up [MailCatcher](https://github.com/sj26/mailcatcher) so the app can intercept
outbound email and you can verify what is being sent.

## Setting up your Discourse

###  Check out the repository

    git@github.com:discourse/discourse.git ~/discourse
    cd ~/discourse # Navigate into the repository, and stay there for the rest of this how-to

### What about the config files?

If you've stuck to all the defaults above, the default `discourse.conf` and `redis.conf` should work out of the box.

### Install the needed gems

    bundle install # Yes, this DOES take a while. No, it's not really cloning all of rubygems :-)

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

All specs should pass

I get many failures with this script based build, which I'm working through and learning from.

### Deal with any problems which arise.

Reset the environment as a possible solution to failed rspec tests.
These commands assume an empty Discourse database, and an otherwise empty redis environment. CAREFUL HERE

    RAILS_ENV=test rake db:drop db:create db:migrate
    redis-cli flushall
    bundle exec rspec # re-running to see if tests pass

Search http://meta.discourse.org for solutions to other problems.
