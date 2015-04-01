DRAFT PROPOSAL, UNFINISHED.

This describe a potential extension of the Core Protocol to enable publisher to preemptively allow specific subscriptions.

## Introduction ##

There is a true concern by many publishers on who has access to their content. For example, spammers could use PubSubHubbub to duplicate original content at very low cost from well-known publishers.
Another concern is to allow subscriptions only on a specific set of feeds. The hub might not always know the which feed urls it has to accept subscriptions on. This mechanism can allow the hub to say if he is in charge of a given feed.

The purpose of this extension is NOT to provide any copyright protection mechanism. It's the publisher's responsibility to acknowledge that this mechanism will not prevent subscribers to poll feeds on their own.

This extension aims at providing an OPTIONAL mechanism to let the publisher know about subscriptions on a hub.

It's based on callback happening during the subscription mechanism so that publishers can accept or refuse specific subscriptions. They can also ask for more information about the subscriber to make that decision.

## Details ##

Whenever a subscriber sends a subscription query to a hub, the hub MUST fetch the feed to identify a "publisher endpoint" to which it should forward the subscription parameters. The publisher MAY define this publisher endpoint url, either via an HTTP header, or via a specific 

&lt;link&gt;

 element inside the feed itself. If no publisher endpoint is found, the hub MUST consider that the publisher accepts any subscription provided that other core parameters are provided.

The hub documentation MAY indicate that the subscription information will be sent to the publisher.

Once this "publisher endpoint" url is identified, before checking the subscriber's intent, the hub should send a POST request the subscription query to the "publisher endpoint", with a `hub.mode=authorize`, as well as the subscription parameters (hub.topic and hub.callback). The hub MUST also include any additional parameters provided by the subscriber.

The publisher MAY accept or refuse the subscription.

If the subscription is accepted it MUST return a 200 response, with an empty body. When the publisher accepted the subscription, the hub MUST proceed and resume the normal subscription process described in the core specification.

If the subscription is refused, publisher MUST return a 401 response (any response which is not 200 MUST be considered as a failure). The publisher MAY include additional information in the body. This information MAY contain an explanation why the subscription was refused, or instructions to get the subscription accepted (list of required additional parameters). When the publisher refused the subscription. The hub MUST then return a 422 response to the subscriber. If a body was included, the hub MUST return it as well to the subscriber. The subscriber MAY then re-try its subscription with any missing parameters.