The quirks of the RSS standard are [well known](http://diveintomark.org/archives/2004/02/04/incompatible-rss), but it is still a very common format that many sites publish exclusively (notably, the [New York Times](http://www.nytimes.com)).

Our demo hub now supports subscribing to and publishing updates of RSS feeds. It works exactly the same way as the spec works with Atom, except the content is RSS (both incoming from the publisher and outgoing to the publisher). The subscription flow is the same for consumers. Publishing is the same for feed owners.

To enable auto-discovery of Hubs for your RSS feeds (so clients can participate in PubSubHubbub), use the `atom:link` namespace element like this:

```
<?xml version="1.0"?>
<rss xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <atom:link rel="hub" href="http://pubsubhubbub.appspot.com"/>
    ...
  </channel>
</rss>
```

Full documentation of RSS support in the spec is being tracked [in this issue](http://code.google.com/p/pubsubhubbub/issues/detail?id=67).