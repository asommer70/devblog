# Now Playing, Playing Now

## Robust Navs

For most of the web apps I’ve developed I’ve gone with a think navigation bar at the top of the page.  Up until now I’ve tried to keep the nav bar to a minimum in [Audio Pila!](https://github.com/asommer70/audiopila-rails).  Some of the reason is to quickly blast in new features, but also I like the clean minimal look of a small nav bar.

The question now is do I put playback controls for Albums and Playlists in a nice thick nav bar, or keep things minimal somehow?

Let’s keep the current nav and see how that serves our needs…

## Updating the Nav

Edit the **app/views/layouts/_nav.html.erb** file:

```
  <dd id="g_current_audio" class="right">
  </dd>
  <dd id="controller" class="right">
    <div class="controls">
      <button id="g_prev" class="warning tiny global control change" data-function="-1" data-collection=""><i class="fi-previous"></i></button>
      <button id="g_play" class="warning tiny global control" data-function="play" data-collection=""><i class="fi-play"></i></button>
      <button id="g_next" class="warning tiny global control change" data-function="1" data-collection=""><i class="fi-next"></i></button>
      &nbsp;&nbsp;
      <button id="g_looper" class="warning tiny global control" data-function="looper" data-collection=""><i class="fi-loop"></i></button>
      <button id="g_shuffle" class="warning tiny global control" data-function="shuffle" data-collection="">
        <i class="fi-shuffle"></i>
      </button>
    </div>
  </dd>
```

We’ve added two **dd** elements one for a place to display the name and Album/Playlist of the currently playing Audio.  And the other *dd* element contains control buttons very similar to those on the Album/Playlist pages.  The only difference is that we’ve adjusted the *id* attribute and will add the **data-collection** after there is a **media_control.current_audio** object.

## New JavaScripts

Now we wire up the “global” control buttons with some JavaScript/CoffeeScript.  Edit the **app/assets/javascripts/collections.coffee** file:

```
```