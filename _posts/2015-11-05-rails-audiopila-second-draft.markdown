---
title:  "Audio Pila! Second Draft"
date:   2015-11-05 13:05:00
categories: rails audiopila
layout: post
image: 2nddraft_cover.jpg
---

## Improvements Along The Way

There are quite a few improvements we can make in our Audio Pila! Rails app.  To name a few (but let’s not get carried away just yet):

* Make the Audio index page display multiple repositories better.
* Be able to create Albums form collections of Audios.
* If we can make Albums, we’ll need a way to upload images to display with the Album.

Let's blast into it...

<!--more-->

## Fixing Up The Index

First off, edit the **app/views/audios/index.html.erb** file adjusting the **.audio-list** ul as so:

```
  <ul class="audio-list">
    <% @audios.each do |audio| %>
      <% if audio.path.index(repo) %>
        <li>
          <%= link_to audio.name, audio_path(audio) %>
        </li>
      <% end %>
    <% end %>
  </ul>
```

There now the audio files for each repository will be displayed under their respective path.

One thing we’re missing is displaying Audios that are manually added to the system.  We’ll get making it easier to add an Audio later on, but for now at the bottom of the file add:

```
<br/><hr/><br/>

<h3>Network Audios</h3>

<% @manuals.each do |audio| %>
  <ul class="audio-list">
    <li>
      <%= link_to audio.name, audio_path(audio) %>
    </li>
  </ul>
<% end %>
```

And in the **app/controllers/audios_controller.rb** add another query to the **index** method:

```
  def index
    @audios = Audio.all
    @repositories = Settings.repositories
    @manuals = Audio.where("path ~ ?", 'http')
  end
```

Don’t forget you can start the Rails development server with:

```
bin/rails s
```

## Removing Old Audios

If a file is removed form a repository directory we should also remove it’s entry from the database.  Adjust the **sync_repo** method in **apps/controllers/audios_controller.rb** to add a repo loop:

```
  def sync_repo
    repo = Settings.repositories[params[:id].to_i]

    files = `find #{repo} -name "*.mp3" -o -name "*.m4a" -o -name "*.ogg"`
    files = files.split("\n")

    files.each do |file|
      file_name = file.split('/')[-1]

      unless Audio.find_by_name(file_name)
        Audio.create(name: file_name, path: file)
      end
    end

    # Remove Audio if no longer in repo.
    Audio.where("path ~ ?", repo).each do |audio|
      unless File.exists?(audio.path)
        audio.destroy
      end
    end

    flash[:info] = 'Repository sync successful.'
    redirect_to audios_path
  end
```

Now if a file doesn’t exist in the directory it’ll be removed from the database.  Very tidy…

## Navigation

After using the site for a few minutes it quickly becomes apparent a navigation, or a simple list of links, for the app would be very useful.  Good thing is that the Rails tempting system makes this very easy to include on each page, and the Foundation framework includes some handy examples of navigation elements.

I used a simple subnav for a past post and I think that’ll work well again.  At least until the app expands in features and we need a better nav system.

First, create a new partial template in **app/views/layouts/_nav.html.erb**:

```
<br/><br/>

<dl class="sub-nav">
  <dd class="<%= 'active' if params[:controller] == 'audios' %>"><a href="/">Audio Files</a></dd>
  <dd class="<%= 'active' if params[:controller] == 'settings' %>"><a href="/settings">Settings</a></dd>
</dl>

<hr/>
```

So the simple description list element is used by Foundation as a [Sub Nav](http://foundation.zurb.com/docs/components/subnav.html) and the **active** class is set based on the current controller (set by the request object).

Next, render the nav partial by editing **app/views/layouts/application.html.erb** and add the following just above the **yield** statement:

```
        <%= render 'layouts/nav' %>
```

Ah, that’s better.  Now we know where we are and where we can go.

## Creating Audios

The next item to clean up is to adjust the Audios **_form** partial to a little more user friendly and to allow Audio Pila! To stream files hosted on another server via HTTP.  First edit the **app/views/audios/_form.html.erb** file and add some placeholder attributes to the inputs:

```
  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :name, placeholder: 'Name' %>
    </div>
  </div>

  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :path, placeholder: 'Path' %>
    </div>
  </div>
```

To stream a file from another server we need to check the Audio path attribute for the string ‘http’, or at least that’s an easy, though non-bullet proof, way to do it.  First, add the check to the **app/views/controllers/audios_controller.rb** file in the **stream** method:

```
  def stream
    audio = Audio.find(params[:id])

    unless audio.path.index('http')
      response.headers['Content-Type'] = "audio/#{audio.name[-3..-1]}"
      response.headers['Content-Length'] = File.size(audio.path).to_s
      response.headers['Accept-Ranges'] = 'bytes'
      response.headers['Cache-Control'] = 'no-cache'

      send_file audio.path
    end
  end
```

After checking for the ‘http’ string in the path a bunch of additional headers are set in order for the browser to play/pause the file appropriately.  I didn’t get into why those particular headers were needed, but I followed header example [here](http://stackoverflow.com/questions/9629223/audio-duration-returns-infinity-on-safari-when-mp3-is-served-from-php) and tried various options until I was able to play the audio, pause the audio, and play it again without it stopping.  

These options worked for me in Google Chrome, but your experience might be different depending on what browser you’re using.

Second, add a similar check to the **app/views/audios/show.html.erb** in the source element of the audio tag:

```
  <% if @audio.path.index('http') %>
    <source src="<%= @audio.path %>"></source>
  <% else %>
    <source src="<%= stream_path(@audio.id) %>"></source>
  <% end %>
```

Cool beans, should now be able to stream a file hosted anywhere on the Internet.

## Playback Time!

A cool thing, at least I think it’s cool, I did with [DVD Pila!](http://dvdpila.thehoick.com/) was to add keyboard and mouse controls to the HTML5 video element.  This allows you to control play/pause with the spacebar, clicking the video element, etc. The coolest thing though is saving the playback duration spot to the server when the video is stopped.  That way you can pick up right where you left off.

I think the same, or similar, features would be awesome for Audio Pila!.  To do that create a new CoffeeScript file in **app/assets/javascripts/audios.coffee** with:

```
ready_audio = ->
  #
  # Handle Player actions.
  #
  if $('.player').length > 0
    #
    # Disable spacebar paging.
    #
    if $('audio').length != 0
      $(window).on 'keydown', (e) ->
        return !(e.keyCode == 32);

    # Save playback_time when paused.
    $('.player').on 'pause', (e) ->
      this.focus()
      $player = $(this)
      $player.focus()

      # Do some Maths on the playback time.
      playbackTime = getPlaybackTime(this.currentTime)

      # Do the putting.
      $.ajax({
        url: '/audios/' + $player.data().audio  + '.json',
        method: 'put',
        data: 'audio[playback_time]=' + playbackTime
      })

    # Start playing at the playback_time.
    .on 'play', (e) ->
      e.preventDefault()
      e.stopPropagation()
      play_location(this)

    # Play and pause on space bar.
    .on 'keyup', (e) ->
      this.focus()
      e.preventDefault()
      e.stopPropagation()
      player = $('.player')[0]

      if (e.keyCode == 32 && player.paused == true)
        player.play()
      else if (e.keyCode == 32 && player.paused == false)
        player.pause()

    # Scroll playback time forward and backward with the Arrow keys.
    .on 'keydown', (e) ->
      player = $('.player')[0]

      if (e.keyCode == 39)
        player.currentTime += 1
      else if (e.keyCode == 37)
        player.currentTime -= 1

    # Seek the audio on scroll.
    .on 'wheel', (e) ->
      e.preventDefault()
      e.stopPropagation()
      player = $(this)[0]
      $player = $(player)
      $player.focus()

      if e.originalEvent.wheelDelta > 0
        player.currentTime += 1
      else
        player.currentTime -= 1

    .on 'ended', (e) ->
      $player = $(this)

      $.ajax({
        url: '/audios/' + $player.data().audio  + '.json',
        method: 'put',
        data: 'audio[playback_time]=' + 0
      })



getPlaybackTime = (time) ->
  hours = Math.floor(time / 3600)
  minutes = Math.floor(time / 60)
  if (minutes > 59)
    minutes = Math.floor(time / 60) - 60

  seconds = Math.round(time - minutes * 60)
  if (seconds > 3599)
    seconds = Math.round(time - minutes * 60) - 3600

  return time


play_location = (audio) ->
  audio.focus()
  self = audio
  $player = $(audio)

  playbackTime = getPlaybackTime(audio.currentTime)

  $.get('/audios/' + $player.data().audio + '.json').then (data) ->
    self.currentTime = data.playback_time;
    self.play()


# Fire the ready function on load and refresh.
$(document).ready(ready_audio)
$(document).on('page:load', ready_audio)
```

These JavaScript/CoffeeScript functions were taken largely, well practically totally, from the DVD Pila! [dvds.coffee](https://github.com/asommer70/dvdpila/blob/master/app/assets/javascripts/dvds.coffee).  The base of the functionality comes from the audio element events.  The Mozilla Developer Network has a [great](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_HTML5_audio_and_video) write up on the HTML5 media elements.

## Adding Playback Time Field

Now that we have the ability to send the **currentTIme** of the audio element we need a place to store it in our Audio model.  Create a new migration to add the **playback_time** field:

```
bin/rails generate migration AddPlabackTimeToAudios playback_time:integer
```

The resulting migration should have this line in the **change** method:

```
    add_column :audios, :playback_time, :integer
```

Apply the migration with:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

## Audio JSON

One more thing we’ll need is a JSON representation of an Audio object.  This will allow us to query the server via AJAX and start playback from the saved time.  Add the additional fields to the **app/views/audios/show.json.jbuilder** file:

```
json.extract! @audio, :id, :name, :path, :playback_time, :created_at, :updated_at
```

You can see the resulting JSON if you browse to *http://localhost:3000/audios/1.json* for example.

## Permit Playback

Lastly, edit the **app/controllers/audios_controller.rb** file and add the **playback_time** field to the permitted attributes in the **audio_params** private method:

```
      params[:audio].permit(:name, :path, :playback_time)
```

## Tests

I’m sure there’s totally some test we could write to make sure things are working the way we’d like.  Testing the audio playback functionality seems pretty complicated, so we’ll test the things that could really run amuck with our app (I.e. The easy stuff).

After attempting to work with the existing Test::Unit tests my frustration level grew to where I’m ready to toss in the towel and migrate those bad boys to RSpec.

Stay tuned for Rspec tests…

## Conclusion

These improvements really make the app more useful.  At this stage the “killer” feature for me is the ability to save playback time and pick up where I left off.

I find it very annoying to be listening to a podcast either on an iOS or Android app and have to scroll through to find where I left off.  With Audio Pila! I know it will be saved and I can listen to different shows depending on what I’m into at the moment.

Also, I setup a Github repo for Rails [Audio Pila!](https://github.com/asommer70/audiopila-rails) project if you’d like to follow along in code.

Party On!
