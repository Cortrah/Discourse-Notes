# Part 4 Git

### Creating your Git Repo

Once you've created your Github account, create a new repository. You can call it anything you want, but generally something that stars with discourse- is good. Make sure the repository is public. 

### Creating your local working folder

At this point I create a local directory on my computer to hold the plugin. I usually put mine in ~/code but you can put it anywhere you like on your computer:

mkdir -p ~/code/discourse-plugin-test
cd ~/code/discourse-plugin-test

Now let's follow the instructions from github to initialize the repo with a README:

echo "# discourse-plugin-test" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin git@github.com:eviltrout/discourse-plugin-test.git
git push -u origin master

Finally, create a plugin.rb file for your plugin as explained in part 131. For this example I just created a dummy one:

plugin.rb

# name: discourse-plugin-test
# about: Shows how to set up Git
# version: 0.0.1
# authors: Robin Ward

Creating a symlink

Because you followed our developer guide7 you should have a copy of discourse checked out on your computer somewhere. I checked mine out to ~/code/discourse but again you could have put it anywhere and this should still work if you adjust the following code accordingly:

cd ~/code/discourse/plugins
ln -s ~/code/discourse-plugin-test .

The above code created a symbolic link4 between your discourse code and your plugin folder. Restart your rails server and you should find your plugin is working!

The beauty of this setup is you can just check your plugin into github and not worry about the discourse codebase it lives inside. Your changes will be isolated to the plugin itself. If you need to edit discourse's code you still can, but git will track the changes separately!

I recommend using one editor window for your plugin codebase and one for Discourse itself. It is easier when you think of them as two different things.