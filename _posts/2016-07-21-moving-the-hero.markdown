---
title:  "Moving The Hero"
date:   2016-07-21 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: hero_ship.png
---

## Movement

I learned how to move an "object"/"pixels" around the screen while working through the Pico-8 Zine paddle game example.  It seems to work well and I was quickly able to move the paddle back and forth.  Very satisfying.

While looking at the source of other games I noticed they used a pretty similar approach.  One thing they did use that seems more segmented is to use **Tables** to create key/value pairs for attributes of an object.  

Coming from web development this looks suspiciously like an object...

<!--more-->

## The Hero

In this case I created a **hero** table and set some starting attributes:

```
-- hero
hero = {}
hero.x = 58
hero.y = 100
hero.sprite = 0
```

So the hero will use **sprite 0** and start at **x 58** and **y 100** which is the lower middle of the screen.

## move_hero()

With the hero placed the **move_hero()** function does the heavy lifting:

```
function move_hero()
  if btn(0) then
    hero.x -= 1
  elseif btn(1) then
    hero.x += 1
  elseif btn(2) then
    hero.y -= 2
  elseif btn(3) then
    hero.y += 2
  end
end
```

Notice that moving along the **x** axis is incremented/decremented by **1** instead of **2** along the **y** axis.  I thought it'd be interesting if the hero had main engines and smaller maneuvering thrusters for side to side movement.

## Exhaust

I thought it would be cool to have an **exhaust** sprite coming out of the ship depending on which direction it's moving.  To start that off I created four different versions of the hero ship.  One with the "aft" engine lit, one with the "fore" engine lit, one with a starboard thruster lit, and a final one with a port thruster lit.  

Like the nautical terms?  Don't worry I had to look it up on [Wikipedia](https://en.wikipedia.org/wiki/Port_and_starboard) to remember which one is right and left.

The **move_hero()** function now looks like:

```
function move_hero()
  if btn(0) then
    text = "button 0, left..."

    hero.sprite = 2
    spr(006, hero.x + 8, hero.y)

    hero.x -= 1
  elseif btn(1) then
    text = "button 1, right..."

    hero.sprite = 3
    spr(007, hero.x - 8, hero.y)

    hero.x += 1
  elseif btn(2) then
    text = "button 2, up..."

    hero.sprite = 0
    spr(004, hero.x, hero.y + 8)

    hero.y -= 2
  elseif btn(3) then
    text = "button 3, down..."

    hero.sprite = 1
    spr(005, hero.x, hero.y - 8)

    hero.y += 2
  end
end
```

As you can see I also added a **text** "element" that will change depending on which button is pushed.  This seemed like a nice debugging technique to make things clear while in the game.

![](/img/hero_ship_sprites.png)

## update and draw

The **_update** and **_draw** functions are pretty simple:

```
function _update()
  move_hero()
end

function _draw()
  cls();

  updatetext()

  -- draw hero
  drawhero()
  move_hero()
end
```

I'm calling **move_hero()** in _update and _draw because it wouldn't show the exhaust sprite unless I called **move_hero** at least once in **_draw**.  I guess that makes sense, but my first thought was that you could call **spr** in any function and have it draw to the screen even if the calling function is only executed in **_update**.  Guess that is not the case.

Also, the little debugging function **updatetext** is:

```
text = "welcome to rumpus blaster"

function updatetext()
  print(text, 12, 6, 15)
end
```

Lastly, gotta have the **cls()** call in _draw to clear the screen.  Pretty interesting if you play another game then load yours without calling **cls**...

## Conclusion

That's a quick rundown of what I have so far.  It's been a fun to allow a character to move along both axis.  To improve the exhaust I'd like to make it sort of sparkly.  We'll leave that for the next post.

Party On!
