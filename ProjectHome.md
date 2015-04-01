A simple, open, server-to-server webhook-based pubsub (publish/subscribe) protocol for any web accessible resources.

Parties (servers) speaking the [PubSubHubbub protocol](https://pubsubhubbub.googlecode.com/git/pubsubhubbub-core-0.4.html) can get near-instant notifications (via webhook callbacks) when a topic (resource URL) they're interested in is updated.

The protocol in a nutshell is as follows:

  * An resource URL (a "topic") declares its Hub server(s) in its HTTP Headers, via `Link: <hub url>; rel=”hub” `.  The hub(s) can be run by the publisher of the resource, or can be a community hub that anybody can use:   [Google's](https://pubsubhubbub.appspot.com/), or [Superfeedr](http://pubsubhubbub.superfeedr.com/).

  * A subscriber (a server that's interested in a topic), initially fetches the resource URL as normal.  If the response declares its hubs, the subscriber can then avoid [lame, repeated polling](WhyPollingSucks.md) of the URL and can instead register with the designated hub(s) and subscribe to updates.

  * The subscriber subscribes to the Topic URL from the Topic URL's declared Hub(s).

  * When the Publisher next updates the Topic URL, the publisher software [pings the Hub(s)](PublisherClients.md) saying that there's an update.

  * The hub [efficiently fetches the published resource](PublisherEfficiency.md) and multicasts the new/changed content out to all registered subscribers.

The protocol is decentralized and free.  No company is at the center of this controlling it.  Anybody can [run a hub](Hubs.md), or anybody can ping (publish) or subscribe using [open hubs](Hubs.md).

[Google](https://pubsubhubbub.appspot.com) and  [Superfeedr](http://pubsubhubbub.superfeedr.com/) offer a public and scalable open hub for anybody to use.

See the links in the sidebar at right and see this [HOWTO](http://blog.superfeedr.com/howto-pubsubhubbub/) to get started.

<a href='http://www.youtube.com/watch?feature=player_embedded&v=B5kHx0rGkec' target='_blank'><img src='http://img.youtube.com/vi/B5kHx0rGkec/0.jpg' width='425' height=344 /></a>

## Overview ##

&lt;wiki:gadget url="https://pubsubhubbub.googlecode.com/git/presentation\_gadget.xml" width="550" height="451" /&gt;