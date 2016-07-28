---
title: "Hitting Things With Lasers"
date:   2016-08-02 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: asteroids.png
bbcode: true
---

## A Hit, A Hit, A Very Palpable Hit



## Updating bullets

As the guide in [Pico-8 Zine #3](https://sectordub.itch.io/pico-8-fanzine-3) teaches the bullets (lasers in this case) need to have a **hitobx** to determine when they collide with something.  With the hero laser there are two "beams" so we'll need to add a hitbox for each one.  Update the **bulletconstruct** function with:

```
obj.hitbox1 = {x=0, y=0, w=1, h=8}
obj.hitbox2 = {x=8, y=8, w=1, h=8}
```
