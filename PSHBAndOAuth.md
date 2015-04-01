## Introduction ##

The goal of an OAuth extension is to authenticate a subscriber. The spec currently contains a mechanism to identify the hub to the subscriber, but not the other way around. To broaden the appeal of PubHubSubBub it would be useful to support use cases where the hub must authenticate who the information is delivered to. This especially makes sense when a publisher with full knowledge of the OAuth credentials is acting as their own hub.

## Signing Subscription Requests ##

The most direct approach is to require that all subscription requests are OAuth signed. However, signing the actual topic URL being subscribed to would keep changes to the PubSubHubBub protocol to a minimum. Standard OAuth signature parameters will be included as query string parameters within the topic URL. The hub will verify the signature and decide whether the subscriber is allowed to access the particular feed.

## Signing Requests from the Hub to the Subscriber ##

All outgoing requests from the hub, verification of intent, confirmation callback, refresh requests, and finally content updates, may also be OAuth signed. Standard OAuth parameters will be passed using headers to minimize changes to the subscriber’s callback URL, as described here http://oauth.net/core/1.0/#consumer_req_param. The subscriber has the option of verifying the signature, but may choose to rely on the existing security mechanisms instead, namely the X-Hub-Signature header, defined in the PubSubHubBub spec (http://pubsubhubbub.googlecode.com/svn/trunk/pubsubhubbub-core-0.2.html#authednotify).

## Verification of Intent ##

The hub should continue to make standard verification of intent requests, as clients may depend on them for tracking the state of their subscriptions.

## Conclusion ##

With this scheme the only change a subscriber will have to make to interoperate with a secure hub will be to OAuth sign the topic URL. No further burden is placed on the developer. This scheme also translates well into fetching secure feeds, if the subscriber wants to pull data identified by the topic URL.