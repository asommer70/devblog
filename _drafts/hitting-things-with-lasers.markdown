---
title: "Hitting Things With Lasers"
date:   2016-08-02 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: asteroids.png
bbcode: true
---

## A Hit, A Hit, A Very Palpable Hit

We've got lasers firing and astroids asteroiding, or whatever it is they do, so now we need to be able to actually hit objects with our lasers.  And the reverse, to know when an object has been hit by a laser, or another object for that matter.

Since starting this foray into game development I've continually been impressed about how supportive and helpful the Pico-8 community is.  The forums have loads of great posts with comments that are usually helpful and questions ask to help someones understanding.  Very fun to be a part of it in whatever small way I can.

## Updating bullets

As the guide in [Pico-8 Zine #3](https://sectordub.itch.io/pico-8-fanzine-3) teaches the bullets (lasers in this case) need to have a **hitobx** to determine when they collide with something.  With the hero laser there are two "beams" so we'll need to add a hitbox for each one.  Update the **bulletconstruct** function with:

```
obj.hitbox = {x=0, y=0, w=8, h=8}
```

This uses the entire sprite for a hitbox, but I think it makes sense because the "beams" are spread apart and if something is hitting it from the side it's still hitting it. Ya, I think that makes sense.

Next, add a hitbox (and an *hp* value which we'll get to later) to the astroid in the **asteroidconstruct** function:

```
obj.hitbox = {x=0, y=2, w=8, h=8}
obj.hp = 5
obj.des = 0
```

The hero will also need a hitbox in the **_init** function add:

```
hitbox = {x=4, y=3, w=8, h=8},
```

## Actually Doing The Hitting

Everything now has a hitbox so it's time to copy/integrate some more code from the **DOM8VERSE** article.  In the **foreach asteroids** loop inside the **_update** function add a **bullets** loop:

```
foreach(asteroids, function(asteroid)
  asteroid.update(asteroid)
  foreach(bullets, function(bullet)
    if (asteroid.hp > 0) then
      if (collide(asteroid, bullet)) then
        del(bullets, bullet)
        asteroid.hp -= 1

        -- knock it back a little
        asteroid.position.y -= 2
        asteroid.position.x -= asteroid.dir + asteroid.dir

        if(asteroid.hp <= 0) then asteroid.explode(asteroid)end
      end
    end
  end)
end)
```

Inside the inner **bullets** foreach we're checking the **asteroid.hp** to make sure it's not **0** then checking to see if a bullet is colliding with the astroid.  If a bullet has hit the astroid it is deleted from the **bullets** array, the astroid's **hp** is decremented by 1 and it is blasted back a little.  Also, if the asteroid's **hp** is **<= 0** (not sure if it can actually go below zero since it's being decremented by 1 each time, but better safe than sorry) the asteroid's **explode** function/method is called.

I used the same **collide** function as the guide:

```
function collide(obj, other)
  if other.position.x+other.hitbox.x+other.hitbox.w > obj.position.x+obj.hitbox.x and
      other.position.y+other.hitbox.y+other.hitbox.h > obj.position.y+obj.hitbox.y and
      other.position.x+other.hitbox.x < obj.position.x+obj.hitbox.x+obj.hitbox.w and
      other.position.y+other.hitbox.y < obj.position.y+obj.hitbox.y+obj.hitbox.h then

    return true
  end
end
```

It's a great way to see if two objects have connected their hitboxes and I'm not sure if I can improve it.

## Exploding asteroids

After five hits by a laser it's bye bye asteroid, but it'd be cool to have it sort of come apart and degrade.   In the **asteroidconstruct** function add:

```
obj.explode = function(this)
  this.sprite = 18
end
```

This function/method (need to look up what the proper term is for lua functions on an object) simply set's the asteroid's sprite to a new one.  Here's a screeny:

![](/img/exploded_asteroid.png)

Changing the sprite once isn't all that great, but changing it again after a few cycles is a little cooler.  Add these lines to the **update** function/method in **astroidconstruct**:

```
obj.update = function(this)
  this.position.y += 1
  this.position.x += obj.dir

  -- remove it when it goes off screen
  if(this.position.y>127)then del(asteroids, this)end
  if(this.hp <= 0)then this.des += 1 end
  if(this.des >= 20)then this.sprite = 19 end
  if(this.des >= 40)then del(asteroids, this) end
end
```

Now the asteroid's update function will check if the **des** attribute is no longer 0 (des for destroyed, heh) and if so increment it.  After des is greater than 20 the sprite changes again and after it reaches 40 the astroid is removed.  

It's pretty much the effect I was going for.

## Ship Bashing

Destroying asteroids is one thing, but what happens if they crash into our hero?  The code to check for hero hits is pretty much the same as for an asteroid.  I created an **update** function on the **hero** object setup in the **_init** function and moved some code from the **_update** function into it:

```
update = function(this)
  -- text = "updating hero..."
  b0=btn(0)
  b1=btn(1)
  b2=btn(2)
  b3=btn(3)
  b4=btnp(4, 0)
  b5=btn(5)

  if (b0 or b1 or b2 or b3) then move_hero() end
  if (b4) then hero_fire() end

  if(hero.des > 0)then hero.des += 1 end
  if(hero.des >= 20)then this.sprite = 21 end
  if(hero.des >= 40)then this.sprite = 22 end
  if(hero.des >= 60)then gameover() end
end,
```

The button check code is now scoped to the hero which makes it a little more readable I guess.  Notice the **hero.des** check pretty much the same as the asteroid's.  When the hero reaches **0** the **gameover** function is called though:

```
function gameover()
  text = "game over man"
  if(btn(4) or btn(5))then run() end
end
```

This simple function sets the **text** printed to the screen then calls the **run** function when button 4 or 5 is pressed.  The **run** function essentially resets the cart.

Also, we'll add an **explode** function/method to the hero:

```
explode = function(this)
  this.sprite = 20
  this.des = 1
end,
```

Once again update the **_update** function to check if an **asteroid** has collided with the **hero**.  Inside the **foreach asteroids** loop add:

```
-- asteroid collids with hero?
if (collide(hero, asteroid) and asteroid.des == 0 and hero.des == 0) then
  sfx(01)
  asteroid.hp -= 1
  hero.hp -= 1

  -- knock it back a litte
  asteroid.position.y -= 2
  asteroid.position.x -= asteroid.dir + asteroid.dir

  if(asteroid.hp <= 0) then asteroid.explode(asteroid) end
  if(hero.hp <= 0) then hero.explode(hero) end
end
```

Pow, now asteroids can take out our plucky hero.

## Conclusion

It's fun to have a game over state and to blast some space rocks.  I'm pretty satisfied with the way things explode too.

I'm kind of going for the effect of things exploding in space like the movie [Gravity](https://youtu.be/vKW-Gd_S_xc?t=149), but you know in 8 bit.  

Feels good to make progress...

Party On!
