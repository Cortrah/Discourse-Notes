# Part Two Plugin Outlets

### Getting Started: Handlebars Templates

Discourse's client application is written using the Ember.js Javascript framework. Ember uses Handlebars54 for all HTML templates. There's a great introduction to the templating language at that link, so definitely read it thoroughly.

### The Problem: Adding elements to the Discourse User Interface

Many plugins need to add and extend the Discourse web interface. We provide a mechanism to do this called plugin outlets in handlebars templates.

If you browse the discourse handlebars templates, you'll often see the following markup:

    {{plugin-outlet "edit-topic"}}

This is declaring a plugin outlet called "edit-topic". It's an extension point in the template that plugin authors can leverage to add their own handlebars markup.

When authoring your plugin, look in the discourse handlebars templates (in .hbs files) you want to change for a {{plugin-outlet}}. If there isn't one, just ask us to extend it! We'll happily add them if you have a good use case. If you want to make it easier and faster for us to do that, please submit a pull request on github!

If you want to see some of the places where plugin outlets exist, you can run the following command if you're on OSX or Linux:

    $ git grep "plugin-outlet" -- "*.hbs"

@Mittineague has also written a plugin68 to show their locations. I have to admit I have not tried it out myself so I'm not sure if it works, but it looks like it could be useful!

### Connecting to a Plugin Outlet

Once you've found the plugin outlet you want to add to, you have to write a connector for it. A connector is really just a handlebars template whose filename includes connectors/<outlet name> in it.

For example, if your handlebars template has:

{{plugin-outlet "evil-trout"}}

Then any handlebars files you create in the connectors/evil-trout directory
will automatically be appended. So if you created the file:

plugins/hello/assets/javascripts/discourse/templates/connectors/evil-trout/hello.hbs

With the contents:

<b>Hello World</b>

Discourse would insert <b>Hello World</b> at that point in the template.

Note that we called the file hello.hbs -- The final part of the filename does not matter, but it must be unique across every plugin. It's useful to name it something descriptive of what you are extending it to do. This will make debugging easier in the future.
Troubleshooting

    Double check the name of the connector and make sure it matches the plugin name perfectly.

    When in doubt, rm -rf tmp and start your development server again.


