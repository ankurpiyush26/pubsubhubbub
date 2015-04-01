**This section by Mariano Guerra**

# Getting Started Guide for Developers #

## screencast ##

If you just want to watch someone else go through this demo, watch the screencast!

<a href='http://www.youtube.com/watch?feature=player_embedded&v=5obK-l8JwSY' target='_blank'><img src='http://img.youtube.com/vi/5obK-l8JwSY/0.jpg' width='425' height=344 /></a>

## naming conventions ##
  * pshb/PSHB = pubsubhubbub
  * gae/GAE = Google App Engine
  * things that start with "$" are bash commands
  * things that start with "!" are important notes

## before you start ##

to follow this guide you will need some tools installed on your system

  * python 2.5 (to run the google app engine hub)
  * subversion client (to fetch the pubsubhubbub code)
  * git client (to fetch the example code)

this can be installed on a debian based distribution with a command like

```
sudo aptitude install python2.5 subversion git-core
```

on other distributions the command should be similar depending on the package manager.

## create directory to hold all the content of this tutorial ##

```
$ mkdir pshb
$ cd pshb
```

## download Google App Engine SDK ##

**!** pshb trunk needs the latest version of GAE (1.3.5 as of today) to work, but
**!** the download page display an older version

```
$ wget http://googleappengine.googlecode.com/files/google_appengine_1.3.5.zip
$ unzip google_appengine_1.3.5.zip
```

**!** GAE needs python 2.5 in order to work, if you have an older or newer version of python install python 2.5 (OS and distro dependent, wont be covered here)

## download the latest version of pshb ##

```
$ svn checkout http://pubsubhubbub.googlecode.com/svn/trunk/ pubsubhubbub
```

## start pshb ##

```
$ python2.5 google_appengine/dev_appserver.py pubsubhubbub/hub/
```

**!** note the python2.5 command

## check that the hub started ##

type http://localhost:8080 on your browser, you should see something like:

_Welcome to the demo PubSubHubbub reference Hub server!_

![http://3.bp.blogspot.com/_XkKIWh0VZYk/SqrdHFQ2i6I/AAAAAAAAGxs/kPxuBPwJ6Nc/s640/pshb0.png](http://3.bp.blogspot.com/_XkKIWh0VZYk/SqrdHFQ2i6I/AAAAAAAAGxs/kPxuBPwJ6Nc/s640/pshb0.png)

## download the example ##

```
# we will need to fetch the tubes library and the example that we will be using.
$ git clone git://github.com/marianoguerra/tubes.git

# now we will start the example

$ cd tubes/ihasfriendz/
$ python main.py
```

we should see something like:

```
 * Running on http://0.0.0.0:8081/
 * Restarting with reloader...
```


### components used by the example ###

  * werkzeug: http://werkzeug.pocoo.org
  * jquery: http://jquery.com
  * tubes: http://github.com/marianoguerra/tubes/tree/master
  * pubsubhubbub\_publish.py: from pshb
  * feedformatter.py: http://code.google.com/p/feedformatter/

## publish the feed to the hub ##

go to the following URL on your browser: http://localhost:8080/publish

on the _Topic_ field enter: http://localhost:8081/atom/stream/_MYUSER_ and click publish

![http://4.bp.blogspot.com/_XkKIWh0VZYk/SqrdHbMyTDI/AAAAAAAAGx0/HU3XaAhjP_w/s640/pshb1.png](http://4.bp.blogspot.com/_XkKIWh0VZYk/SqrdHbMyTDI/AAAAAAAAGx0/HU3XaAhjP_w/s640/pshb1.png)

**!** if everything goes OK, then you wont notice anything on the page, that's ok, browsers act that way to 204 responses

**!** _MYUSER_ is a placeholder for the user you will use to post notices on the test app later (for example http://localhost:8081/atom/stream/marianoguerra)

## subscribing to the hub ##

go to the following URL on your browser: http://localhost:8080/subscribe

on the _Callback_ field enter: http://localhost:8081/callback
on the _Topic_ field enter: http://localhost:8081/atom/stream/_MYUSER_
on the _Verify token_ field enter something random like: _iwantmahcookie_

![http://4.bp.blogspot.com/_XkKIWh0VZYk/SqrdHx3cWDI/AAAAAAAAGx8/V4UrVzmx1Tc/s640/pshb2.png](http://4.bp.blogspot.com/_XkKIWh0VZYk/SqrdHx3cWDI/AAAAAAAAGx8/V4UrVzmx1Tc/s640/pshb2.png)

**!** _MYUSER_ is a placeholder for the user you will use to post notices on the test app later (for example http://localhost:8081/atom/stream/marianoguerra)

click _Do it_

**!** if everything goes OK, then you wont notice anything on the page, that's OK, browsers act that way to 204 responses

## create some content ##

go to the following URL on your browser: http://localhost:8081/files/index.html

post some content on the form, use the user you used as _MYUSER_

![http://3.bp.blogspot.com/_XkKIWh0VZYk/SqrdIRtBMoI/AAAAAAAAGyE/hOtiESEaaHY/s640/pshb3.png](http://3.bp.blogspot.com/_XkKIWh0VZYk/SqrdIRtBMoI/AAAAAAAAGyE/hOtiESEaaHY/s640/pshb3.png)

you can check that the item was posted going manually to http://localhost:8081/atom/stream/_MYUSER_, you should see an atom feed there

![http://1.bp.blogspot.com/_XkKIWh0VZYk/SqrdJC1gwRI/AAAAAAAAGyM/vKPiADfZpLI/s640/pshb4.png](http://1.bp.blogspot.com/_XkKIWh0VZYk/SqrdJC1gwRI/AAAAAAAAGyM/vKPiADfZpLI/s640/pshb4.png)

## manually processing the tasks ##

when running the hub on the dev server we have to run the task queues by hand, to do that go to http://localhost:8080/_ah/admin/queues

![http://2.bp.blogspot.com/_XkKIWh0VZYk/Sqrd_5aSiyI/AAAAAAAAGyU/pjEz8TOZXWc/s640/pshb5.png](http://2.bp.blogspot.com/_XkKIWh0VZYk/Sqrd_5aSiyI/AAAAAAAAGyU/pjEz8TOZXWc/s640/pshb5.png)

on the _Tasks in Queue_ column of the _feed-pulls_ you should see a number different than 0 (that is the number of messages you created since the last execution of that task).

click on the _feed-pulls_ link, there click on the _run_ button.

![http://3.bp.blogspot.com/_XkKIWh0VZYk/SqreATS1_OI/AAAAAAAAGyc/loOLwdvXcSk/s640/pshb6.png](http://3.bp.blogspot.com/_XkKIWh0VZYk/SqreATS1_OI/AAAAAAAAGyc/loOLwdvXcSk/s640/pshb6.png)

when we run the feed pulls task, we tell pshb to fetch the feeds that have new content (the ones that did a post to the hub to inform that there is new content)

**!**  on production hubs this tasks are done automatically

now we go again to the _Task Queues_ page, there the _event-delivery_ queue should have a number different than 0 (the number of messages that are pending to be sent to the subscribers), we click on the _event-delivery_ and then we click on the _run_ button.

![http://1.bp.blogspot.com/_XkKIWh0VZYk/SqreA9bViFI/AAAAAAAAGyk/TOOj_6tdoJ8/s640/pshb7.png](http://1.bp.blogspot.com/_XkKIWh0VZYk/SqreA9bViFI/AAAAAAAAGyk/TOOj_6tdoJ8/s640/pshb7.png)

when we run the event delivery task, we tell pshb to do a POST on every callback url registered for the feeds that were fetched on the _feed-pulls_ task.

## seeing it work ##

now that we created a feed, informed the hub that we had new content, the hub fetched the content and sent it to the callback, we want to see this content, for this go with your browser to http://localhost:8081/new-notices/, you will see the notices that pshb posted back to you the last time.

![http://1.bp.blogspot.com/_XkKIWh0VZYk/SqreBWaXNCI/AAAAAAAAGys/wxGClI7Q2S0/s640/pshb8.png](http://1.bp.blogspot.com/_XkKIWh0VZYk/SqreBWaXNCI/AAAAAAAAGys/wxGClI7Q2S0/s640/pshb8.png)

**!** if you refresh the page you will notice that the messages aren't there anymore, that's because the example stores the new messages in a Queue that is flushed when the request for new notices is made, in this way you can see only the new messages.

**!** the example stores all the information on global variables on main.py (this is to make the example simpler), so every time you change something on main.py and save the server will reload the changes and all the data will disappear.

## for lazy people ##

```
#!/usr/bin/env sh

# create the example directory
mkdir pshb
cd pshb

CWD=$(pwd)
EXAMPLE=$CWD/tubes/ihasfriendz

wget http://googleappengine.googlecode.com/files/google_appengine_1.3.5.zip
unzip google_appengine_1.3.5.zip

svn checkout http://pubsubhubbub.googlecode.com/svn/trunk/ pubsubhubbub


# we will need to fetch the tubes library and the example that we will be using.
git clone git://github.com/marianoguerra/tubes.git

# now we will start the example

echo "run \"cd $EXAMPLE; python main.py\" on a shell to run the example"
echo "run \"cd $PWD; python2.5 google_appengine/dev_appserver.py pubsubhubbub/hub/\" on a shell to run the hub"
```