# Introduction #
The PubSubHubbub protocol works very well for subscribing to get PUSH updates for Atom or RSS feeds. These formats are built on top of XML are easily extended using namespaces. In addition they provide a timestamp and ids for all the entries so it easy to syndicate deltas and updates to previously syndicated entries.
This proposal allows a consumer to subscribe to any resource that can be identified with a URL (even if the resource is a jpg) and be notified of a variety of changes to the resources. This proposal also generalizes the notion of feeds to support more underlying formats than xml.

The main use case being targeted is APIs with JSON output but we wanted to keep this simple and generic so the proposal is content agnostic.

# Details #

## Discovery ##

To make any resource available via PUSH you first need to find at least one hub to push the changes to and let subscribers know where this hub is located.
This will be done using HTTP Headers on the response as described in the Web Linking http://tools.ietf.org/html/draft-nottingham-http-link-header-10 specification

Example:

HTTP/1.1 200 OK

Content-Type: application/json; charset=UTF-8

Date: Sun, 06 Jun 2010 16:15:03 GMT

Content-Length: 268

Link:<http://hub.company.com/>;rel="hub"

Link:<http://hub.organization.org/>;rel="hub"

Additional discovery mechanism can be supported on a Content-Type basis. For example for application/atom the Link element can still be used to convey rel="hub" and in application/stream+json the links array can have a key "hub" which will contain the value of the hub

## Anatomy of PuSH Request ##
When the content changes, just like before, the publisher sends a light ping to the hub indicating to refetch the resource. The hub verifies the update request is legitimate by comparing the HTTP response to the previous one. If the Date header is newer then this is considered an update.

### Option 1 : RESTful Approach ###
The hub then proceeds to notify all the subscribers copying all the HTTP headers which it got from the publisher that contain details about the resource such as:

  * Content-Type
  * Content-Length
  * Date
  * E-Tag

To the HTTP POST request it will make to the subscriber.

In addition the POST request will continue to include the X-Hub-Signature header for the entire message including headers, which subscribers can use to validate the authenticity and include a new Link rel="self" header which indicates the resource that changed.

  * Link:<http://my.site.com/page.html>; rel="self"

The subscriber can then use this information to update their data repository even if updates are syndicated out of order. The unique identifier is the self URL and the date of the change is conveyed via the Date header.

#### Modeling different resource types ####

A URL can point to a single resource or a list of resources. When it points to a list as in the case of a feed, the updates sent from the hub to the publishers are deltas and the subscriber can update their cached copy by correlating identifiers of items in the list.

If the hub understands the Content-Type and Response Status Codes from the publisher it can give more details to the subscribers.

The hub would use the methods PATCH, PUT and DELETE to provide explicit directives of what to do with the content.
Subscribers that can only support GET or POST requests can support the X-HTTP-Method-Override.


### Option 2 : Translation ###
An alternative option is to send the details obtained from the publisher about the updates to the resource, in the body using a feed format. So for example, the subscriber would request to subscribe to a resource such as a user's profile and specify via Accept Headers that it only supports application/stream+json. The hub would then wrap the new value of the resource it got after doing a get inside an entry in the feed adding the verb "update".
How a feed represents each change would be outside the scope of the PubSubHubbub spec and would be described in the appropriate specification for the feed such as Activity Streams Atom Extension or JSON Activity Streams.

The main benefit of this is that its safer because the X-Hub-Signature sent in the headers of the HTTP POST from the Hub to the subscriber only covers the body of the request and there will be n collision of headers or confusing headers sent.

Example using JSON Activity Stream:
{
> "items" : [
> > {
> > > "postedTime" : 121212121,
> > > "verb" : "update",
> > > "object" : {
> > > > "link" : "![http://farm5.static.site.com/4140/4799357039_d29acebf27_b_e.jpg](http://farm5.static.site.com/4140/4799357039_d29acebf27_b_e.jpg)",
> > > > "content" : "base64string representing the image"

> > > }

> > }

> ]
}

## Subscription Flow ##

So overall the subscription flow remains the same. One optional enhancement hubs can provide is this content translation service. In that scenario subscribers would pass the specific Accept header when making the subscription.

The hub can then store the desired Content Type by the subscriber and push changes using that content type even if the original feed had a different content type.

### Feed Content Types ###

  * application/atom+xml
  * application/stream+json
  * application/rss+xml

## Delete Notifications ##

Delete notifications are very drastic and thus we need to make sure they are intentional. After the light ping, the hub would make a request to the publisher which would return a 410 Gone instead of a 404 Not Found. See http://tools.ietf.org/html/rfc2616#section-10.4.11 indicating that the content needs to be purged. It should also include the Date HTTP header of when it got deleted in case the requests come out of order to the subscriber

One privacy consideration of this approach is that it does leak the fact that the content was previously there.

If privacy is a concern the recommended approach is to use Authentication between the publisher and the hub so the hub can "trust" the delete notifications from the publisher and not need to share publicly this information. More on the authorization topic will be added to [Pshb\_OAuth2](Pshb_OAuth2.md)