## Preconditions ##

One important assumption these flows make is that hubs will send **empty** feed payloads to subscribers in the event that any of the feed's meta-data has changed. This is the portion of the feed outside the `entry` or `item` elements that includes any `atom-link` elements.


---


## Moving Feeds ##

Often when a company redesigns their site they also redo their URL scheme. This includes moving feed URLs to different locations. This has the potential to break feed subscribers. Currently the de facto standard is to have a host serve a 302 or 301 redirect on the old feed URL that directs to the new one. This is not just used for moves; FeedBurner's entire proxying model works by this mechanism.

For polling clients this redirect scheme is fine. The poller will fetch the feed, get the redirect, then fetch the feed from the new location. From the standpoint of the end-user of a feed reader, the feed has never moved and there are no problems. Feed readers may cache the new feed location or update their configuration to never pull from the old location again (in the case of a 301 permanent redirect).

In the Hubbub world, this is potentially more difficult because a topic URL represents a stable location for updates. Consider an example where we have two topic URLs (topic A, topic B) that are actually the same feed that has been moved. For this example, say A is the old location and B is the new one.

  1. Originally, subscriber used the hub to subscribe to topic A
  1. Publisher adds new content to topic A, pings the hub for topic A
  1. Hub contacts the subscriber with the events for topic A
  1. Publisher decides to move their feed
    * Serves topic A as a permanent redirect to topic B
    * Publisher updates the `atom:link[@rel="self"]` element in the feed to point to topic B's URL, the new canonical address
  1. When publishing to topic B, the publisher will send events to the hub for _both_ topic B _and_ topic A
    * Hubs MUST follow redirects from topic A to topic B
    * Old subscribers on topic A will still get events as long as the redirect is in place and the publisher pings both A and B on new events
  1. Subscriber gets update event with `atom:link` for topic B, the new canonical URL
    * Subscriber knows it wasn't subscribed to topic B
    * Subscriber knows that only the hub has its subscriber notification URL
    * Subscriber recognizes that this must be a feed move
  1. Subscriber sends a request to the feed's hub to subscribe to topic B
  1. Subscriber starts receiving events for both topic A and topic B
    * De-dupes the events is easy since the `atom:link` link and content for both feeds will be the same.
  1. (optionally) After N days (say, 14), publisher stops serving events to both feeds
  1. (optionally) Subscriber re-fetches all feeds periodically to see if any `atom:link` addresses have changed


---


## Changing Hubs ##

Multiple hubs may be specified in the feed body of a single topic. This means the publisher may send publish events to multiple hubs at the same time. This is useful for enabling subscribers to prefer a particular hub for whatever reason (e.g., extra features, latency guarantees). There are situations where a publisher may decide they no longer want to publish their feed through a certain hub (e.g., that hub has poor performance). How do you stop using a hub while ensuring all existing subscribers move to the new one?

For this example, assume we have two hubs, A and B. A is the old hub, B is the new hub.
  1. Publisher publishes to hub A
    * Feed has a `atom:link[@rel="hub"]` element pointing to hub A
  1. When new content is added to the feed, publisher sends an event to hub A
  1. Publisher decides to move to hub B
    * Feed changed to only include a `atom:link[@rel="hub"]` for hub B; hub A reference is removed
  1. Publisher continues to notify hub A _and_ hub B about new content
  1. Subscriber receives notification with new hub link specified
    * Subscribers know what hub they use to subscribe to each topic URL
    * Subscriber notices that the hub they were subscribed with is no longer listed
    * Subscriber recognizes that this is a hub change
  1. Subscriber contacts new hub B to create a new subscription
  1. Subscriber starts receiving events for the topic from both hub A and B
    * De-dupes the events is easy since the `atom:link` link and content for both feeds will be the same
  1. (optionally) After N days (say, 14), the publisher stops contacting hub A