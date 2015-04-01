PubSubHubbub provides high-throughput event delivery for subscribers by aggregating multiple Atom feeds together using the _atom:source_ element. This enables a single HTTP POST to contain a lot of data for the single subscriber.

But what about the Publisher side? One question I've heard from people is, _"How can this be efficient if the Hub has to re-pull the feed on every publish event?"_ This document attempts to answer this question.


---


## HTTP persistent connections and pipelining ##

By default in HTTP 1.1, TCP connections will be reused between requests. For a publisher serving many separate Atom feeds, this allows Hubs to get around the expense of creating a new TCP connection every time an event happens. Instead, Hubs MAY leave their TCP connections open and reuse them to make HTTP requests for freshly published events.

[Pipelining](http://en.wikipedia.org/wiki/HTTP_pipelining) allows a single TCP connection to make a series of HTTP requests simultaneously in batch. This reduces the round-trip time of the HTTP requests and maximizes the data sent per packet, cutting down on the number of total packets sent and received. To this end, both Hubs and Publishers SHOULD support HTTP piplining.


---


## Feed windowing ##

Publishers SHOULD properly supply the caching headers _Last-Modified_ and _ETag_ for their Atom feed responses. Hubs SHOULD respect these fields and supply them properly on requests (receiving a 304 response if no content has changed). The great thing about ETag headers is that they are opaque and must be reproduced by the requesting client exactly. That means Publishers can encode information in the ETag to represent the state of the requesting client.

For example, as a Publisher you only want to supply the Hub with the content that has changed since the Hub last pulled the feed. You can do this by encoding the ID of the newest _atom:entry_ from the feed in the ETag. Then, when the Hub requests again, you can use the ID in the ETag to formulate the query for the newest content, or as a cache key for an already-cached result.

The time-line would look like this:
  1. Publisher adds new entries to feed X, sends Hub a publish event
  1. Hub pulls feed X, gets entries (1,2,3,4) and ETag of 4
  1. (later) Publisher publishes adds to feed X again, sends Hub an event
  1. Hub pulls feed X with If-None-Match = 4; Publisher serves content _after_ 4: (5,6,7)

To make this safe, Publishers should be sure to cryptographically sign their ETag header values. The ETag value could then be _entry\_id:hmac_. To do this in Python you would use this code:

```
import hmac
entry_id = '1234 entry id'
signature = hmac.new('secret key', entry_id).hexdigest()
header_value = '%s:%s' % (entry_id, signature)
```

For additional HTTP correctness, publishers SHOULD ensure they serve a `Vary: If-None-Match` or `Vary: If-Modified-Since` header in their response to indicate the [cachability of the response](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.44).


---


## Atom feed archiving ##

[RFC5005](http://www.ietf.org/rfc/rfc5005.txt) defines a convention for having archives of feeds organized in chunks of an arbitrary size based on time. Using this convention, publishers can reduce the size of their "latest items" feed, and thus save bandwidth and CPU serving the latest contents to Hubs.


---


## Serving feeds from multiple datacenters ##

One thing many large publishers do is serve their feeds from multiple datacenters at the same time. Often this is done using master-slave replication, but this also applies to master-master replication. Essentially, each feed has a primary datacenter where all writes go. After the write, the data is replicated to all other datacenters in a lazy fashion (eventually consistent).

In the [world of RSS pings](PriorArt.md), replication causes a significant issue:
  1. Publisher adds content to a feed in the master datacenter
  1. Publisher sends a ping to Weblogs.com, etc
  1. Feed consumers pull the feed from the publisher
  1. Due to geographical locality of each feed consumer:
    * Some consumers hit the master datacenter-- they will see the new content
    * Other consumers hit the read-only datacenters that are behind on replication-- they will see no changes!

The race condition here shows that the model of sending pings with only the feed URL is **flawed**. It can only provide low-latency for for singly-homed services running in one datacenter. For large-scale systems run by many companies out there, this can't work reliably. This problem is exacerbated by transparent proxies and edge caches.

PubSubHubbub to the rescue! In the Hubbub model, the Hub can be integrated into the publisher's content management system itself. This has some large benefits that overcome the limitations of traditional pinging:
  * The Hub can be configured to always pull from the master for each feed
    * This means event notifications may be delivered to subscribers _before_ the feed update is visible to traditional polling feed consumers
  * Multi-homed providers don't have to worry about cache invalidations and situations where replication delays are elevated (due to networking issues, etc)
  * Notification latency can be lower because the feed update doesn't have to propagate to all feed servers before sending out pings
  * Notifications may be distributed even in the case that feed serving is down!