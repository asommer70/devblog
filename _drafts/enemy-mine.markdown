---
title: "Enemy Mine"
date:   2016-08-11 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: enemy_fire.png
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

## First Enemy

As you might be able to tell I whipped up an enemy ship that shoots out a red and orange bullet/bomb/beam out our plucky hero.  To get things working we'll copy 'n paste a lot of code from the **asteroid** and **bullet** objects.  Add an **enemy** function:

```
function enemy()
  local obj = {}

  -- only come in from random top area.
  x = flr(rnd(127))

  -- set the direction randomly left, right, or center
  xs = {0, 1, 2}
  rnd_x = xs[flr(rnd(3)) + 1]
  if(rnd_x == 2)then rnd_x = -1 end
  obj.dir = rnd_x

  obj.position = {x=x, y=0}
  obj.sprite = 22

  obj.update = function(this)
    this.position.y += 1
    this.position.x += obj.dir

    if(this.position.x == hero.position.x or this.position.x + 8 == hero.position.x + 8)then obj.fire(obj) end

    -- remove it when it goes off screen
    if(this.position.y>127)then del(asteroids, this)end
    if(this.hp <= 0)then this.des += 1 end
    if(this.des >= 20)then this.sprite = 24 end
    if(this.des >= 40)then del(enemies, this) end
  end

  obj.explode = function(this)
    this.sprite = 23
  end

  obj.hitbox = {x=0, y=2, w=8, h=8}
  obj.hp = 10
  obj.des = 0

  obj.fire = function(this)
    if (this.des == 0) then
      text = "enemy fire!!! ..."

      add(bullets, bullet(this.position.x, this.position.y + 10, 'down', 6))
    end
  end

  return obj
end
```

At this point the **enemy** is pretty much the same as an asteroid.  Notice the single line **if** in the **update** method though.  If the enemy happens to line up with the hero on the x axis, then FIRE!  The **fire** method is new and basically adds a **bullet** to the **bullets** array.

Notice that the function call is different than what we used before.  I did some refactoring of the **$construct** functions and made them "first class".  Not sure what assigning a function to a variable gets you in Lua, but it seems to work the same as using the *function* keyword.  

Also, I setup the [symbols-tree-view](https://atom.io/packages/symbols-tree-view) package for Atom to be able to list the functions in a list in the right-hand side of the editor window.  I find it super useful when jumping around between functions.  The details can be found in [this](http://www.lexaloffle.com/bbs/?tid=3440) thread.  Thanks to [@Sanju](http://www.lexaloffle.com/bbs/?uid=11527) for pointing it out to me.

## Updated bullets

Here's the updated bullet definition:

```
function bullet(x, y, dir, sprite)
  local obj = {}
  obj.position = {x=x, y=y}
  obj.sprite = sprite
  obj.des = 0
  obj.dir = dir

  obj.update = function(this)
    if(this.des == 0 and this.dir == 'up')then this.position.y -= 6 end
    if(this.des == 0 and this.dir == 'down' and this.sprite == 6)then this.position.y += 4 end

    -- it's hit something
    if(this.des > 0)then this.destroy(this) end
    if(this.des >= 5)then del(bullets, this) end

    -- remove it when it goes off screen
    if(this.position.y<0)then del(bullets, this)end
  end

  obj.hitbox = {x=0, y=0, w=8, h=8}

  obj.destroy = function(this)
    this.des += 1

    this.sparks(this)
    this.hitbox = {x=0,y=0,w=0,h=0}
  end

  obj.sparks = function(this)
    -- if(this.des > 0)then pset(this.position.x, this.position.y, 10) end
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

  --return the bullet
  return obj
end
```

The main changes are in the parameter list.  Since enemies will be firing down and will use different beams we need to specify a direction and a sprite.  Then in the **update** method the direction is checked and the bullet's **y** position is either incremented or decremented.

## Updating update

Next, the **_update** function needs adjusted to take into account enemies and having enemy bullets hit our hero. Before we dive into that let's take a look at the **each_enemies** function.  The **_update** function was starting to get a little crowded so I moved some looping code into another function:

```
function each_enemies()
  foreach(enemies, function(enemy)
      enemy.update(enemy)

      -- collide enemies with bullets
      foreach(bullets, function(bullet)
        if (enemy.hp > 0) then
          if (collide(enemy, bullet)) then
            text = "enemy hit!!!"
            -- display bullet sprite
            bullet.destroy(bullet)

            enemy.hp -= 1

            -- knock it back a little
            enemy.position.y -= 2

            if(enemy.hp <= 0) then enemy.explode(enemy)end
          end
        end
      end)

      -- enemy collides with hero?
      if (collide(hero, enemy) and enemy.des == 0 and hero.des == 0) then
        sfx(01)
        enemy.hp -= 1
        hero.hp -= 1

        -- knock it back a litte
        enemy.position.y -= 2
        enemy.position.x -= enemy.dir + enemy.dir

        if(enemy.hp <= 0) then enemy.explode(asteroid) end
        if(hero.hp <= 0) then hero.explode(hero) end
      end
  end)
end
```

Basically the same as the asteroids loop this function uses a **foreach** to loop the **enemies** array (added to the **_init** function) and inside the enemies loop the **bullets** array is looped an **collide** is used to check for hits on the enemy ship.  Like the astroids loop **collide** is also used to check for collisions with the hero ship.  Thinking to have a "drone" type enemy that will try to blast into the hero, but we're not quite there yet.

Then inside the **_update** function add:

```
each_enemies()
```

Getting due for some major refactoring, but that'll probably be a post all by itself.

## Hero Bullet hits

In the **hero** object setup inside the **_init** function we need to do another bullet loop:

```
-- bullet hits hero
foreach(bullets, function(bullet)
  if (this.hp > 0) then
    if (collide(this, bullet)) then
      text = "hero hit!!!"
      sfx(4)

      -- display bullet sprite
      bullet.destroy(bullet)

      this.hp -= 1

      -- knock it back a little
      this.position.y += 2

      if(this.hp <= 0) then this.explode(this)end
    end
  end
end)
```

The loop is placed in the **hero's** **update** method and is similar to the other bullet loops.  This loop uses **collide** to check if they are hitting the hero, sets the text for debug purposes, plays a sound cause there's probably be some noise when something hits the outside of a ship in space, subtracts a *hp* while knocking the hero back a little, and then a check to see if the hero is out of *hp* and if so call the **explode** method.

## Conclusion

Currently the enemies only enter the screen when the btn(5) is pressed.  So I think next I'll work on some AI/attack patterns.  Should make the game more fun.

Then the great refactor might happen.  Not sure when I'll hit the token ceiling, but I feel like I've still got some space... (heh)

Party On!
