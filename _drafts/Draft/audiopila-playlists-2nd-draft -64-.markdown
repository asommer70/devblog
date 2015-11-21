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

Let’s adjust the **app/views/playlists/show.html.erb** file:

```

```

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
    console.log('action:', action, 'id:', id)
    $('#play').toggleClass('warning')
    if !$('#pause').hasClass('warning')
      $('#pause').toggleClass('warning')

    $.get("/#{action}/#{id}.json")
      .then (collection) ->
        console.log('album:', collection)
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
          console.log('current:', current)
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
      console.log(media_control.current_audio.current_player)
      media_control.current_audio.current_player.play()
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
    console.log('setting loop...', id)
    # Give some feedback.
    $('#looper').toggleClass('warning')

    if media_control.looping?
      media_control.looping = false
    else
      media_control.looping = true

  shuffle: (id) ->
    console.log('shuffling...')
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
    console.log('update_collection collection:', collection)
    console.log('current_audio:', current_audio)
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

So basically the word “album” has been replaced with the word “collection” to make things more generic.  The URLS and fields to be PUT have beed adjusted to the **action** variable set at the top of the file.


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