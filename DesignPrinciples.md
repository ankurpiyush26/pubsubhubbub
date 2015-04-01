# Introduction #

We wanted to give some background on the design principles which shape the PubSubHubbub protocol.

  * **Keep pub & sub simple.  Complexity at the hub**.  There will be tons of publishers and tons of subscribers.  We want to keep that process simple.  Relatively few people will run hubs, and those who do will know what they're doing.  As such, we push complexity to the middle (the hub) wherever possible.

  * **Pragmatic.**  This isn't a perfect protocol, or theoretically pure.  What it is, however, is pragmatic, solving huge, known use cases with minimum effort.  The use of Atom could've been avoided, for instance, but Atom doesn't actually bring much overhead and gives us everything we'd need anyway (unique ID, date, opaque content body).

  * **Bootstrappable**.  We wanted to solve all chicken-and-egg problems and make something that people could start using right away with minimal efforts.  This includes:
    * _Our open, reference hub_.  The hub we're running at http://pubsubhubbub.appspot.com is Open Source, and open for anybody to publish to or subscribe from.  This lets people start experimenting with the protocol immediately, without running a hub, or waiting for others to run a hub.
    * _Bookmarklet_.  In lieu of your favorite publisher/CMS actually pinging a hub, we have a [bookmarklet](https://pubsubhubbub.googlecode.com/git/bookmarklet/bookmarklet_config.html) that will speak the _Pub_ part of the protocol for you.

  * **...**

TODO: expand this.

See also: <a href='http://moderator.appspot.com/#15/e=43e1a&t=426ac'>live FAQ</a>.  Go add a question!