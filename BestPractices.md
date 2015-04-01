# Publisher best practices #

  * Publishers may add references and publish to more than one hub for better reliability. This is particularly be useful if a hub goes down, in maintenance or just discontinued. However, this reliability is different than what's required for MovingFeedsOrChangingHubs.

  * Publishers should retry pings to their Hubs when they publishing in case of failure to make sure the notifications are eventually sent. Hubs also may respond with 503 errors on notification, so be sure to respect the `Retry-After` header.

  * Publishers should serve caching headers and follow these suggestions about PublisherEfficiency.

# Subscriber best practices #

  * Subscribers must be low-latency. They should not fully process a notification upon delivery, but store it somewhere for processing asynchronously at a later time. The 200 response returned by a subscriber to the Hub is a receipt of the delivery of the message, not an acknowlegement that it was properly handled. For example, a subscriber should not parse the whole notification message in a synchronous manner upon event delivery; it should save the bytes and process it later; if the notification payload was actually invalid XML, the subscriber should _still_ return a 200 code because the response code is only meant to indicate the receipt of the message, not the correctness of the message.

  * Subscribers must use the content payload from the trusted hubs delegated by a feed as authoritative. They cannot use a new content notification as another form of light pinging. This is important because Hubbub notifications may deliver content before the public feed has it; this means polling after a notification may result in no visible changes! See this wiki page on [serving feeds from multiple datacenters](http://code.google.com/p/pubsubhubbub/wiki/PublisherEfficiency#Serving_feeds_from_multiple_datacenters) to see why.

  * Subscribers should consider subscribing to all the Hubs referenced by a publisher in a feed. That would guarantee that if a hub doesn't receive a publication, it might still be received from another hub. The trade-offs here are bandwidth, CPU usage, and the complexity of de-duping repeated messages from both hubs.

  * Subscribers can encode state and context information in their callback URLs. For example, the URL could encode a local reference to the feed to identify it easily upon notification (instead of having to reparse the whole feed URL).

  * Subscribers should keep their hubbub notification URLs private.

  * Subscribers receiving notifications on behalf of many users should only maintain one subscription for all users and do fan-out locally, instead of maintaining a separate subscription on hubs for all end-users that will receive a notification. This greatly reduces the number of notifications and subscriptions required by the whole system.

# Hub best practices #

  * Hubs should retry publications of new content multiple itmes if the initial notification failed.

  * Hubs may subscribe to the hub indicated by a feed if it's not them, on behalf of their users. This is especially useful in the case of federated hubs that add more value on top of simple Hubbub notifications. However, when doing this hubs should only subscribe once for all of their local subscribers.

  * Publishers integrating a hub into their CMS should consider the technique of [fat pinging](http://code.google.com/p/pubsubhubbub/source/browse/trunk/nonstandard/fat_publish.py) (which is not in the spec) to help work around their own replication delays.