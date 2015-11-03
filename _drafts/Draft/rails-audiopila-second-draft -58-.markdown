# Audio Pila! Second Draft

## Improvements Along The Way

There are quite a few improvements we can make in our Audio Pila! Rails app.  To name a few (but let’s not get carried away just yet):

* Make the Audio index page display multiple repositories better.
* Be able to create Albums form collections of Audios.
* If we can make Albums, we’ll need a way to upload images to display with the Album.

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

Don’t forget you can start the Rails development server with:

```
bin/rails s
```

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

## Album Model

## Dragonfly Image Uploads

## Tests

## Conclusion