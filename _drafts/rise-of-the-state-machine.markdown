---
title: "Rise Of The State Machine"
date:   2016-08-02 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: update_function.png
bbcode: true
---

## Refactor Fun

After reading more of the [Pico-8 Zine 2](https://sectordub.itch.io/pico-8-fanzine-2) it's crazy obvious that I should use a "Finite State Machine" to control the hero ship, enemies, etc.  I've been looking through the code of the iconic [P.A.T. Shooter](http://www.lexaloffle.com/bbs/?tid=1867) and it seems a state machine is used to keep track of everything.

It makes sense and it seems to allow for more "portable" functions.  As in hey this function is great I can totally use it pretty much as is in a whole other project.  Yay, me! Yay, us guys!  (at least that's what I want to say)

<!--more-->

## Using init

The first thing to refactor, I guess, is the **global** variables declared at the top of the file.  They can be blasted into the **_init** function and everything should work the same:

```
function _init()
  hero = {
    x = 58,
    y = 100,
    sprite = 0,
    state = 0,
    pst = 0
  }

  bullets = {}
  asteroids = {}

  text = "welcome to rumpus blaster"
end
```

Notice the two new attributes **state** and **pst**.  The state will hold a number designating what the current state of the hero is: 0=idle, 1=moving, 2=firing.  The **pst** is for the *player state time* which will help track how long the player has been in the current state.  It will be incremented in **_update()**.

## Changing State and Updating Functions

Okay I lied, the **hero.state** and **hero.pst** won't be directly updated in **_update()**, but in the **move_hero** and **hero_fire** functions.  Following the guide add a **change_state()**:

```
function change_state(p, s)
  p.state = s
  p.pst = 0
end
```

The functions takes an object **p** and a number/state **s**.  The object's **state** and **pst** attributes are then updated.  Thinking to use this same function for enemies as well.

Then edit **move_hero** adding this line at the top:

```
  change_state(hero, 1)
```

And the same in **hero_fire**:

```
  change_state(hero, 2)
```

Finally, to be more state machine like (I think) update the **_update** function:

```
function _update()
  b0=btn(0)
  b1=btn(1)
  b2=btn(2)
  b3=btn(3)
  b4=btnp(4, 0)
  b5=btn(5)

  if (b0 or b1 or b2 or b3) then move_hero() end
  if (b4) then hero_fire() end

  foreach(bullets, function(obj)
      obj.update(obj)
  end)
end
```

## Conclusion

I think things are now in a better state (haaa).  Or at least some ground work has been laid to make it easier to manage multiple characters in the game at one time.  Also, might help determine when/what things are happening around the screen.

New territory for me so I'm sure I'll be learning a lot the more I work with it.

Party On!
