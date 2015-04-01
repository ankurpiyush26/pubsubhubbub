# Introduction #

Hubs that implement the PubSubHubbub protocol provide near-instant feed updates to interested subscribers. Feed proxies (such as [FeedBurner](http://feedburner.google.com) and [Pheedo](http://www.pheedo.com)) provide analytics, distribution, and monetization services to publishers of feeds. So getting hubs and feed proxies to work well together is very important. Feed proxies can subscribe to hubs for source feed updates, and also can notify hubs of proxied feed updates.

There are a few problems that need to be addressed:

  * The hub spec assumes that every URL resolves to the same feed content for all subscribers; but publishers using feed proxies often conditionally redirect their feed URL to their proxied feed URL for all user agents except the feed proxy, which receives their source feed. (This allows them to continue serving their source feed in its original URL to the proxy, and serve the proxied version of the feed to subscribers that were already subscribed to the URL before it was proxied.) Since the feed proxy gets the source feed and everyone else gets the proxied feed, it's not possible for the hub to have both versions of the feed under the same URL.
  * Publishers who have set up conditional redirects as described above typically will redirect hubs to the proxied feed as well; so the feed proxy can't get updates to the source feed from the hub. On the other hand, if the conditional redirect does not apply to the hub, then any subscriber to the hub will get the non-proxied version of the feeds (which isn't what the publisher wants.)
  * A single feed may wind up being served under multiple URLs by the feed proxy; an update to the proxied feed should cause updates to subscribers of all of these URLs. But the feed proxy generally speaking doesn't know what all these URLs are, and shouldn't have to send pings for all of them. Note that this problem isn't unique to feed proxies; all platforms that support serving a single feed under different URLs share this problem.
  * A source feed may contain a hub link that can be used to subscribe to the source feed; but there may be problems if that hub link is included in the proxied feed, since the source feed's hub may not receive the appropriate updates from the feed proxy, or may not be set up to give access to feeds under the proxied feed domain.

Below I'll discuss the approach by which we are addressing these problems in [FeedBurner](http://feedburner.google.com) and the reference hub. We believe this approach can be used by other hubs and feed proxies to solve these problems.

# Approach #

  * **When using conditional redirects, hubs should be redirected too.** If a publisher has set up their feed to conditionally redirect to a feed proxy, this redirect should apply to all hubs as well; this ensures no one but the feed proxy will receive the non-proxied feed content. Generally speaking, this is what publisher platforms do anyway, so this doesn't require a change to anything.
  * **When a hub crawls a proxied feed, the proxy should treat it as a ping.** Hubs generally crawl feeds when they receive a ping; so assuming the proxy can trust the hub, any time a hub crawls a proxied feed, the proxy can assume that the hub was pinged, and can therefore treat the crawl as a ping, and go crawl the source feed. For conditionally redirected feeds, this allows feed proxies to receive a notification that the source feed has changed without being directly pinged by the publisher's platform -- the platform just pings the hub, which crawls the source feed URL, and is redirected to the proxied feed. (Note that this requires that the proxy is capable of identifying a request as having originated from a hub. We'll discuss how that could work below.)
  * **When hub links appear in the source feed, proxies should subscribe to them.** In cases where the source feed URL differs from the proxied feed URL, and the feed is not conditionally redirected, the feed proxy can receive updates to the source feed by subscribing to the hub(s) that show up in the source feed. The feed proxy can treat these updates as pings, and go crawl the source feed to get the latest content (if the proxy requires the full source feed, rather than just the deltas provided by the hub.)
  * **Feed proxies should ignore updates from hubs that came from their own servers.** Since a feed proxy isn't able to tell (and shouldn't care) whether a source feed URL will conditionally redirect hubs to the proxy, it should subscribe to hubs for updates for all source feeds that have hub links in them. This means that when the publisher conditionally redirects requests for the source feed URL to the feed proxy, the proxy will receive updates from the hub for proxied feed content. Proxies should insert an element into the feed (Atom) or channel (RSS) element in the proxied feed content that allows them to identify that content they receive from hubs has already been proxied by their service; when this element is present, the feed proxy can simply ignore the update, rather than treating it as a ping (which would lead to a ping loop.) In [FeedBurner's](http://feedburner.google.com) case, this is accomplished using the "feedburner:info" element.
  * **Pings should be rate-limited.** In order to prevent abuse / DOSing of source feeds, and to avoid ping loops involving hubs and proxies, feed proxies should limit the rate at which pings are accepted. Ideally this would be implemented as an exponential backoff, allowing for multiple updates in a short time period, while still preventing abuse and ping loops.
  * **All hub links that appear in the source feed should be carried over to the proxied feed.** A publisher that wants their source feed to be available for subscribers through a particular hub probably also wants the proxied version of the feed to be available through that hub. So the feed proxy should include all hub links that appeared in the source feed. It can choose to add other hub links; for example, [FeedBurner](http://feedburner.google.com) always adds the reference hub to the feeds it proxies. Ideally this behavior should be configurable by publishers.
  * **Hubs should accept subscription requests for proxied feeds; subscribers should try another hub if they don't.** Since the feed proxy should copy over hub links that appear in the source feed, hubs should accept subscription requests for proxied feeds, which may be served under a different domain than the source feed. If a hub doesn't allow subscription requests for the proxied feed, then a subscriber should try another hub (assuming there is another one in the proxied feed.)
  * **When feeds update, proxies should ping all hubs in the proxied feed (apart from a hub that is crawling it).** When a hub link appears in a proxied feed, that tells subscribers that they can receive updates to the feed by subscribing to that link. To make sure that hub actually receives updates, the feed proxy should send pings to any hub links that appear in the proxied feed. However, when a hub crawls a proxied feed URL, it will already receive an updated version of the feed; so a feed proxy should not send a ping to a hub when it gets crawled by that hub and notices that the proxied feed has changed. (This helps avoid ping loops.) Note that this requires that the proxy is capable of telling which hub link in the proxied feed not to send a ping for, based on the headers in the hub's request. We'll discuss how that could work below.
  * **When a hub receives a ping for a feed URL, it should crawl and update subscribers for all feeds that have the same Atom ID or channel link.** Since the same feed content can be served under different URLs, when a hub receives a ping for a URL, it should crawl that URL, look at the atom ID (Atom) or channel link (RSS) in the resulting feed, and crawl and update subscribers for all other URLs that resolve to feeds with the same Atom ID / channel link. (This requires the hub to maintain a mapping from Atom ID / channel link -> <feed URLs>.) A hub can also match up Atom and RSS versions of the same feed based on the alternate link in Atom matching the channel link in RSS. Crawling all the URLs for a given atom ID or channel link eliminates the need for a feed proxy to send a ping for every URL under which a feed may be served. **It is necessary for the hub to crawl each URL, since different URLs may resolve to different variants of the same feed.**

When things are implemented according to the rules described above, then feed proxies receive updates to source feeds from hubs (whether they are conditionally redirected or not), and hubs receive updates from feed proxies for proxied feed content.


# Identifying hubs #

In order to implement the approach above, a feed proxy must be able to identify a request as coming from a hub, and must be able to tell from the request which hub link in the proxied feed (if any) the hub uses. At present, there's nothing in the PubSubHubbub spec that allows feed proxies to do this. [FeedBurner](http://feedburner.google.com) is currently special casing this for the reference hub, but we need to come up with a more general solution, presumably by putting something special in the request headers.

One such solution would be for hubs to include in all of their requests for feeds a special X-header:

`X-Hub-Url: <hub link URL>`

The presence of such a header could indicate to a feed proxy that it was dealing with a hub, corresponding to the hub link with the specified hub link URL.

Another way of doing this would be to use web linking (see http://tools.ietf.org/html/draft-nottingham-http-link-header-09):

`Link: <hub link URL>; rel="hub"`

And yet another solution would be to modify the User Agent header for hubs to include the hub link URL according to some standard convention, e.g.:

`User-Agent: My Hub (<hub website URL>; 27 subscribers; hub=<hub link URL>)`

or perhaps just:

`User-Agent: My Hub (<hub link URL>; 27 subscribers; hub=true)`

Any of these solutions should work fine as far as feed proxies are concerned. The user agent changes should not get stripped by proxies in between the hub and the feed proxies; I'm not certain whether X-Hub-Url or Link headers would.

# Temporary solutions #

Until hubs and feed proxies change to implement the approach above, things may not work properly for all hub / proxy combinations, meaning that a subscriber may not receive updates when subscribing to a hub link in a proxied feed. To mitigate this, subscribers should ideally subscribe to all hub links that appear in a feed, and not just choose the
first hub that appears or a hub of choice.

[FeedBurner](http://feedburner.google.com) is temporarily going to include all hub links that appear in the source feed, after a hub link to the reference hub. This ensures that all subscribers that subscribe to the first hub link that appears in a feed will work while these issues are being addressed, since [FeedBurner](http://feedburner.google.com) and the reference hub already work as described.