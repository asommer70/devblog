---
title:  "Exhausting Exhaust"
date:   2016-07-26 13:05:00
categories: pico-8 games rumpusblaster
layout: post
image: hero_exhaust.png
bbcode: true
---

## Making Exhaust Look Good

Using a straight up sprite for an exhaust trail didn't look that cool to me.  Similar space shooter games have different ways of animating fire/exhaust coming out of a ship, but I really like the "sparkly" effect in games like [Cave Berries](http://www.lexaloffle.com/bbs/?tid=1834) when the eye laser hits something.  

Brief flashes of bright pixels are surprisingly satisfying...

## Using pset in exhaust()

The built in **pset** function is quite handy it allows you to set the color of any pixel identified by x, y coordinates.  Wonder if you could build an entire game with only using loops, ifs, and pset...

I created an **exhaust** function to determine the direction (based on button number) of the hero ship and adjust some trailing pixels with pset:

```
function exhaust(x, y, button)
  rnd_x = 0
  modx = 0
  mody = 0
  submod = 0
  len = 0

  if button == 2 then
    -- random color of red, yellow, or orange
    rnd_color = flr(rnd(3)) + 8

    -- adjust pixel location.
    rnd_x = 3
    mody = 8
    modx = 1
    submod = 2
    len = 30

    -- add some base fire.
    pset(x + rnd_x, y + mody, 10)
    pset(x + rnd_x + modx, y + mody + 1, 9)

  elseif button == 3 then
    -- random color of red, yellow, or orange
    rnd_color = flr(rnd(3)) + 8

    -- adjust pixel location.
    rnd_x = 3
    mody = -1
    modx = 1
    submod = 2
    len = 30

    -- add some base fire.
    pset(x + rnd_x, y + mody, 10)
    pset(x + rnd_x + modx, y + mody + 1, 9)

  elseif button == 0 then
    -- random color of dark_gray, light_gray, or white
    rnd_color = flr(rnd(3)) + 5

    -- adjust pixel location.
    rnd_x = 8
    mody = 4
    submod = -9
    len = 10
  elseif button == 1 then
    -- random color of dark_gray, light_gray, or white
    rnd_color = flr(rnd(3)) + 5

    -- adjust pixel location.
    mody = 4
    submod = 11
    len = 10
  end

  for i = 1,10 do
    rnd_y = flr(rnd(len))

    -- makes exhaust wide.
    if (i % 2 == 0) then rnd_x += modx end

    -- makes exhaust go up.
    if (button == 3) then rnd_y = -rnd_y end

    -- make exhaust come out the right side.
    if (button == 0) then rnd_y = 0 end
    if (button == 0) then rnd_x = flr(rnd(len)) end

    -- make exhaust come out the left side.
    if (button == 1) then rnd_y = 0 end
    if (button == 1) then rnd_x = flr(rnd(len)) end

    pset(x + rnd_x - submod, y + mody + rnd_y, rnd_color)
  end
end
```

This function is pretty long, and I'm sure there's a bunch of places to optimize, but it works.  At the top of the function I set some variables to determine the **x** and **y** of the pixel to set.  Then the number of the **button** parameter is checked.  

Inside each *if* statement the color is selected.  I shamelessly got the color function:

```
rnd_color = flr(rnd(3)) + 8
```

From the [Cave Berries](http://www.lexaloffle.com/bbs/?tid=1834) cart.  In the case above it will select a number from 1-3 then if you add 8 you get an index for the colors red, orange, and yellow.  Very slick!

The next statements determine where to position the exhaust pixels: top, bottom, left, or right by adding/subtracting from x,y.  Lastly, a few "base" exhaust pixels are added to make things a little beefy.

After the *ifs* is a for loop which adds random pixels in the desired location.  For top and bottom exhaust the **x** pixel is "waggled" some to make a wider spread.  

Conclusion

I'd like to refactor this function to set each needed variable before the loop. That way the button wouldn't have to be checked again.  

At least it feels like a lot of duplication calling if so many times.

Party On!
