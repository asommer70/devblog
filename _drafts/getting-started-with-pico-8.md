---
title:  "Getting Started with Pico-8"
date:   2016-07-19 13:05:00
categories: pico-8 games
layout: post
image: me_pico-8.png
---

## Discovery, or The Future is in the Past

For a few months I've been chewing on the idea of developing a 8 bit style game to run on [Raspberry Pi](https://www.raspberrypi.org/) alongside old ROMS from major game developers.  I was thinking the idea was original, but after some reflection I realize I must have picked it up somewhere...

I ordered a [C.H.I.P](https://getchip.com/pages/chip) (well okay a couple of them) back in like November and read up on the [PocketCHIP](https://getchip.com/pages/pocketchip) about the same time.  While reading about PocketCHIP I read, and probably even watched some videos, about the [Pico-8](http://www.lexaloffle.com/pico-8.php) fantasy console.  I think that was my first exposure to Pico-8.

This is when I think I internalized the idea of a "brand new" pixelated "game engine" for developing games that run on low end hardware.

I am so glad that Zep and Lexaloffle did the hard work and developed such a cool platform for others to play in.

<!--more-->

## Learning Pico-8 Development

I started out watching [Youtube](https://www.youtube.com/results?search_query=pico-8) videos of people developing/modifying games with Pico-8.  One awesome feature of Pico-8 is that the editor, graphics creator, map editor, music, and sounds are all in the same interface.  It's bananas really!

From there I read the first issue of the [Pico-8 Zine](https://sectordub.itch.io/pico-8-fanzine-1) which has great tutorials on creating a paddle and other games.  It walks you through the code, creating sounds, and adding some sweet jams.  Reading through some [other](http://thenewstack.io/retro-game-pico-8-basics/) tutorials was also super helpful.

Once the foundation of how the coding, graphics, and sounds all fit together I started my own project.

## Rumpus Blaster

Being super new to game development I'm going to be basically copying, but hopefully adding some new features, graphics, etc, current games until I get the feel of things.  I think this might be a good way to learn anyway...

So for my first game I'll try a space shooter similar to Galaga, but with far fewer ships.  The [P.A.T. Shooter](http://www.lexaloffle.com/bbs/?tid=1867) cart is an inspiration and a high bar to shoot for.  There's a lot to like about this title and I found that playing it with a USB NES controller is a lot of fun.

## My Development Environment

Since I'm on a Mac the installation directory, or maybe it's more like the data directory, for Pico-8 is:

```
/Users/$USER/Library/Application Support/pico-8
```

Where **$USER** is the current username of the workstation and if you're on a Windows or Linux machine the path is slightly different.  See the [manual](http://www.lexaloffle.com/pico-8.php?page=manual) for the exact location for your platform.  

The **carts** subdirectory is where games are downloaded and where new games are stored while in development.  To make it easier to work with I created a *symlink* from the **/Users/$USER/Library/Application Support/pico-8/carts** directory to **~/work/carts**:

```
ln -s /Users/$USER/Library/Application Support/pico-8/carts ~/work/
```

That way I can do a simple ```cd work/carts``` from my home directory.

Next, I fired up the [Atom](https://atom.io/) editor to start creating/editing carts.  Since the **.p8** files are simple text you can easily change things up in your favorite editor.  Obviously this works best with the code part of the file, but I imagine if you're super brave you can change the sprite by editing the numbers by hand... I guess.

After getting access to my carts with in a more "professional" editor I installed the [language-pico8](https://github.com/keiyakins/language-pico8) package with:

```apm install language-pico8```

This gives syntax highlighting (based on the Lua language highlighting) and also shows a sort of preview of the sprite map which is very nice.

Finally, I created a [Git](https://git-scm.com/) repository to have my carts under version control:

```
git init
git add .
git commit -am "Initial import."
git push
```

At this point I put the files into a private Git repo, but will probably create a repo on Github in the near future.

## Workflow

I find the workflow is pretty quick to get used to.  Especially if you've done any mobile app development or web development.  Basically you edit code in the editor Alt+Tab to the Pico-8 console execute:

```
LOAD $CARTNAME
RUN
```

And your game fires up and you can see if things behave the way you expect.  Also, if there's a syntax error the RUN command exits and prints a helpful line number.

One sort of "gotcha" is when creating sprites inside the Pico-8 console you want to make sure that you've saved before Alt+Tabing back to the editor.  If you don't save in both the console and the external editor changes in one can overwrite changes in the other.  Which isn't always fun.  Guess that's what source code management is for though.


## Conclusion

I'll be honest, I've wanted to develop games since the first time I played Wolfenstein 3d circa 1989 (maybe 1988) on a neighbors computer.  He was way ahead of his time and I wished I had taken more advantage of having a neighbor with a computer... I think basketball and bicycles were more interesting at the time.

Getting to game development after learning web and some mobile development feels good though.  Not to mention understanding how operating systems work from a systems administration standpoint.

This will be a lot of fun...

Party On!
