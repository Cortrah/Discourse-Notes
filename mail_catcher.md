# MailCatcher

Lets set up mailcatcher before setting up discourse as even the first test run will require email to work for all tests to pass.

Catches mail and serves it through a dream.

MailCatcher runs a super simple SMTP server which catches any message sent to it to display in a web interface. Run mailcatcher, set your favourite app to deliver to smtp://127.0.0.1:1025 instead of your default SMTP server, then check out http://127.0.0.1:1080 to see the mail that's arrived so far.

![MailCatcher screenshot](http://f.cl.ly/items/3w2T1p0F3g003b2i1F2z/Screen%20shot%202011-06-23%20at%2011.39.03%20PM.png)

## Features

* Catches all mail and stores it for display.
* Shows HTML, Plain Text and Source version of messages, as applicable.
* Rewrites HTML enabling display of embedded, inline images/etc and open links in a new window.
* Lists attachments and allows separate downloading of parts.
* Download original email to view in your native mail client(s).
* Command line options to override the default SMTP/HTTP IP and port settings.
* Mail appears instantly if your browser supports [WebSockets][websockets], otherwise updates every thirty seconds.
* Runs as a daemon run in the background.
* Sendmail-analogue command, `catchmail`, makes [using mailcatcher from PHP][withphp] a lot easier.
* Written super-simply in EventMachine, easy to dig in and change.
* Keyboard navigation between messages

## How

1. `gem install mailcatcher`
2. `mailcatcher`
3. Go to http://localhost:1080/
4. Send mail through smtp://localhost:1025

Use `mailcatcher --help` to see the command line options. The brave can get the source from [the GitHub repository][mailcatcher-github].

### Bundler

Please don't put mailcatcher into your Gemfile. It will conflict with your applications gems at some point.

Instead, pop a note in your README stating you use mailcatcher. Simply run `gem install mailcatcher` then `mailcatcher` to get started.

### RVM

Under RVM your mailcatcher command may only be available under the ruby you install mailcatcher into. To prevent this, and to prevent gem conflicts, install mailcatcher into a dedicated gemset and create wrapper scripts:

    rvm default@mailcatcher --create do gem install mailcatcher
    rvm wrapper default@mailcatcher --no-prefix mailcatcher catchmail

### Rails

To set up your rails app, I recommend adding this to your `environments/development.rb`:

    config.action_mailer.delivery_method = :smtp
    config.action_mailer.smtp_settings = { :address => "localhost", :port => 1025 }
