---
title:  "Firing Freakin Lasers"
date:   2016-07-28 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: hero_lasers.png
---

## FIRE!!!

When I hear the word FIRE (all caps necessary) I always think of Captain Kirk at the end of Undiscovered Country:

<iframe width="420" height="200" src="https://www.youtube.com/embed/txVh2QgRmHs" frameborder="0" allowfullscreen></iframe>

And it's now time to make our hero ship fire it's own weapons.  In this case we'll start with lasers and go from there.  Thinking to have some type of bomb/missile to add a little extra kick to our plucky hero.

Plucky, ya definitely plucky!

<!--more-->

## Adding the Sprite

In this case the sprite is borrowed again from [Cave Berries](http://www.lexaloffle.com/bbs/?tid=1834).  Except the color is going to be changed up to green and dark_green.

I created a simple alternating green/dark_green pattern on the edge of the 8x8 sprite position.  This will match up with the "top" of the hero ship's "laser pylons".

![](/img/laser_sprite.png)

## Finding Help

I found it kind of tricky to have the laser fire when the *Z* key is pressed.  It's simple to make the sprite appear when **btn(4)** returns **true**, but it's more complicated to then have the sprite "fly" up the screen.

I guess when you think about it it's the same principal as making the hero ship move up the screen...

Either way I was about to post a question to the forum asking if there are any good "bullet/projectile" tutorials for Pico-8 games, but then I said to myself "self, you should check all the PICO-8 Zines first".  So even though I was only about half way through PICO-ZINE #2, I went ahead and downloaded [#3](https://sectordub.itch.io/pico-8-fanzine-3).  And blamo! A great tutorial by [@schminitz](http://hauntedtie.be/).

## Coding the laser

Or rather coding the **bullets**.  I grabbed the **bulletconstruct** function pretty much directly from the tutorial, but I did make some modifications to adapt it for my game:

```
bulletconstruct = function(x, y)
  local obj = {}
  --an array containing x and y position
  obj.position = {x=x, y=y}

  --the sprite number used to draw the bullet
  obj.sprite = 16

  --define an ‘update’ function that will be called by the program
  obj.update = function(this)
    --move the bullet to the right
    this.position.y -= 6
  end

  --return the bullet
  return obj
end
```

The main change is to update the **this.position.y** instead of the *x* position and to subtract **6** because my "bullets/lasers" are longer.  Plus cranking it up to 6 makes them move faster.

The tutorial says to loop over all the objects in the game in the _draw and _update functions, but I'm not there yet so I created a **bullets** global object at the top and adjusted the **foreach** loops like so:

```
function _update()
  move_hero()
  hero_fire()

  foreach(bullets, function(obj)
      obj.update(obj)
  end)
end

function _draw()
  cls();

  updatetext()

  -- draw hero
  drawhero()
  move_hero()

  foreach(bullets, function(obj)
      spr(obj.sprite, obj.position.x, obj.position.y)
  end)
end
```

I then created the **hero_fire** function:

```
function hero_fire()
  if btnp(4, 0) then
    sfx(00)

    rx = hero.x
    lx = hero.x

    ry = hero.y - 5
    ly = hero.y - 5

    add(bullets, bulletconstruct(rx, ry))
    add(bullets, bulletconstruct(lx, ly))
  end
end
```

Using the **btnp** function I learned about in the tutorial when button 4 is pressed the left and right "laser pylon" positions are calculated then the **bullets** are added using the **add** function to the global **bullets** object.

## Laser Sound, pew pew

Notice the **sfx** call at the beginning of the **hero_fire** function.  I whipped up a simple, sort of muted, sound effect:

![](/img/hero_laser_sfx.png)

This will probably need to be revisited, but it works for now and is great feedback.

## Conclusion

Getting this simple functionality working is a great way to learn Pico-8 development, and I guess some Lua too.  I keep coming across other things I'll need, or potentially need, to do.  Things like a **finite state machine**, whatever that is, and keeping an object of game "characters" to be able to update them all at once.

Fun stuff!

Party On!
