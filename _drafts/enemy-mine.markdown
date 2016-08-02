---
title: "Enemy Mine"
date:   2016-08-11 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: asteroids.png
---

## Remember That Movie From The 80s

Coming up with the title or this post made me remember that movie [Enemy, Mine](https://en.wikipedia.org/wiki/Enemy_Mine_(film)) from the 80s.  I think it was starring Dennis Quaid and Lewis Gossit Jr.  They were enemies crash landed on a hostile deserted world and had to work together to survive. Then of course they become friends and have a bunch of adventures.

Interesting that they were both starship pilots and I'm working on a space shooter game... cause you know I liked them when I was a kid... in the 80s...

<!--more-->

## Making Sparks

To copy another element from [Cave Berries](http://www.lexaloffle.com/bbs/?tid=1834), because I really like the way the laser sparks when it hits a wall, I added a **sparks** function/method to the **bulletconstruct** function:

```
obj.sparks = function(this)
  if (this.des > 0) then
    for i = 1,4 do
      rnd_x = flr(rnd(12))
      modx = 2
      rnd_y = flr(rnd(4))
      c = 11

      if (i % 2 == 0) then rnd_x += modx, c = 3 end

      pset(this.position.x - 4 + rnd_x, this.position.y + rnd_y, c)
    end
  end
end
```

The **sparks** method (I'm going to go ahead and refer to a function on an object/table as a method) also needs to be called in the **_draw** method in order to actually have the **pset** function work:

```
foreach(bullets, function(obj)
    spr(obj.sprite, obj.position.x, obj.position.y)
    obj.sparks(obj)
end)
```

The above is the updated **foreach bullets** in the **_draw** function.  Also, inside the **bulletconstruct** **destroy** method call it:

```
obj.destroy = function(this)
  this.des += 1
  this.sparks(this)
  this.hitbox = {x=0,y=0,w=0,h=0}
end
```

There now we have a cool visual... but no sound effect cause you know... space and all.
