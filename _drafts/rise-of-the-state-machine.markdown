---
title: "Rise Of The State Machine"
date:   2016-08-02 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: asteroids.png
---

## Refactor Fun

After reading more of the [Pico-8 Zine 2](https://sectordub.itch.io/pico-8-fanzine-2) it's crazy obvious that I should use a "Finite State Machine" to control the hero ship, enemies, etc.  I've been looking through the code of the iconic [P.A.T. Shooter](http://www.lexaloffle.com/bbs/?tid=1867) and it seems a state machine is used to keep track of everything.

It makes sense and it seems to allow for more "portable" functions.  As in hey this function is great I can totally use it pretty much as is in a whole other project.  Yay, me! Yay, us guys!  (at least that's what I want to say)

## Using init

The first thing to refactor, I guess, is the **global** variables declared at the top of the file.  They can be blasted into the **_init** function and everything should work the same:

```
function _init()
  hero = {
    x = 58,
    y = 100,
    sprite = 0,
  }

  bullets = {}
  asteroids = {}

  text = "welcome to rumpus blaster"
end
```
