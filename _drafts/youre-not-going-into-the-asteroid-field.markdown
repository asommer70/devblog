---
title: "You're Not Going Into The Asteroid Field?"
---

## Space Rocks

Adding asteroids to the mix seems like a good idea.  They don't have bullets (hopefully) so they should be relatively easy to have float down the screen.  Then again maybe they'll come into the the scene from any angle... haven't really decided on that part yet.

I think using the same type of object with an update method and an "array" of objects as we used with the bullets/lasers should work for asteroids too.

## Setting the objects

## Updating Bullets

Adding the remove asteroid if off the screen line makes me realize something similar for bullets is needed too.

In the **update** function (or is it now a method???) inside the **bulletconstruct** function add this line after the *this.position.y** update line:

```
if(this.position.y<0)then del(bullets, this)end
```

## Conclusion
