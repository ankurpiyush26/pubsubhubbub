# Introduction #

We currently have a [hub compliance test suite](http://code.google.com/p/pubsubhubbub/source/browse/trunk/testsuite/) in trunk that you can run from the command line and it'll check that you respond properly to publish and subscribe requests based on the latest spec.

For the first version, the suite only checks the compliance of general hubs. A future compliance tester will address hubs that are tightly integrated into a CMS and only accept subscriptions for their own URLs, and/or which only trust their own internal-network publish pings. A hosted hub compliance app my pop up (perhaps you can throw this suite on Heroku and provide that for us?)

## Notes about the Hub Test Suite ##

This suite is intended to be used for hub compliancy. The test examples came directly from the current 0.1 PubSubHubbub Core working draft. However, not everything can be tested currently. This may change as the spec changes.

The suite requires Ruby and RSpec, the latter which can be installed with Ruby Gems
using "gem install rspec". RSpec is a great approach to testing and Ruby allows
for nice DSL-like testing. The interface is HTTP, so it's language independent.

Using the Test Suite:
```
$ HUB_URL=<url to hub> spec hub_spec.rb --format specdoc
```
Example:
```
$ HUB_URL=http://localhost:8000 spec hub_spec.rb --format specdoc
```
If your publish and subscribe endpoints are not /publish and /subscribe
respectively, you can specify them with the PUBLISH\_PATH and SUBSCRIBE\_PATH
environment variables along with HUB\_URL.

## Future Tests ##

The tests are currently based on the spec, but we may want to have it test other aspects of hubs. Put your ideas here:

  * good publish events are respected, even if zero subscribers

  * good publish events are respected, when >0 subscribers

  * bad publish events return proper error

  * ....