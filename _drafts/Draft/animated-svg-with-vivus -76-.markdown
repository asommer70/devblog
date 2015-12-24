# Animated SVG with Vivus

## Viva la Vivus

I’ve been meaning to work on my portfolio site for a while and I’ve finally found some time to put together some assets and elements I’d like to use.  I guess vacations are good for that kind of thing…

One element I’d like to try out is an animated SVG of the logo, or at least the logo I’m thinking of using.  I think it’ll add something cool to the site and shouldn’t be that hard with all the great SVG animation libraries out there.

The one I’ve chosen to test out first is the [Vivus](https://github.com/maxwellito/vivus) library.

## Setting up your SVG

It’s best to read the documentation thoroughly before blasting into things, but that isn’t what I did.  Instead I tried out  a super complicated SVG that had a bunch of paths and transforms in it.  I then wondered why the thing wasn’t animating.

According to the [Principals](https://github.com/maxwellito/vivus#principles) section you need to set the fill to none and you can only use certain type of shapes.  Vivus is great, but it’s not a “full featured” animate every darn thing about an SVG library (there are other libraries out there that do a great job of that).  Like the site says Vivus is about animating the outline of an SVG to give it the “appearance” of being drawn.

So I take that to mean that the simpler the drawing the better.

Vivus also doesn’t work with text elements, but with the ideas in this [issue](https://github.com/maxwellito/vivus/issues/22) I converted the text elements.

In [Affinity Designer](https://affinity.serif.com/en-us/) I converted the text elements for letters into curves by selecting the element > clicking Layer in the top menu > then click convert to curves.  Pow, you now have an SVG with letters that will work with Vivus. 

## Installing Vivus

Since I’m just playing around at this point, I created a new directory for the project and installed Vivus via **npm**:

```
mkdir svg_test
cd svg_test
npm install vivus
```

## Index

I then created an **index.html** file with:

```
<html>
<head>
  <title>SVG Test</title>

  <script src='node_modules/vivus/dist/vivus.js'></script>
  <script src='https://cdnjs.cloudflare.com/ajax/libs/jquery/3.0.0-alpha1/jquery.min.js'></script>
</head>
<body>
  <svg id="letters" width="100%" height="100%" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" xml:space="preserve" style="fill-rule:evenodd;clip-rule:evenodd;stroke-linejoin:round;stroke-miterlimit:1.41421;">
      <g transform="matrix(17.2814,0,0,21.8821,-1.92048,872.76)">
          <path id="tletter" d="M27.45,-34.65L27.45,-30.45L16.55,-30.45L16.55,0L11.7,0L11.7,-30.45L0.8,-30.45L0.8,-34.65L27.45,-34.65Z" style="fill:none;stroke-width:0.63px;stroke:black;"/>
      </g>
      <g transform="matrix(16.9328,0,0,21.5021,373.723,877.905)">
          <path id="hletter" d="M1.733,-32.446L1.733,-35.547L14.502,-35.547L14.502,-32.446L10.522,-31.763L10.522,-19.482L27.759,-19.482L27.759,-31.763L23.779,-32.446L23.779,-35.547L36.548,-35.547L36.548,-32.446L32.568,-31.763L32.568,-3.76L36.548,-3.076L36.548,0L23.779,0L23.779,-3.076L27.759,-3.76L27.759,-15.698L10.522,-15.698L10.522,-3.76L14.502,-3.076L14.502,0L1.733,0L1.733,-3.076L5.713,-3.76L5.713,-31.763L1.733,-32.446Z" style="fill:none;stroke-width:0.65px;stroke:black;"/>
      </g>
  </svg>

  <script>
    new Vivus('letters', {duration: 200}, function() {
      $('#tletter').attr('style', 'fill:black')
      $('#hletter').attr('style', 'fill:black')
    });
  </script>
</body>
</html>
```

So this file is pretty straight forward.  I’m importing the **vivus.js** script directly from the *node_modules* directory and also including jQuery from a CDN.

The SVG is inline with the HTML, cause it was just easier.  To get the SVG text I opened the file in Atom and copied the SVG element skipping the first two doc type lines.

The magic of this file is in the bottom script element.  We’re calling the **Vivus** function and setting up some options.  I forgot to mention that after copying the SVG element from the original file I added some **id** attributes to the **svg** element and **path** elements.  

The Vivus function uses the **id** attribute of the **svg** element to start the animation, then in the call back function I’m grabbing the **path** elements using jQuery and adding a **style** attribute with a **fill:black** value.

This will add a fill to the SVG after the animation is complete.

## Conclusion

Virus is pretty awesome and very small so I can see the appeal in using it.  I think it will work great for my intended use, but I guess we’ll have to see.  Doing these kind of little experiments is pretty fun.

I think that I’m going to start including some JavaScript posts as well as Rails posts now.  I’m running out of ideas for Rails posts, or they’re coming in shorter bursts now.  I have a list of JavaScript post ideas so we’ll see how far I can get with that.

Party On!