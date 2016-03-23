### plugin.rb
When Discourse starts up, it looks in the plugins directory for subdirectories containing a plugin.rb file. The plugin.rb file has two purposes: it is the manifest for your plugin with the required information about your plugin including: its name, contact information and a description. The second purpose is to initialize any ruby code necessary to run your plugin.

In our case, we won't be adding any ruby code but we still need the plugin.rb. Let's create the directory basic-plugin with the file plugin.rb inside it, with the following contents:

### basic-plugin/plugin.rb
    # name: basic-plugin
    # about: A super simple plugin to demonstrate how plugins work
    # version: 0.0.1
    # authors: Awesome Plugin Developer

Once you've created this file, you should restart your rails server and the plugin should be loaded.

### An important Gotcha!
If you're used to regular rails development you might notice that plugins aren't quite as nice when it comes to reloading. In general, when you make changes to your plugin, you should Ctrl+c the server to stop it running, then run it again using bundle exec rails server.

### My changes weren't picked up! warning
Sometimes the cache isn't cleared fully, especially when you create new files or delete old files. To get around this issue, remove your tmp folder and start rails again. On a mac you can do it in one command: 

    rm -rf tmp; bundle exec rails s.


### Checking that your plugin was loaded
Once you've restarted your rails server, visit the url /admin/plugins (make sure you're logged in as an admin account first, as only admins can see the plugin registry).

If everything worked, you should see your plugin in the list:

Congratulations, you just created your first plugin!

### Let's add some Javascript
Right now your plugin doesn't do anything. Let's add a javascript file that will pop up an alert box when discourse loads. This will be super annoying to any user and is not recommended as an actual plugin, but will show how to insert Javascript into our running application.

Create the following file:

_plugins/basic-plugin/assets/javascripts/discourse/initializers/alert.js.es6_

    export default {
      name: 'alert',
      initialize() {
        alert('alert boxes are annoying!');
      }
    };

Now if you restart your rails server, you should see "alert boxes are annoying!" appear on the screen. Let's step through how this worked:

1. Javascript files placed in assets/javascripts/discourse/initializers are executed automatically when the Discourse application loads up.

2. This particular file exports one object, which has a name and an initialize function.

3. The name has to be unique, so I just called it alert.

4. The initialize() function is called when the application loads. In our case, all it does is execute our alert() code.

You're now an official Discourse plugin developer! 
