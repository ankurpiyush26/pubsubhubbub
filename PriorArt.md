# Prior art #

## RSS Cloud element ##

In the RSS 2.0 spec, Dave Winer defined a [Cloud element](http://cyber.law.harvard.edu/rss/soapMeetsRss.html#rsscloudInterface), which provides a way for feed publishers to ping the "cloud" when their feed is updated. Before working on the PubSubHubbub spec, we knew about RSS but had never heard of the _cloud_ element; we figure this is because nobody actually uses it.

The _cloud_ element accomplishes the _pub_ part of _pub-sub_. What's missing is the _sub_ half, which is what makes the whole system useful. Some people have said, "Hey, rssCloud did define subscriptions." In our view the subscription part of rssCloud still just solves the _publishing_ part of the problem. rssCloud only distributes light-weight pings to subscribers-- subscribers still must re-fetch the feed to determine if it's changed. This is not easy for subscribers to do, complicating their implementations, and has scalability issues because of the [thundering herd problem](http://en.wikipedia.org/wiki/Thundering_herd_problem).

In contrast, the PubSubHubbub _hub_ is a distinct component that pushes updates to subscribers in response to publisher events. Hubbub pings are "fat" and contain the full Atom or RSS body with _only_ what has changed in the feed. This reduces load on the publisher, lowers end-to-end latency, and simplifies publisher and subscriber implementations. That is what makes PubSubHubbub different: It puts [complexity in the center](DesignPrinciples.md) and provides a consistent interface for all three pieces of the solution (pub, sub, and hub). These interfaces are efficient and save bandwidth and CPU time for all parties involved. This is what is needed for Internet-scale publish and subscribe.

So we definitely give Dave credit for coming up with this _cloud_ idea, but we think PubSubHubbub takes this to the next level.


## weblogs.com pings, changes.xml ##

Like the RSS Cloud element, the [weblogs.com ping interface](http://weblogs.com/api.html#1) and [changes.xml file](http://weblogs.com/api.html#10) handle the _pub_ part of the problem, but miss the _sub_ half that makes it useful.

These ping interfaces also do not require the pinged URL to the subscribed feed itself-- a channel link is good enough. That means ping server implementors must do auto-discovery on channel links to find the actual Atom and RSS feeds to subscribe to. The [extended pinging interface](http://weblogs.com/api.html#5) allows you to supply the actual feed URL, but it's optional and [big sites](http://technorati.com/developers/ping/) don't use it. Auto-discovery is expensive (fetching, parsing) and potentially dangerous because of DoSing by proxy. The PubSubHubbub ping protocol requires the real feed URL so no auto-discovery is required.

Another issue has been that control of these existing pings are currently in the hands of just a few sites. Our goal is to make PubSubHubbub decentralized and free; anybody can run a hub.