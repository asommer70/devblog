# Albums 3rd Draft


## Album Order

While writing up the [Albums 2nd Draft](http://devblog.thehoick.com/rails/audiopila/2015/11/17/albums-2nd-draft.html) post I added an integer field to the Audio class to determine the order of Audio files in an Album, but we never added a field to the Audio form to change things.

After some thinkering, I’ve come to the conclusion that the best way to change the order of an Audio in an Album is to use a drag ’n drop list re-order the Audio list on the Album show page.  That way we can change the **album_order** field for two Audios at one time… in theory.

## Dragin ’n Droppin

To make drag ’n drop much easier we’ll use the [html5sortable](https://github.com/voidberg/html5sortable) library and like the Chosen library we’ll place it in the vendor asset pipeline. 

Download the latest release archive from [here](https://github.com/voidberg/html5sortable/releases) and extract the contents.  You should have a **html5sortable-X.X.X** folder where X.X.X is the current version number.  Inside that directory copy the **dist/html.sortable.js** file to **vendor/assets/javascripts**.

Then add it to the requires in **app/assets/javascripts/application.js** file:

```
//= require html.sortable
```

## New Layouts

Let’s adjust the **app/views/albums/show.html.erb** file:

```

<h2><%= @album.name %></h2>
<div class="row">
  <div class="columns large-4">
    <%= image_tag @album.image.url, class: 'th' if @album.image %>
  </div>
  <div class="columns large-6 large-offset-1">
    <ul class="no-bullet album-details">
      <li>
        <strong>Artist:</strong> <%= @album.artist %>
      </li>
      <li>
        <strong>Released:</strong> <%= @album.year.strftime('%Y') if @album.year %>
      </li>
      <li>
        <strong>Genre:</strong> <span class="label warning"><%= @album.genre %></span>
      </li>
    </ul>
  </div>
</div>

<br/>

<div class="row">
  <div class="columns large-6">
    <h4>Play Album</h4>
    <div class="controls">
      <button class="warning large control change" data-function="-1" data-album="<%= @album.id %>"><i class="fi-previous"></i></button>
      <button id="play" class="warning large control" data-function="play" data-album="<%= @album.id %>"><i class="fi-play"></i></button>
      <button id="pause" class="warning large control" data-function="pause" data-album="<%= @album.id %>"><i class="fi-pause"></i></button>
      <button class="warning large control change" data-function="1" data-album="<%= @album.id %>"><i class="fi-next"></i></button>
      <br/>
      <button id="looper" class="warning small control" data-function="looper" data-album="<%= @album.id %>"><i class="fi-loop"></i></button>
      <button id="shuffle" class="warning small control" data-function="shuffle" data-album="<%= @album.id %>">
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
    <% unless @album.audios.empty? %>
      <ul id="album-audio-list" class="no-bullet">
        <% @album.audios.each do |audio| %>
          <li data-audio="<%= audio.id %>" class="album_audio" data-album="<%= @album.id %>">

            <div class="row">
              <div class="columns large-12">
                <strong><%= link_to audio.name, audio %></strong>
                <br/>

                <div class="row">
                  <div class="columns large-6">
                    <audio id="<%= audio.id %>" controls data-audio="<%= audio.id %>" class="player">
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


<%= link_to 'Edit', edit_album_path(@album), class: 'button small' %> |
<%= link_to 'Back', albums_path, class: 'button small' %>
```

There are quite a few changes here.  The layout has been adjusted to use only one Album image and to put the “meta data” fields next to it.  Then under that we have the Album play controls with the Audios list next to it.  Makes things more compact and uses the lateral space a little better.  Also, notice the new **data** attributes on the **li** elements in the Audios list.  Those will be used to handle the re-ordering of things.

## JavaScript

To enable the **li** (audio) elements to be re-sorted and to update the objects in the database we need to add some JavaScript.  Edit **app/assets/javascripts/albums.coffee** and add the following before the **@media_control** object:

```
  #
  # Handle re-ordering of Audios an Album.
  #
  $('#album-audio-list').sortable({
    handle: '.handle',
    sortableClass: 'fade',
  })

  $('#album-audio-list').bind 'sortupdate', (e, ui) ->

    # Find the audio at the oldIndex.
    $oldIndexAudio = $($('.album_audio')[ui.oldindex])

    # Send a PUT to each Audio to update the album_order field.
    $.ajax({
      url: '/audios/' + $oldIndexAudio.data().audio + '.json'
      method: 'put',
      data: 'audio[album_order]=' + (ui.oldindex + 1)
    })

    console.log('audio[album_order]=' + (ui.index + 1))
    $.ajax({
      url: '/audios/' + ui.item.data().audio + '.json'
      method: 'put',
      data: 'audio[album_order]=' + (ui.index + 1)
    })
```

This code sets the **album-audio-list** ul element to be sortable via the *html5sortable* library.  Then the **sortupdate** event is bound via jQuery.  This event let’s the DOM know that the drag and drop has completed.  Inside the bind function we get the element at the position where the dragged element was. Then a PUT is sent via AJAX for both the drug/dragged element and the one that replaced it.  Not 100% accurate as far as the numbers in the **album_order** field are concerned, but is seems to work for now.

## Style Updates

Also, apply some additional styles in the **app/assets/stylesheets/albums.scss** file:

```
.fade {
  opacity: 0.5;
}

.album-details {
  font-size: 24px;
}

#album-audio-list {
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

This is a shorter post then the last few, but I wanted to make sure it was clear how to adjust the playback order of an Album.  Plus we’ll need to do something similar for playback of Playlists.

On to Playlist playback…

Party On!