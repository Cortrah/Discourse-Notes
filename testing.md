## FakeWeb

Discourse uses the [FakeWeb](https://github.com/chrisk/fakeweb) gem to fake external web 
requests.
For example, check out the specs on `specs/components/oneboxer`.

This has several advantages to making real requests:

* We freeze the expected response from the remote server.
* We don't need a network connection to run the specs.
* It's faster.

So, if you need to define a spec that makes a web request, you'll have to record 
the real response to a fixture file, and tell FakeWeb to respond with it for the 
URI of your request.

Check out `spec/components/oneboxer/amazon_onebox_spec.rb` for an example on 
this.

### Recording responses

To record the actual response from the remote server, you can use curl and save the response to a file. We use the `-i` option to include headers in the output

    curl -i http://en.m.wikipedia.org/wiki/Ruby > wikipedia.response
