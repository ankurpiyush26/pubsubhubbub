# Why Polling Sucks #

The current way that people get updates to feeds today on the web is polling.

That is, the subscriber sits in a loop and asks repeatedly,

```
"Anything new yet?"
"Anything new yet?"
"Anything new yet?"
```

The server (if it's smart enough to) has to check and reply,

```
"No, you have the most recent version."
"No, you have the most recent version."
"No, you have the most recent version."
```

This sucks.  It's a waste of resources, and it's impossible to schedule the polling correctly, even with adaptive rate control.  The big knob you have is balancing between:

  * Poll fast, i.e. every minute.  This wastes server resources on both sides, and a minute isn't even that fast.

  * Poll slow, i.e. every few minutes or hour or day, depending on how active that feed is.  This is nice to servers, but when the feed is updated, it'll take on average minutes, hours, or days to see it.  This is way too slow.

PubSubHubbub, like many other protocols, is a "push" protocol.  The publisher pushes to the hub, and the hub multiplexes the push out to all the subscribers.  If the publisher and hub are performing well, the subscriber could get notified within seconds (and definitely under a minute), regardless of how often the feed is updated, and using only minimal resources for all parties involved.

So that's what this is all about.

# What's wrong with feed updates every few minutes/hours/days? #

It might be fine for reading your blogs (it's obviously been fine for most people for years now), but it's not fast enough for the decentralized applications of tomorrow that people want to build.

We and others are thinking about totally decentralized social networking, with real-time updates.  Don't assume any of this is about blogs just because we're using the Atom format and Atom's typically used for blog posts.  Atom feeds can be used to represent any topics, including private person-to-person topics such as direct messages or even real-time moves in a game.