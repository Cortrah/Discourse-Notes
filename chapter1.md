# Development Setup

There are 3 options
1. Discourse with vagrant
2. Discourse with advanced vagrant
3. Advanced installation without vagrant.

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

## 2. Discourse with Advanced Vagrant

This guide is aimed at advanced Rails developers who have installed their own Rails apps before. If you are new to Rails, you are likely much better off with our **[Discourse Vagrant Developer Guide](VAGRANT.md)**.

Note: If you are developing on a Mac, you will probably want to look at [these instructions](DEVELOPMENT-OSX-NATIVE.md) as well.

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


# Building your own Vagrant VM

Here are the steps we used to create the **[Vagrant Virtual Machine](VAGRANT.md)**. They might be useful if you plan on setting up an environment from scratch on Linux:


## Base box

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

## Some basic setup:

    sudo su -
    ln -sf /usr/share/zoneinfo/Canada/Eastern /etc/localtime
    apt-get -yqq update
    apt-get -yqq install python-software-properties
    apt-get -yqq install vim curl expect debconf-utils git-core build-essential zlib1g-dev libssl-dev openssl libcurl4-openssl-dev libreadline6-dev libpcre3 libpcre3-dev imagemagick

## Unicode

    echo "export LANGUAGE=en_US.UTF-8" >> /etc/bash.bashrc
    echo "export LANG=en_US.UTF-8" >> /etc/bash.bashrc
    echo "export LC_ALL=en_US.UTF-8" >> /etc/bash.bashrc
    export LANGUAGE=en_US.UTF-8
    export LANG=en_US.UTF-8
    export LC_ALL=en_US.UTF-8

    locale-gen en_US.UTF-8
    dpkg-reconfigure locales

## RVM and Ruby

    apt-get -yqq install libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config curl build-essential git

    su - vagrant -c "sudo bash -s stable < <(curl -s https://raw.github.com/wayneeseguin/rvm/master/binscripts/rvm-installer)"
    adduser vagrant rvm
    source /etc/profile.d/rvm.sh
    su - vagrant -c "rvm pkg install libyaml"
    su - vagrant -c "rvm install 2.0.0-turbo"
    su - vagrant -c "rvm use 2.0.0-turbo --default"

    echo "gem: --no-document" >> /etc/gemrc
    su - vagrant -c "echo 'gem: --no-document' >> ~/.gemrc"

## Postgres

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

## Redis

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

## Sending email (SMTP)

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
