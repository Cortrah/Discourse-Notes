# Part Six Aceptance Tests

Did you know that Discourse has two large test suites for its code base? On the server side, our Ruby code has a test suite that uses rspec. For the browser application, we have a qunit suite that has ember-testing included.

Assuming you have a development environment set up, if you visit the http://localhost:3000/qunit URL you will start running the Javascript test suite in your browser. One fun aspect is that you can see it testing the application in a miniature window in the bottom right corner:

### Adding an Acceptance Test in your Plugin

First, make sure you have the latest version of Discourse checked out. Being able to run Acceptance tests from plugins is a relatively new feature, and if you don't check out the latest version your tests won't show up.

For this article I am going to write an acceptance test for the purple-tentacle plugin that we authored in part 5 of this series.

Adding an acceptance test is as easy as adding one file to your plugin. Create the following:

_test/javascripts/acceptance/purple-tentacle-test.js.es6_

    import { acceptance } from "helpers/qunit-helpers";
    acceptance("Purple Tentacle", { loggedIn: true });

    test("Purple tentacle button works", () => {
      visit("/admin/plugins/purple-tentacle");

      andThen(() => {
        ok(exists('#show-tentacle'), "it shows the purple tentacle button");
        ok(!exists('.tentacle'), "the tentacle is not shown yet");
      });

      click('#show-tentacle');

      andThen(() => {
        ok(exists('.tentacle'), "the tentacle wants to rule the world!");
      });
    });

I tried to write the test in a way that is clear, but it might be a little confusing if you've never written an acceptance test before. I highly recommend that you read the ember docs on acceptance testing as they have a lot of great information.

The first line of importance is visit("/admin/plugins/purple-tentacle");. This tells our test to navigate to that URL in our application. That URL was the one that displays the tentacle.

The next bit andThen() is a piece of code that will execute after the page has fully loaded. This might be a little confusing if you've not done much asynchronous programming before. Browser Applications like Discourse are built using javascript promises. Often, performing an action means that the Javascript application will have to contact the server and wait for a reply.

Visiting a URL almost always involves fetching data from a server, so in Ember all requests are considered asynchronous. That means that we need to write some code that will be called when the routing is finished and ember has loaded the URL. That's where andThen() comes in.

It almost reads like english doesn't it? "Visit a URL and THEN perform some tests".

In this case, I am checking that when the page is initially visited, the #show-tentacle button exists, and that the purple tentacle is hidden. (P.S. the previous version of the purple-tentacle plugin didn't have the #show-tentacle element id in the handlebars template. Check out the latest version to follow along!)

Once those tests pass it's time to test the interaction. The next command is click('#show-tentacle'); which tells our testing framework that we want to click the button and show the tentacle.

Finally, we have another andThen function to wait until whatever processing happens when the button is clicked, and then makes sure that the purple tentacle is present.

Not too bad is it? You can try the test yourself by visiting http://localhost:3000/qunit?module=Acceptance%3A%20Purple%20Tentacle on your development machine. You should very quickly see the purple tentacle appear and all tests will pass.

### Where to go from here

I hate to sound like a broken record but the Ember documentation on testing is excellent. You might also want to see how Discourse tests various functionality by browsing the tests in our javascript tests directory. We have quite a few examples in there you can learn from.