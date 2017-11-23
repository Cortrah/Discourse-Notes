# Developer Setup

There are 3 levels of instructions to choose from
1. Discourse with vagrant
2. Discourse with a custom vagrant setup
3. Installation from script (osx only)

I've had good luck with #1 on two computers, havn't tried #2 yet and have had many failed tests with #3 on one computer that I'm working through and learning from.

This is my slightly modified version of #1, all three are included in this diretory for future reference, but this one should be sufficient

## Discourse with Vagrant (quick start with vagrant)

The following instructions will automatically download and provision a virtual machine for you to begin hacking
on Discourse with:

## Install Prerequisites

1. Install Git: http://git-scm.com/downloads 
2. Install VirtualBox: https://www.virtualbox.org/wiki/Downloads
3. Install Vagrant: http://www.vagrantup.com/ (We require Vagrant 1.7.2 or later)

### Getting Started

Open a terminal

Since I am using it on the ssd (much faster) remember to change to that volume 

    cd /Volumes/Turnstyles

Clone the project: 

    `git clone https://github.com/discourse/discourse.git`

Enter the project directory: 

    `cd discourse`

### Using Vagrant

When you're ready to start working, boot the VM:
```
vagrant up
```

Vagrant may prompt you for your admin password. This is so it can mount your local files inside the VM for an easy workflow.

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

### Before starting rails create your admin account

Before starting rails use rake and the rails console to set yourself up as an administrator

    rake admin:create

### Starting Rails

Once your VM is up to date, you can start a rails instance using the following command from the /vagrant directory:

```
bundle exec rails s -b 0.0.0.0
```

In a few seconds, rails will start serving pages. To access them, open a web browser to [http://localhost:4000](http://localhost:4000) - if it all worked you should see discourse! Congratulations, you are ready to start working!

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
   
Here are some more [details on mailcatcher](mail_catcher.md)
