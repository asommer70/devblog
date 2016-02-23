---
title:  "DVD Pila! Web Sockets"
date:   2016-02-23 13:05:00
categories: rails dvdpila
layout: post
image: rails_sockets.jpg
---

## Useful Software

I've been going down a path of building software that people don't use lately.  To be honest I've been doing that a few more time than feels good to admit.  I know that not everything I build will be used by a lot of people, heck most things I build aren't even used by me, but enough is enough.

I feel it's time to try and only work on things that I'm getting paid for, or people will actually use.

To that end I whipped out the [DVD Pila!](https://github.com/asommer70/dvdpila) code and fixed some bugs and added a web socket to listen for and send events for video playback.

My first experience with web sockets was building a realtime [Note](https://github.com/asommer70/thehoick-notes-server) taking app for a local hackathon.  It was very cool using [Socket.io](http://socket.io/) to update a note using a web interface, Android app, and an iOS app at the same time.

My thought is to build a DVD Pila! Remote app at some point, but the first step is to integrate a web socket server into the existing Rails app.

<!--more-->

## web-socket Rails

There are a few libraries, projects, etc that bring web sockets to Rails, and the one I chose to use for DVD Pila! Is [websocket-rails](http://websocket-rails.github.io/).  The installation and configuration of **websocket-rails** is pretty straight forward.

First, edit the **Gemfile** and add:

```ruby
gem 'websocket-rails'
gem 'faye-websocket', '0.10.0'
```

The **faye-websocket** entry for version **0.10.0** was specifically needed for the JavaScript socket connection on the front end.  Initially a new version of faye-websocket was installed, but I received a bunch of socket timeout errors and found a thread stating to use the **0.10.0** version to get things working.  So there  you have it, but maybe by the time you read this the **websocket-rails** project will be able to use the latest version of **faye-websocket**.

Next, install the new gems:

```
bundle install
```

Run the installation generator:

```
rails g websocket_rails:install
```

The web socket URL will change depending on the Rails environment so setup a new config file **config/socket.yml** and configure an option for each environment:

```
development:
  url: 'localhost:3000/websocket'

test:
  url: 'localhost:3000/websocket'

production:
  url: 'dvdpila:3001/websocket'
```

Then create a new initializer to read the config file, create **config/initializers/socket_url.rb**:

```ruby
SOCKET_CONFIG = Rails.application.config_for(:socket)
```

Finally, edit **config/environments/development.rb** and add:

```ruby
  config.middleware.delete Rack::Lock
```

Restart the Rails development server if you have it running.

## Model Update

I realized that there's not a good place in current DVD Pila! models to store the information about the status of the video that is playing.  On the front-end JavaScript side the information is determined by listening to the HTML5 video element events.  We'll use those to send events to the socket, but before that create a new model to store that info on the backend.

Create the model with:

```
bin/rails g migration create_playing
```

In the **db/migrate/$DATETIMESTAMP_create_playing.rb** file add the following to the **change** method:

```ruby
    create_table :playings do |t|
      t.belongs_to :dvd, index:true
      t.belongs_to :episode, index:true
      t.string :status

      t.timestamps null: false
    end
```

Then perform the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

We can now store the playback status for Dvd and Episode objects.

## Events

A new file for subscribing to socket events was generated in **config/events.rb**.  This file is sort of like **config/routes.rb** in that you define an event to listen to and map that to a method in a controller.   The generate file comes with some great comments, but here is the final events setup for video playback:

```ruby
WebsocketRails::EventMap.describe do
  subscribe :play, :to => PlayingsController, :with_method => :play

  subscribe :pause, :to => PlayingsController, :with_method => :pause

  subscribe :stop, :to => PlayingsController, :with_method => :stop
end
```

This example is very simple, so please refer to the **websocket-rails** documentation for more advanced configurations.  For my purposes this works nicely the event, named right after the **subscribe** method, is listened for, the controller is set via the **:to** key, and the method inside the controller is called with the **:with_method** key.

## Socket Controller

Now that we can listen for events on the socket we need to create the controller that will actually do something after the event has been received.

Create a new file **app/controllers/playings_controller.rb** with:

```ruby
class PlayingsController < WebsocketRails::BaseController
  before_action :set_playing

  def play
      @playing.update(status: 'play')
      send_message :playing_success, @playing
  end

  def pause
    # Need to check for stop status cause a Pause is sent when DVD is page is left and video is playing.
    if (@playing.status != 'stop')
      @playing.update(status: 'pause')
      send_message :pause_success, @playing
    end
  end

  def stop
    @playing.update(status: 'stop')
    send_message :stop_success, @playing
  end

  private
    def set_playing
      @dvd = Dvd.find(message[:id]) if message[:type] == 'dvd'
      @episode = Episode.find(message[:id]) if message[:type] == 'episode'
      if @dvd
        @playing = Playing.where(dvd: @dvd).last
      else
        @playing = Playing.where(episode: @episode).last
      end

      unless @playing
        @playing = Playing.new(dvd: @dvd, status: 'play', episode: @episode)

        if
          @playing.save
          send_message :playing_success, @playing
        else
          send_message :playing_fail, @playing
        end
      end
    end
end
```

First, notice that this controller inherits from **WebsocketRails::BaseController** and not the normal **ActionController** class.  The methods for play, pause, and stop are pretty much the same.  They update the **@playing** Playing object based on the event then send an event to the socket using the **send_message** method.

The more involved method is the **private** **set_playing** method.  I created this after realizing I was repeating a lot of code looking up the Dvd/Episode and the Playing object in each method.  Yay for DRY!

In **set_playing** if the Playing object for the Dvd/Episode isn't found a new one is created.

## Sockets on the Frontend

The server is listening for socket events so let's send some via JavaScript, or rather CoffeeScript, and maybe listen for some socket events as well.

First, create a new file to connect to the socket for the environment in **app/assets/javascripts/socket.js.erb**:

```javascript
//
// Connect to the web socket.
//
window.dispatcher = new WebSocketRails("<%= SOCKET_CONFIG['url'] %>");
```

Edit **app/assets/javascripts/dvds.coffee** and at the top of the file add:

```coffeescript
  #
  # Handle socket playback events.
  #
  window.dispatcher.bind 'playing_success', (playing) ->
    console.log('successfully playing:', playing)

  window.dispatcher.bind 'pause_success', (playing) ->
    console.log('successfully paused:', playing)

  window.dispatcher.bind 'stop_success', (playing) ->
    console.log('successfully stopped:', playing)

  if (window.location.pathname.split('/')[window.location.pathname.split('/').length - 2] == 'dvds')
    # Send stop event to socket on refresh, nav away, close
    $(window).on 'beforeunload', (e) ->
      send_stop()

      return 'Setting video to stop...'

    $(document).on 'page:before-unload', (e) ->
      send_stop()
```

The first line creates a new web socket connection to **/websocket** and adds it to the **window** object.

Next the **window.dispatcher.bind** methods are listening for socket events coming from the server.  Though there's really nothing done with the data being sent back at this time.

The current path is checked and if the page is a DVD page the **$(window).on 'beforeunload'** is listened for and the **send_stop()** function called.  Likewise the [Turbolinks](https://github.com/turbolinks/turbolinks-classic) **page:before-unload** event is listened for because that is fired when users navigate to another page.

Next, add the **send_stop()** function further down the file:

```coffeescript
@send_stop = () ->
  $player = $('.player')
  # console.log('$player:', $player.data().dvd)

  if $player.hasClass('episode')
    url = '/episodes/' + $player.data().episode + '.json'
    window.dispatcher.trigger('stop', {id: $player.data().episode, type: 'episode'})
  else
    url = '/dvds/' + $player.data().dvd + '.json'
    window.dispatcher.trigger('stop', {id: $player.data().dvd, type: 'dvd'})

  if (window.location.pathname.split('/')[window.location.pathname.split('/').length - 2] != 'dvds')
    $(document).off('page:before-unload')
```

This function will find the video element via the **.player** class and determine if it's an Episode or a Dvd.  Then send a socket event via the **window.dispatcher.trigger** method.  The second argument after the event name is an object with the type of video and the id.  These are used in the controller to find the Dvd and the Episode respectively.

With **stop** out of the way, edit the **@play_location** function to send the **play** event:

```coffeescript
@play_location = (video) ->
  video.focus()
  self = video
  $player = $(video)

  videoTime = getVideoTime(video.currentTime)

  if $player.hasClass('episode')
    url = '/episodes/' + $player.data().episode + '.json'
    window.dispatcher.trigger('play', {id: $player.data().episode, type: 'episode'})
  else
    url = '/dvds/' + $player.data().dvd + '.json'
    window.dispatcher.trigger('play', {id: $player.data().dvd, type: 'dvd'})

  $.get(url).then (data) ->
    self.currentTime = data.playback_time
    self.play()
```

And finally, edit the **$('.player').on 'pause'** event function to trigger the **pause** event:

```coffeescript
    # Save playback_time when paused.
    $('.player').on 'pause', (e) ->
      this.focus()
      $player = $(this)
      $player.focus()

      # Do some Maths on the playback time.
      videoTime = getVideoTime(this.currentTime)

      # Determine the put URL.
      if $player.hasClass('episode')
        url = '/episodes/' + $player.data().episode + '.json'
        type = 'episode'
        window.dispatcher.trigger('pause', {id: $player.data().episode, type: type})
      else
        url = '/dvds/' + $player.data().dvd + '.json'
        type = 'dvd'
        window.dispatcher.trigger('pause', {id: $player.data().dvd, type: type})
…
```

Awesome, the Playing object for the Dvd/Episode will now be updated whenever the HTML5 video element is played, pause, or the page is closed or left.

## Production

If you're using Passenger, like me, to serve the production environment change the **stand.alone** config option in **config/initializers/websocket_rails.rb** to **true**:

```
  config.standalone = true
```

And install [Redis](http://redis.io/) if you haven't got it running on your server.

## Conclusion

Of all the apps I've developed I think DVD Pila! is the one I use the most.  Maybe it's because I like movies so much, or maybe the app is actually useful, but I use it at least a couple of times a week.

Maybe once I get [The Hoick Habit App](https://github.com/asommer70/thehoick-habit-app) into the Play Store, App Store, etc I'll use that on a daily basis… maybe.

Party On!
