# Playlists 2nd Draft

## Playing Playlists and Additional Features

Right, we’ve already whipped up the JavaScript/CoffeeScript needed to play Albums, and we should be able to adjust things slightly to work with Playlists.  There is one quandary to deal with though.  

The Audio model has a field (album_order) that determines where in the Album the Audio is located.  Since an Audio can be part of many Playlists and a Playlist has many Audios it doesn’t make sense to add a *playlist_order* field to either of them.  

Fortunately Rails comes with the ability to **has_many :through** which allows for adding additional fields to the “pointer” table linking the two models.  Placing the **playlist_order** field on that table should allow us to say this Audio goes with this Playlist in this order.  Pow!

## Adjusting the Model

The first step is to create a new migration:

```
bin/rails g migration adjust_playlist_audios
```

Now edit the **db/migrate/DATESTAMP_adjust_playlist_audios.rb** file:

```
    drop_table :audios_playlists

    create_table :playlist_audios do |t|
      t.belongs_to :playlist, index: true
      t.belongs_to :audio, index: true
      t.integer :playlist_order

      t.timestamps null: false
    end
```

Place the above in the **change** method, then execute the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

Create a new model **app/models/playlist_audio.rb** with:

```
class PlaylistAudio < ActiveRecord::Base
  belongs_to :audio
  belongs_to :playlist
end
```

Now edit **app/models/playlist.rb** and add the associations:

```
  has_many :playlist_audios
  has_many :audios, through: :playlist_audios
```

And similar for **app/models/audio.rb**:

```
  has_many :playlist_audios
  has_many :playlists, through: :playlist_audio
```

## Playlist Show

To generic things up we’ll create a new partial in **app/views/layotus/_collections.html.erb**.  This will use **@collection** instead of **@playlists/@albums**.  This allows us to use one template for both Playlists and Albums:

```
<h2><%= @collection.name %></h2>
<div class="row">
  <div class="columns large-4">
    <%= image_tag @collection.image.url, class: 'th' if @collection.image %>
  </div>
  <div class="columns large-6 large-offset-1">
    <ul class="no-bullet collection-details">
      <% if controller_name == 'playlists' %>
        <li>
          <strong>Description:</strong> <%= @collection.description %>
        </li>
      <% else %>
        <li>
          <strong>Artist:</strong> <%= @album.artist %>
        </li>
        <li>
          <strong>Released:</strong> <%= @album.year.strftime('%Y') if @album.year %>
        </li>
        <li>
          <strong>Genre:</strong> <span class="label warning"><%= @album.genre %></span>
        </li>
      <% end %>
    </ul>
  </div>
</div>

<br/>

<div class="row">
  <div class="columns large-6">
    <h4>Play <%= @collection.name %></h4>
    <div class="controls">
      <button class="warning large control change" data-function="-1" data-collection="<%= @collection.id %>"><i class="fi-previous"></i></button>
      <button id="play" class="warning large control" data-function="play" data-collection="<%= @collection.id %>"><i class="fi-play"></i></button>
      <button id="pause" class="warning large control" data-function="pause" data-collection="<%= @collection.id %>"><i class="fi-pause"></i></button>
      <button class="warning large control change" data-function="1" data-collection="<%= @collection.id %>"><i class="fi-next"></i></button>
      <br/>
      <button id="looper" class="warning small control" data-function="looper" data-collection="<%= @collection.id %>"><i class="fi-loop"></i></button>
      <button id="shuffle" class="warning small control" data-function="shuffle" data-collection="<%= @collection.id %>">
        <i class="fi-shuffle"></i>
      </button>
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
  <div class="columns large-6">
    <% unless @collection.audios.empty? %>
      <ul id="audio-list" class="no-bullet">
        <% @collection.audios.each do |audio| %>
          <li data-audio="<%= audio.id %>" class="audio" data-collection="<%= @collection.id %>"
            data-playlist-audio="<%= audio.playlist_audios.find_by(audio_id: audio.id, playlist_id: @playlist.id).id  if @playlist %>">

            <div class="row">
              <div class="columns large-12">
                <strong><%= link_to audio.name, audio %></strong>
                <br/>

                <div class="row">
                  <div class="columns large-6">
                    <audio id="<%= audio.id %>" controls data-audio="<%= audio.id %>" class="player" autobuffer=“false”>
                      <% if audio.path.index('http') %>
                        <source src="<%= audio.path %>"></source>
                      <% else %>
                        <source src="<%= stream_path(audio.id) %>"></source>
                      <% end %>
                    </audio>
                  </div>
                  <div class="columns large-2 right">
                    <div class="button button tiny secondary handle left">
                      <i class="fi-arrows-in"></i>
                    </div>
                  </div>
                </div>

              </div>
            </div>

          <% end %>
        </li>
      </ul>
    <% end %>
  </div>
</div>


<br/><hr/><br/>

<% if controller_name == 'playlists' %>
  <%= link_to 'Edit', edit_playlist_path(@playlist), class: 'button small' %> |
  <%= link_to 'Back', playlists_path, class: 'button small' %>
<% else %>
  <%= link_to 'Edit', edit_album_path(@album), class: 'button small' %> |
  <%= link_to 'Back', albums_path, class: 'button small' %>
<% end %>
```  

Let’s adjust the **app/views/playlists/show.html.erb** and **app/views/albums/show.html.erb** files to have only:

```
<%= render 'layouts/collections' %>
```

Then change the **show** method in **app/controllers/playlists_controller.rb**:

```
  def show
    @collection = @playlist
    @last_audio = Audio.find(@playlist.current_audio) if @playlist.current_audio
  end
```

And to keep our code duplication down to a minimum change the **show** method in **app/controllers/albums_controller.rb**:

```
  def show
    @collection = @album
    @last_audio = Audio.find(@album.current_audio) if @album.current_audio
  end
```

Basically just adding the **@collection** variable.

## JavaScripts (CoffeeScripts)

Because we’ve already developed our CoffeeScript to play an Album we can change things to be more general to accommodate the new Playlist model.  To achieve this, rename the **app/assets/javascripts/albums.coffee** file to **app/assets/javascripts/collections.coffee** (renaming it collections seems to cover both Playlists and Albums) and at the top of the file change the **ready_albums** function to:

```
ready_collections = ->
  #
  # Determine the action from the pathname.
  #
  paths = window.location.pathname.split('/')
  action = paths[1]
  object_id = paths[2]
```

Don’t forget to change the bottom function calls:

```
$(document).ready(ready_collections)
$(document).on('page:load', ready_collections)
```

Edit the **media_control** object:

```
@media_control = {
  play: (action, id) ->
    $('#play').toggleClass('warning')
    if !$('#pause').hasClass('warning')
      $('#pause').toggleClass('warning')

    $.get("/#{action}/#{id}.json")
      .then (collection) ->
        if collection.current_audio?
          if !media_control.current_audio?
            media_control.current_audio = media_control.find_current_audio(collection)
        else
          media_control.current_audio = {
            collection: collection,
            audio: collection.audios[0],
            current_player: $('#' + collection.audios[0].id)[0],
          }

        media_control.current_audio.current_player.play()
        media_control.update_collection(action, collection, media_control.current_audio)


  pause: (id) ->
    if media_control.current_audio?
      $('#pause').toggleClass('warning')
      if !$('#play').hasClass('warning')
        $('#play').toggleClass('warning')

      if media_control.current_audio.current_player.paused
        media_control.current_audio.current_player.play()
        $('#play').toggleClass('warning')
      else
        media_control.current_audio.current_player.pause()

  change_audio: (id, direction, action) ->
    if !media_control.current_audio?
      media_control.play(action, id)
    else if media_control.current_audio?
      media_control.current_audio.current_player.pause()

      # Find the index of the next/previous Audio
      audios = media_control.current_audio.collection.audios
      for audio, idx in audios
        if media_control.current_audio.audio.id == audio.id
          current = idx + direction
          if current == audios.length
            current = 0
          else if current == -1
            current = media_control.current_audio.collection.audios.length - 1
          media_control.current_audio = {
            collection: media_control.current_audio.collection
            audio: audios[current],
            current_player: $('#' + audios[current].id)[0],
          }
          break
    else if media_control.current_audio.collection.current_audio?
      # Collection isn't playing so play the last audio.
      media_control.current_audio = media_control.find_current_audio(media_control.current_audio.collection)
    else

      # Collection doesn't have a current_audio so start the first, or the last, audio.
      if direction == -1
        current = media_control.collection.audios.length - 1
      else
        current = 0

      media_control.current_audio = {
        collection: media_control.current_audio.collection,
        collection: media_control.current_audio.collection.audios[current],
        current_player: $('#' + media_control.current_audio.collection.audios[current].id)[0],
      }

    media_control.current_audio.current_player.play() if media_control.current_audio.current_player
    media_control.update_collection(action, media_control.current_audio.collection, media_control.current_audio)

  looper: (id) ->
    # Give some feedback.
    $('#looper').toggleClass('warning')

    if media_control.looping?
      media_control.looping = false
    else
      media_control.looping = true

  shuffle: (id) ->
    $('#shuffle').toggleClass('warning')
    if media_control.shuffling?
      media_control.shuffling = false
    else
      media_control.shuffling = true


  find_current_audio: (collection) ->
    for audio, idx in collection.audios
      if audio.id == collection.current_audio
        audio = audio
        current_player = $('#' + audio.id)[0]

        return {
          collection: collection,
          audio: audio,
          current_player: $('#' + audio.id)[0],
        }

  update_collection: (action, collection, current_audio) ->
    if action == 'albums'
      $('#playing_audio').html(current_audio.audio.name + ' <em>' + current_audio.audio.album_order + '</em>')
      field = 'album[current_audio]='
    else
      $('#playing_audio').html(current_audio.audio.name + ' <em>' + current_audio.audio.name + '</em>')
      field = 'playlist[current_audio]='

    $.ajax({
      url: "/#{action}/#{collection.id}.json",
      method: 'put',
      data: field + current_audio.audio.id
    })
}
```

There’s a lot of code in there, but basically the word “album” has been replaced with the word “collection” to make things more generic.  The URLS and fields to be PUT have beed adjusted to the **action** variable set at the top of the file.  There are a few more clean up items, like renaming a misspelled function name, but of the most part it’s the same as what we used before.

## Re-ordering Playlists

The last piece of the puzzle is to adjust the code for re-ordering Audios.  First, add a new route to **config/routes.rb**:

```
  put '/playlist_audios/:id', to: 'playlists#update_playlist_audio', as: :update_playlist_audio
```

Next, adjust the **has_many through:** statement in **app/models/playlist.rb** to include an **order** lambda:

```
  has_many :audios, -> { order 'playlist_audios.playlist_order' }, through: :playlist_audios
```

It looks a little odd, at least to me, but it works fine.  Now in **app/controllers/playlists_controller.rb** create a new **update_playlist_audio** method:

```
  def update_playlist_audio
    playlist_audio = PlaylistAudio.find(params[:id])
    if playlist_audio
      playlist_audio.update(playlist_params)
      render json: true
    else
      render json: { error: playlist_audio.errors.full_messages }
    end
  end
```

Also, add **:playlist_order** to the permitted parameters in the **playlist_params** method:

```
      params[:playlist].permit(:name, :description, :image, :current_audio, :playlist_order, :audio_ids => [])
```

Finally, update the “Handle re-ordering of Audios” section of **app/assets/javascripts/collections.coffee**:

```
  #
  # Handle re-ordering of Audios.
  #
  $('#audio-list').sortable({
    handle: '.handle',
    sortableClass: 'fade',
  })

  $('#audio-list').bind 'sortupdate', (e, ui) ->
    # Find the audio at the oldIndex.
    $oldIndexAudio = $($('li.audio')[ui.oldindex])

    # Determine the correct order field.
    if action == 'albums'
      field = 'audio[album_order]='
      old_url = "/audios/#{$oldIndexAudio.data().audio}.json"
      new_url = "/audios/#{ui.item.data().audio}.json"
    else
      field = 'playlist[playlist_order]='
      new_url = "/playlist_audios/#{ui.item.data().playlistAudio}"
      old_url = "/playlist_audios/#{$oldIndexAudio.data().playlistAudio}"

    # Send a PUT to each Audio to update the album_order field.
    $.ajax({
      url: old_url
      method: 'put',
      data: field + (ui.oldindex + 1)
    })

    $.ajax({
      url: new_url
      method: 'put',
      data: field + (ui.index + 1)
    })
```

So now we’re sending different data and to different URLs based on if it’s a Playlist or an Album.

## Styles

To generalize the styles, and have them match the changes in the show templates, edit the **app/assets/stylesheets/albums.scss** (you can keep this file the same name, or you could rename it to collections.css):

```
.collection-details {
  font-size: 24px;
}

#audio-list {
  border: 1px solid #dddddd;
  padding: 10px;

  & li {
    padding: 10px;
  }
  & li:nth-child(odd) {
    background-color: #eeeeee;
  }
}
```

## Conclusion

It’s awesome to be able to play both an Album and a Playlist as well as re-ordering the Audio files that are part of each.  There are a few wonky bugs in the process, but I think we’ll be able to iron them out in future posts.

Party On!