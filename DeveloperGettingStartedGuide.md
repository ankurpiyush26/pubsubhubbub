**Also see:**
  * [Joseph Smarr's subscriber how-to](http://josephsmarr.com/2010/03/01/implementing-pubsubhubbub-subscriber-support-a-step-by-step-guide/)
  * [Superfeedr's PubSubHubbub subscriber HOW TO](http://blog.superfeedr.com/howto-pubsubhubbub/)
  * [Nick Johnson's how-to (App Engine + XMPP)](http://blog.notdot.net/2010/02/Consuming-RSS-feeds-with-PubSubHubbub)


---


# PubSubHubbub Developer Tutorial #


Note: the project and this document are hosted here: https://github.com/marianoguerra/pshb-example improvements and corrections are welcome!

welcome, this document contains an example application written in **python** that will help you play with a pshb (pubsubhubbub from now on) hub and an application that publishes its content to it.

this guide explains how to install a pubsubhubbub server in your computer so you can play with it, normally on a web application you would use a pshb compatible server like:

  * official pshb hub on app engine: http://pubsubhubbub.appspot.com/
  * superfeedr hub: https://superfeedr.com/


## Requirements ##

  * python >= 2.5 (I'm using 2.6.1)
  * git (for the example)
  * subversion (to download pshb code)
  * bash or similar shell to run the scripts

## Installing ##

```

	git clone https://github.com/marianoguerra/pshb-example.git
	cd pshb-example
	bash setup.sh
```

the first command will fetch the project from github, the third one will get some
libraries needed for the example to run.

note: answer "y" to sammy and "n" to all the other questions that the script
does (we don't need those libraries)

## Running ##

now we will start the example application called pleinu twice (so we can test the communication using the hub) and we will start a local pshb hub.

open 3 terminals and run one comment on each one:

```
google_appengine/dev_appserver.py src/ -p 8000 --datastore_path=/tmp/tubes1
google_appengine/dev_appserver.py src/ -p 8001 --datastore_path=/tmp/tubes2
google_appengine/dev_appserver.py pubsubhubbub/hub/
```

this commands asume that you are at the root of the pshb-example folder.

## Playing ##

open a browser tabs pointing to:

> http://localhost:8000/

click the signup link and create a new user.


![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-1.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-1.png)


I will create one called spongebob, you will have to change the username whenever you see it.

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-2.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-2.png)


after the signup process click the login button and enter the user and password you just entered.


![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-3.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-3.png)


create a message and click send, the message should appear below.


![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-4.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-4.png)

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-5.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-5.png)


now go to http://localhost:8000/atom/messages/from/spongebob/


![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-6.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-6.png)


you should see an atom feed with the message you just created.

## Publishing ##

now that we have a page that generates information we need to publish it on the hub.

open a tab in your browser pointing to http://localhost:8080/ and click on the publish link near the bottom.


![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-7.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-7.png)

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-8.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-8.png)


if you get an error remove the s from the https protocol in the address bar and refresh.


![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-9.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-9.png)


on the Topic field enter the url to the atom feed we saw before: http://localhost:8000/atom/messages/from/spongebob/

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-10.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-10.png)


and click Publish, the page wont change, that's ok.

## Subscribing ##

now we need to subscribe one user from the other site (http://localhost:8001/) to the messages sent by our user.

go to http://localhost:8001/ and create another user, I will call it patrick

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-11.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-11.png)


in the main page of the hub (http://localhost:8080/) click on the subscribe like near the bottom

enter http://localhost:8001/p/notify/patrick/ on the Callback field (change patrick for your username if you used another one) enter http://localhost:8000/atom/messages/from/spongebob/ on the Topic field (change spongebob for your username if you used another one)

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-12.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-12.png)

click the "Do it" button, the page won't change, that's ok.

## Sending a message ##

Go to http://localhost:8000/ (login if you closed it) and send a message.

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-13.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-13.png)

> Now go to http://localhost:8001/ and refresh the page, you should see the messages published by the user in the other site.

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-16.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-16.png)


### Note ###


to make it work and avoid an exception I had to add a return statement at the beginning of the log\_message function at google\_appengine/google/appengine/tools/dev\_appserver.py

if you get that exception like this:

```
in log_request
    self.requestline, str(code), str(size))
  File "/home/asd/pubsubhubbub/pshb/google_appengine/google/appengine/
tools/dev_appserver.py", line 3314, in log_message
    if self.channel_poll_path_re.match(self.path):
AttributeError: DevAppServerRequestHandler instance has no attribute
'path'
```


![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-15.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-15.png)


edit the function to look like this:

```
def log_message(self, format, *args):
  """Redirect log messages through the logging module."""
  return
  if self.channel_poll_path_re.match(self.path):
    logging.debug(format, *args)
  else:
    logging.info(format, *args)
```



you will have to set write permissions to the file to save it (chmod u+w dev\_appserver.py)

![https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-14.png](https://github.com/marianoguerra/pshb-example/raw/master/doc/img/pshb-14.png)

you will have to restart the pshb server:


google\_appengine/dev\_appserver.py pubsubhubbub/hub/