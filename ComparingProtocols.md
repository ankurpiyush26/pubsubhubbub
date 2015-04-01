# Comparison of PubSubHubbub to light-pinging protocols #

People want a comparison of the concrete differences between fat pinging (PubSubHubbub, [XMPP pubsub](http://xmpp.org/extensions/xep-0060.html)) and light pinging ([rssCloud](http://rsscloud.org/), [XML-RPC pings](http://www.xmlrpc.com/weblogsCom), [changes.xml](http://www.weblogs.com/api.html#10), [SUP](http://code.google.com/p/simpleupdateprotocol/), [SLAP](http://joecascio.net/joecblog/2009/05/18/announcing-slap/)). This document aims to construct and convey an evaluation of these protocols that's easy to understand.

The core difference is how new information from feeds is delivered from a publisher to a subscriber:

  * Light pings: Send the URL of the feed that has updated to the subscriber.
  * Fat pings: Send the updated content of the feed to the subscriber.

There is also another series of criteria to consider for each protocol. Green is good, red is bad, yellow is so-so.

| **Consideration** | **XML-RPC ping** | **changes.xml** | **SUP** | **SLAP** | **XMPP pubsub** | **rssCloud** | **PubSubHubbub** |
|:------------------|:-----------------|:----------------|:--------|:---------|:----------------|:-------------|:-----------------|
|Transport|<font color='green'>HTTP</font>|<font color='green'>HTTP</font>|<font color='green'>HTTP/HTTPS</font>|<font color='red'>UDP</font>|<font color='gold'>TCP/XMPP</font>|<font color='green'>HTTP</font>|<font color='green'>HTTP/HTTPS</font>|
|Distribution style|<font color='gold'>Ping/Poll</font>|<font color='red'>Polling</font>|<font color='red'>Polling</font>|<font color='gold'>Ping/Poll</font>|<font color='green'>Push</font>|<font color='gold'>Ping/Poll</font>|<font color='green'>Push</font>|
|Latency|<font color='gold'>Low</font>|<font color='red'>High</font>|<font color='red'>High</font>|<font color='gold'>Low</font>|<font color='green'>Minimum possible</font>|<font color='gold'>Low</font>|<font color='green'>Minimum possible</font>|
|[Thundering herd](http://en.wikipedia.org/wiki/Thundering_herd_problem)|<font color='red'>Yes</font>|<font color='red'>Yes</font>|<font color='red'>Yes</font>|<font color='red'>Yes</font>|<font color='green'>No</font>|<font color='red'>Yes</font>|<font color='green'>No</font>|
|Spamable (no topics)|<font color='red'>Yes</font>|<font color='red'>Yes</font>|<font color='green'>No</font>|<font color='green'>No</font>|<font color='green'>No</font>|<font color='green'>No</font>|<font color='green'>No</font>|
|DoSes Publishers|<font color='gold'>Preventable</font>|<font color='green'>No</font>|<font color='green'>No</font>|<font color='gold'>Preventable</font>|<font color='gold'>Preventable</font>|<font color='gold'>Preventable</font>|<font color='gold'>Preventable</font>|
|DoS Relay attacks|<font color='red'>Yes</font>|<font color='green'>No</font>|<font color='green'>No</font>|<font color='green'>No</font>|<font color='green'>No</font>|<font color='red'>Yes</font>|<font color='green'>No</font>|
|Possible to implement on $5/month hosting|<font color='red'>No<table><thead><th><font color='red'>No</th><th><font color='red'>No</th><th><font color='red'>No</th><th><font color='red'>No</th><th><font color='gold'><a href='http://rsscloud.org/walkthrough/proposedChange.html'>Maybe</a></font></th><th><font color='green'>Yes</font></th></thead><tbody>
<tr><td>Message format</td><td><font color='gold'>XML schema</font></td><td><font color='gold'>XML schema</font></td><td><font color='gold'>JSON</font></td><td><font color='red'>Binary packet</font></td><td><font color='red'>Complex XMPP</font></td><td><font color='gold'>XML schema</font></td><td><font color='green'>Original RSS or Atom content</font></td></tr>
<tr><td>Secure notifications</td><td><font color='red'>No</font></td><td><font color='red'>No</font></td><td><font color='gold'><a href='http://friendfeed.com/simple-update-protocol/c3372901/it-is-possible-given-sufficient-resources-or'>Somewhat</a></font></td><td><font color='red'>No</font></td><td><font color='green'>Yes</font></td><td><font color='red'>No</font></td><td><font color='green'>Yes</font></td></tr>
<tr><td>Publisher complexity</td><td><font color='gold'>XML-RPC client</font></td><td><font color='gold'>XML-RPC client</font></td><td><font color='gold'>SUP IDs</font></td><td><font color='red'>UDP send</font></td><td><font color='red'>XMPP send</font></td><td><font color='gold'>XML-RPC</font>/<font color='green'>REST ping</font></td><td><font color='green'>REST ping</font></td></tr>
<tr><td>Subscriber complexity</td><td><font color='red'>Crawl pipeline</font></td><td><font color='red'>Crawl pipeline</font></td><td><font color='red'>Crawl pipeline</font></td><td><font color='red'>Crawl pipeline</font></td><td><font color='gold'>XMPP client</font></td><td><font color='red'>Crawl pipeline</font></td><td><font color='green'>Simple webapp</font></td></tr></tbody></table>


<br />
The rest of this document will compare light and fat pinging by these metrics:<br>
<br>
<ul><li><a href='#Latency.md'>Latency</a>
</li><li><a href='#Bandwidth.md'>Bandwidth</a>
</li><li><a href='#CPU_Usage.md'>CPU Usage</a>
</li><li><a href='#Publisher_complexity.md'>Publisher complexity</a>
</li><li><a href='#Subscriber_complexity.md'>Subscriber complexity</a>
</li><li><a href='#Hub_complexity.md'>Hub complexity</a></li></ul>

<hr />

<h2>Latency</h2>

To simplify this explanation, latency is represented as network "hops": the time it takes on average for data to propagate between two Internet nodes.<br>
<br>
<b>Naively, light pings and fat pings look the same:</b>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_naive_pings.png' />

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/pshb_naive_pings.png' />

There are four network hops required to deliver new content to a subscriber.<br>
<br>
<b>However, this leaves light pinging hubs open to relay denial of service attacks, so the Hub must verify there is new content:</b>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_proxy.png' />

<ul><li>Result:<br>
<ul><li>Light pings require at <b>least</b> six network hops.<br>
</li><li>Fat pings require at <b>most</b> four network hops.<br>
</li><li>Fat pinging is 33% faster than naive light pinging.</li></ul></li></ul>

<b>Often publishers will be combined with their own hub</b> (for better integration with their application, better statistics gathering, optimizations) <b>yielding:</b>

<ul><li>Light ping: 3 hops.</li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_combined_publisher.png' />

<ul><li>Fat ping: 1 hop.</li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/pshb_combined_publisher.png' />

<ul><li>Result: Fat ping is 66% faster.</li></ul>

<b>With popular sites, a feed will be served from multiple datacenters:</b>

<ul><li>Light ping case:<br>
<ul><li>Wait for propagation delay to fill <b>all</b> caches before sending pings, meaning the whole system operates as fast as the slowest node<br>
</li><li>All caches represent a dependent point of failure, meaning more waiting and retries</li></ul></li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_cache_sync.png' />

<ul><li>Fat ping case:<br>
<ul><li>Integrated hub may immediately send fat pings before feeds are updated externally</li></ul></li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/pshb_cache_sync.png' />

<ul><li>Result:<br>
<ul><li>Latency incurred by caching/replication delays can be <b>zero</b> with fat pings.<br>
</li><li>In the light pinging case, it is <b>always</b> non-zero.<br>
</li><li>This is irrelevant for single-host sites, but it gets worse and worse the bigger a site is.<br>
</li><li>Mitigating this problem for light pings requires specialized knowledge of datacenter topology, which violates the abstraction of DNS.</li></ul></li></ul>


<hr />

<h2>Bandwidth</h2>

Assume an average feed is 100KB consisting of fifty 2KB posts.<br>
<br>
<b>Take the case of a single new item with 2KB of data and 100 subscribers to the feed:</b>

<ul><li>Light ping: 10MB served by publisher</li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_bandwidth.png' />

<ul><li>Fat ping: 200KB served by publisher</li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/pshb_bandwidth.png' />

<ul><li>Result:<br>
<ul><li>Fat ping requires 98% less data (i.e., light ping requires 50x more).<br>
</li><li>Light pinging requires publisher to serve 100x more HTTP requests (the <a href='http://en.wikipedia.org/wiki/Thundering_herd_problem'>"thundering herd"</a>).<br>
</li><li>This result remains the same even with 1,000,000 subscribers to a feed.</li></ul></li></ul>

<b>To prevent denial of service attacks, light pinging Hubs must verify there is new content:</b>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_proxy_bandwidth_slow.png' />

<ul><li>Result:<br>
<ul><li>Even worse than naive light pinging case.<br>
</li><li>Same bandwidth overhead as naive light pings.<br>
</li><li>Fat pinging is 33% faster than light pinging.</li></ul></li></ul>

<b>Light-ping advocates suggest that the Hub should re-serve only the new content on behalf of the publisher:</b>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_proxy_bandwidth.png' />

<ul><li>Result:<br>
<ul><li>Equal bandwidth as fat pings.<br>
</li><li>Still 100x as many incoming HTTP requests as fat pings.<br>
</li><li>33% more latency than naive fat pings.<br>
</li><li>66% more latency than combined publisher/hub fat pings.<br>
</li><li>Trust/security model for proxied feed on behalf of publisher unclear.</li></ul></li></ul>

<hr />

<h2>CPU Usage</h2>

Assume parsing a whole feed on average takes 10ms per item. Again assume an average feed has 25 items.<br>
<br>
<b>Take the case of a single new item being sent to 100 subscribers to the feed with naive pings:</b>

<ul><li>Light ping: 25 seconds of CPU time consumed by subscribers (250ms each).</li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_cpu.png' />

<ul><li>Fat ping: 1.25 seconds of CPU time consumed total; 250ms by the hub, 10ms by each subscriber.</li></ul>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/pshb_naive_cpu.png' />

<ul><li>Result:<br>
<ul><li>Fat pings require 95% less CPU (i.e., light pinging requires 20x).<br>
</li><li>Fat pings 99.6% cheaper for subscribers (i.e., light pinging requires 25x).<br>
</li><li>Light pinging requires publisher to serve 100x more HTTP requests (the <a href='http://en.wikipedia.org/wiki/Thundering_herd_problem'>"thundering herd"</a>) which have overhead.<br>
</li><li>This result remains the same even with 1,000,000 subscribers to a feed.</li></ul></li></ul>

<b>However, the Hub must verify the feed is new to make this safe:</b>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_proxy_cpu_slow.png' />

<ul><li>Result:<br>
<ul><li>Light pings require 25.25 CPU seconds consumed; 250ms by the hub, 250ms for each consumer<br>
</li><li>Even worse than the naive case.</li></ul></li></ul>

<b>And when, for light pings, Hubs re-serve only the new content on behalf of the publisher:</b>

<img src='http://pubsubhubbub.googlecode.com/svn/trunk/media/light_pings_proxy_cpu.png' />

<ul><li>Result:<br>
<ul><li>Equal CPU as fat pings.<br>
</li><li>Still 100x as many incoming HTTP requests as fat pings.<br>
</li><li>33% more latency than naive fat pings.<br>
</li><li>66% more latency than combined publisher/hub fat pings.<br>
</li><li>Trust/security model for proxied feed on behalf of publisher unclear.</li></ul></li></ul>

<hr />

<h2>Publisher complexity</h2>

Assume the publisher tells hubs the feed URLs.<br>
<br>
<ul><li>Light ping: Send a feed URL in some format.<br>
</li><li>Fat ping: Send a feed URL in some format.<br>
</li><li>Result: More or less equivalent, though some interop issues (e.g., SOAP) could exist.</li></ul>

<hr />

<h2>Subscriber complexity</h2>

<ul><li>Light ping<br>
<ul><li>Subscription protocol code<br>
</li><li>Feed fetching pipeline<br>
</li><li>Feed parsing code</li></ul></li></ul>

<ul><li>Fat ping<br>
<ul><li>Subscription protocol code<br>
</li><li>Parsing code</li></ul></li></ul>

<ul><li>Result<br>
<ul><li>Fat pinging does not require the complexity of a feed fetching pipeline.<br>
</li><li>Significant because efficiently doing feed fetches in an asynchronous way can be hard, if not impossible for simple hosting providers.</li></ul></li></ul>

<hr />

<h2>Hub complexity</h2>

Assuming that Hubs must verify that the original feed has changed or else they will just be an open relay for DoS attacks.<br>
<br>
<ul><li>Light ping:<br>
<ul><li>Receive ping<br>
</li><li>Verify feed document has changed (one hash of the text contents)<br>
</li><li>List subscribers<br>
</li><li>Send ping to subscribers</li></ul></li></ul>

<ul><li>Fat ping:<br>
<ul><li>Receive ping<br>
</li><li>Parse feed document, determine if individual entries have changed (multiple hashes, one for each item)<br>
</li><li>List subscribers<br>
</li><li>Send new content to subscribers</li></ul></li></ul>

<ul><li>Result:<br>
<ul><li>Roughly the same.<br>
</li><li>Fat pinging requires a reparse of the feed on each publish notification, light pinging does not; they only need to check a hash of the whole content has changed.<br>
</li><li>Light ping requires only one hash per feed instead of one per item for fat pings.  Storage usage can be mitigated by fat pinging by having a cap on total storage allowed per feed.