# Introduction #

At f8, Facebook launched a first version of real time updates for the Graph API. These updates allow consumers to subscribe to users of their application and get notified via a HTTP Post when the data has changed so they can go and fetch it by invoking the Graph API endpoint.
Some in the community were wondering why the PubSubHubbub protocol was not used; it comes down to these three reasons:

# Details #
1. The need for simple data modeling of any resource:
  * PubSubHubbub currently only supports Atom and RSS and we wanted to use JSON to match the rest of the Graph API so developers don't have to write additional wrappers.
  * We need to syndicate changes to any type of resource not just feeds or lists. The changes may include updating properties of a given resource or deleting the resource altogether. PubSubHubbub only supports appending to the list.
2. The need for user authorization
  * We needed to let users remain in control over what data is shared and with whom, based on their privacy settings.
  * PubSubHubbub v 0.3 does not address authenticating the publishers or the subscribers.

  * As you may have seen, the Graph API is extremely powerful in its simplicity, flexibility and efficiency.
  * To query the Graph API you just need to figure out the url of the resource you are interested in fetching and use OAuth 2.0

  * Example: https://graph.facebook.com/ciberch/feed?token=XXXX and we wanted to use a similar elegant approach for subscribing to notifications
3. The need for more granular synchronization commands.
  * Since we are modeling objects and connections we need to be able to instruct subscribers when the object is deleted or updated and needing replacement.
  * We also want to have the option to do light pings as some of our consumers may want to batch changes to fetch together.


While some of these details are Facebook specific, others can benefit a wider audience, in particular: service providers with users and APIs. With that in mind, here is our wish list:


## PubSubHubbub Wishlist ##
1. Support the use of OAuth 2.0 for authenticating the subscriber and publisher.

2. Move discovery of the hub to the HTTP layer by making it an HTTP Header
  * Leaves backwards compatibility for existing discovery mechanism.

3. Support data in any arbitrary format including JSON

4. Support more synchronization changes than those which can be supported with Atom and POST
  * Includes PUT for replacing a resource and DELETE for asking that the resource gets deleted - Also includes the ability to specify different end points which receive DELETE notifications and PUT notifications

## Proposed Initial Spec Changes ##
http://github.com/ciberch/pshb_oauth


For more details and diagrams see: http://montrics.blogspot.com/2010/05/facebooks-realtime-updates-use-cases.html