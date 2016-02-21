# DVD Pila! Web Sockets

## Useful Software

I've been going down a path of building software that people don't use lately.  To be honest I've been doing that a few more time than feels good to admit.  I know that not everything I build will be used by a lot of people, heck most things I build aren't even used by me, but enough is enough.

I feel it's time to try and only work on things that I'm getting paid for, or people will actually use.

To that end I whipped out the [DVD Pila!](https://github.com/asommer70/dvdpila) code and fixed some bugs and added a web socket to listen for and send events for video playback.

My first experience with web sockets was building a realtime [Note](https://github.com/asommer70/thehoick-notes-server) taking app for a local hackathon.  It was very cool using [Socket.io](http://socket.io/) to update a note using a web interface, Android app, and an iOS app at the same time.

My thought is to build a DVD Pila! Remote app at some point, but the first step is to integrate a web socket server into the existing Rails app.

## web-socket Rails

Edit the **Gemfile** and add:

```
gem 'websocket-rails'
gem 'faye-websocket', '0.10.0'
```

The **faye-websocket** entry for version **0.10.0** was specifically needed for the JavaScript socket connection on the front end.  Initially a new version of faye-websocket was installed, but I received a bunch of socket timeout errors and found a thread f