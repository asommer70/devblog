---
title: "You're Not Going Into The Asteroid Field?"
date:   2016-08-02 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: asteroids.png
---

## Space Rocks

Adding asteroids to the mix seems like a good idea.  They don't have bullets (hopefully) so they should be relatively easy to have float down the screen.  Then again maybe they'll come into the the scene from any angle... haven't really decided on that part yet.

I think using the same type of object with an update method and an "array" of objects as we used with the bullets/lasers should work for asteroids too.

<!--more-->

## Creating Asteroids

Whip up a **astroidconstruct** function super similar to the *bulletconstruct* function:

```
asteroidconstruct = function()
  local obj = {}

  -- only come in from random top area.
  x = flr(rnd(127))

  -- set the direction randomly left, right, or center
  xs = {0, 1, 2}
  rnd_x = xs[flr(rnd(3)) + 1]
  if(rnd_x == 2)then rnd_x = -1 end
  text = rnd_x
  obj.dir = rnd_x

  obj.position = {x=x, y=0}
  -- obj.position = {x=52, y=52}
  obj.sprite = 17
  obj.update = function(this)
    this.position.y += 1
    this.position.x += obj.dir

    -- remove it when it goes off screen
    if(this.position.y>127)then del(asteroids, this)end
  end
  return obj
end
```

There will be differences though.  First off we set the **x** coordinate to a random value from 0 - 127.  Then a there's some statements to get a random direction for the asteroid left, right, or center.  The value is applied to the object.  The sprite and position are then set.

After all that the update "method" is used to make the asteroid track across the screen in a top to bottom ish movement.  Finally, after the asteroid leaves the screen it is removed from the **asteroids** object/array.

Here's a look at sprite 17:

![](/img/asteroid_sprite.png)

Oh ya, shading... woo!

## Creating Random asteroids

Now that we can create an astroid and have it blast down the screen in a southerly direction wee need to randomly send some 'roids through space.

After trying a few things out I came up with these relatively simple statements to be added to the **_update** function:

```
ast_timer += 1
if(ast_timer % 100 == 0)then add(asteroids, asteroidconstruct()) end
```

So if the **ast_timer** variable is divisible by 100 then a new asteroid is created and it's update function will meander it through the play area.  Which reminds me, back up in the **_init** function create the **ast_timer** (asteroid timer) variable:

```
function _init()
  -- states
  -- 0 idle
  -- 1 moving
  -- 2 firing

  hero = {
    x = 58,
    y = 100,
    sprite = 0,
    state = 0,
    pst = 0
  }

  bullets = {}
  asteroids = {}
  ast_timer = 0

  text = "welcome to rumpus blaster"
end
```

Great, now every so often a chunky asteroid will fly through.

## Updating Bullets

Adding the remove asteroid if off the screen line makes me realize something similar for bullets is needed too.

In the **update** function (or is it now a method???) inside the **bulletconstruct** function add this line after the *this.position.y** update line:

```
if(this.position.y<0)then del(bullets, this)end
```

## Conclusion

Things are progressing, but not quite as fast as I'd like.  Mostly due to the day job taking up more development time than I'd like, but then again it's nice to make some money.  

Another thing I've noticed while developing games is that the temptation to play the game I'm working on as well as other games is a lot hight than when I'm developing a web app, mobile app, etc.  I guess it's kind of obvious, but I didn't really take it into account before setting off on this adventure.

I think the next thing to work on is making bullets/lasers actually hit something...

Party On!
