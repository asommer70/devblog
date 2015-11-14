# Albums 2nd Draft

## Albums The Complete Picture

There are some additional features needed for the Album object.  When outlining the features of the app creating Albums seemed like a very small step, but after the last post it turned out to be more complicated than I originally planned.  Not to mention much more time consuming.

Most of the delays had to do with getting a good auto-complete library setup.  I did try out a couple of others, but ended up going back with Chosen.  Which coincedently is the library I used in the [BCN](https://github.com/asommer70/bcn) project.  

Another sizable delay was getting the model setup correctly to “reference” back to the Audio object.  Creating the table is quite simple, but then you also have to create the “relationship” table that has ID numbers for Audios and their **belong_to** Album.

Once that was rocking things went much more smoothly…

## Album Covers

The first new feature for our Albums is going to be a cover image.  To make uploading image files easy we’ll use the [Dragonfly](http://markevans.github.io/dragonfly/) library and via gem.  This is a great library and makes it super easy to upload files and link to them in your view templaes.

### Installing Dragonfly

Edit the **Gemfile** and add:

```
gem 'dragonfly', '~> 1.0.12'
```

And bundle it:

```
bundle install
```

Next, execute:

```
rails generate dragonfly
```

You’ll notice that this creates the Dragonfly initializer in **config/initializers/dragonfly.rb**.  At this point Dragonfly is installed and we can now add it to our models.

### Adding Dragonfly to Albums

To add Dragonfly to a model we just add an accessor statement.  Edit **app/models/album.rb** and add the following line toward the top:

```
dragonfly_accessor :image
```

Next, the Album model needs to be updated with the new image fields.  Using the **rails** command create a new migration:

```
bin/rails g migration add_image_to_album
```

And in the **db/migrate/DATESTAMP_add_image_to_album.rb** file add the following to the **change** method:

```
    add_column :albums, :image_uid, :string
    add_column :albums, :image_uid, :string
    add_column :albums, :image_name, :string
```

Now apply the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

### Updating the Album Form

Finally, adjust the **app/views/albums/_form.html.erb** file and add the following file input element:

```
  <div class="row">
    <div class="columns large-4">
      <%= f.label :image, 'Upload a cover image?'  %>
      <%= f.file_field :image %>
    </div>
  </div>

  <br/>
```

Also, don’t forget to update the **app/controllers/albums_controller.rb** file and add the **:image** filed to the permitted parameters in the **album_params** method:

```
      params[:album].permit(:name, :artist, :year, :genre, :image, :audio_ids => [])
```

### Displaying the Album Cover

To see what we’re working with as far as Album covers go edit the **app/views/albums/show.html.erb** file and add:

```
        <div class="row">
          <div class="columns large-6 text-center panel">
            <strong><%= audio.name %></strong>
            <br/>
            <%= image_tag @album.image.thumb('300x300').url if @album.image %>
            <audio id="<%= audio.id %>" controls data-audio="<%= audio.id %>" class="player">
              <% if audio.path.index('http') %>
                <source src="<%= audio.path %>"></source>
              <% else %>
                <source src="<%= stream_path(audio.id) %>"></source>
              <% end %>
            </audio>
          </div>
        </div>
```

Okay, this does a little more than just add an image tag to the page.  There’s a new row div and we’re wrapping the player Audio name and Album image in a Foundation panel div.  I think the affect looks pretty good.

The Audio show page should also have the Album image if it’s available. Edit **app/views/audios/show.html.erb** and change the player element to have:

```
<div class="row">
  <div class="columns large-6 text-center panel">
    <%= image_tag @audio.album.image.thumb('300x300').url if @audio.album && @audio.album.image %>
    <audio id="<%= @audio.id %>" controls data-audio="<%= @audio.id %>" class="player">
      <% if @audio.path.index('http') %>
        <source src="<%= @audio.path %>"></source>
      <% else %>
        <source src="<%= stream_path(@audio.id) %>"></source>
      <% end %>
    </audio>
  </div>
</div>
```

Now things are looking pretty good.

## Play Album

Playing an Album is great, but it’s even greater to know which file to play and which file is next.  It will be cool to save this data to the database so that we can pick up where we left off if we’d like to.  Also, being able to set the order of Audios is great in case one files are created in an order other than intended.  

### More Albums Table Updates

Create a new migration to add a **current_audio** field:

```
bin/rails g migration add_current_audio_order_to_album
```

Add the **add_column** method to the **db/migrate/DATESTAMP_add_current_audio_to_album.rb** file:

```
    add_column :albums, :current_audio, :integer, index: true
    add_column :audios, :album_order, :integer
```

And execute the migration task:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

## Album Show For Playback

We’ll need some controls to play each Audio in an Album so edit **app/views/albums/show.html.erb** adding underneath the Album details ul element:

```
<div class="row">
  <div class="columns large-4">
    <h4>Play Album</h4>
    <div class="controls">
      <button class="warning tiny control change" data-function="-1" data-album="<%= @album.id %>"><i class="fi-previous"></i></button>
      <button class="warning tiny control" data-function="play" data-album="<%= @album.id %>"><i class="fi-play"></i></button>
      <button class="warning tiny control" data-function="pause" data-album="<%= @album.id %>"><i class="fi-pause"></i></button>
      <button class="warning tiny control change" data-function="1" data-album="<%= @album.id %>"><i class="fi-next"></i></button>
    </div>
    <div class="current-song">
      <h5>Current Audio</h5>
      <span id="playing_audio">
        <% if @last_audio %>
          <strong><%= @last_audio.name %></strong> was last played.
        <% else %>
          Don't know what was last played...
        <% end %>
      </span>
    </div>
  </div>
</div>
```

We’ll use some JavaScript (well CoffeeScript really) to make the controls work, but first we need to be able to query the Audios in the Album using JSON.  Edit **app/views/albums/show.json.jbuilder** and add audios:

```
json.extract! @album, :id, :created_at, :updated_at, :audios
```

### Albums CoffeeScript (Where the Magic Happens)

Now for some JavaScripts, edit the **app/assets/javascripts/album.coffee** and add the following to the **ready_albums** function:

```
  #
  # Handle Album playback controls.
  #
  $('.control').on 'click', (e) ->
    $this = $(this)
    if $this.hasClass('change')
      media_control.change_audio($this.data().album, parseInt($this.data().function))
    else
      media_control[$this.data().function]($this.data().album)

@media_control = {
  play: (id) ->
    $.get("/albums/#{id}.json")
      .then (album) ->
        if album.current_audio != null
          if !window.current_audio?
            window.current_audio = media_control.find_current_aduio(album)
        else
          window.current_audio = {
            audio: album.audios[0],
            current_player: $('#' + album.audios[0].id)[0],
          }

        window.current_audio.current_player.play()
        media_control.update_album(album, window.current_audio)


  pause: (id) ->
    if window.current_audio?
      window.current_audio.current_player.pause()

  change_audio: (id, direction) ->
    $.get("/albums/#{id}.json")
      .then (album) ->
        if window.current_audio?
          window.current_audio.current_player.pause()


          # Find the index of the next/previous Audio
          for audio, idx in album.audios
            if window.current_audio.audio.id == audio.id
              current = idx + direction
              if current == album.audios.length
                current = 0
              else if current == -1
                current = album.audios.length - 1
              window.current_audio = {
                audio: album.audios[current],
                current_player: $('#' + album.audios[current].id)[0],
              }
              break
        else if album.current_audio?
          # Album isn't playing so play the last audio.
          window.current_audio = media_control.find_current_aduio(album)
        else
          # Album doesn't have a current_audio so start the first, or the last, audio.
          if direction == -1
            current = album.audios.length - 1
          else
            current = 0

          window.current_audio = {
            audio: album.audios[current],
            current_player: $('#' + album.audios[current].id)[0],
          }


        window.current_audio.current_player.play()
        media_control.update_album(album, window.current_audio)

  find_current_aduio: (album) ->
    for audio, idx in album.audios
      if audio.id == album.current_audio
        audio = audio
        current_player = $('#' + audio.id)[0]

        if idx + 1 == album.audios.length
          next = 0
          previous = album.audios.length - 1
        else
          next = idx + 1
          previous = idx

        return {
          audio: audio,
          current_player: $('#' + audio.id)[0],
        }

  update_album: (album, current_audio) ->
    $('#playing_audio').html(current_audio.audio.name + ' <em>' + current_audio.audio.album_order + '</em>')
    $.ajax({
      url: '/albums/' + album.id + '.json',
      method: 'put',
      data: 'album[current_audio]=' + current_audio.audio.id
    })
}
```

So the broad strokes of the playback functionality happens **@media_control** object.  The **media_control.play** method obviously plays the desired Audio file using it’s audio tag element.  Then it updates the **playing_audio** span with the name of the Audio file.

Whenever an Album is paused, or changed, the Album is updated via Ajax in the **media_control.update_album** method as well as the playback position of the Audio, but that happens via the **audios.coffee** functions.

The Audio that is currently playing as well as it’s audio (player) element is saved into the **media_control.current_audio** object.  This is then used to determine which Audio should be played **next**, or **previous**, which enables the next and previous buttons.

There’s probably some holes in this, but it works for now.

### Keep Things Under Control

The **app/controllers/audios_controller.rb** needs to be updated to give the **current_audio** field permission:

```
      params[:audio].permit(:name, :path, :playback_time, :album_id, :current_audio)
```

Also, we want to display the name of the last played Audio so in the **show** method add a variable for it:

```
    @last_audio = Audio.find(@album.current_audio) if @album.current_audio
```

