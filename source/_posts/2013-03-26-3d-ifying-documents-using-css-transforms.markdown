---
layout: post
title: "3D-ifying Documents Using CSS Transforms"
date: Tue Mar 26 2013 15:31:00 GMT-0700 (PDT)
comments: true
categories: [css, 3d, crocodoc, demos, javascript, svg, html, experiments]
---

**This article was originally posted on the [Crocodoc blog](http://blog.crocodoc.com/post/46369766700/3d-ifying-documents-using-css-transforms). I am reposting here for persistence.**

We recently launched a preview of Crocodoc’s newest document to HTML converter. If you haven’t checked it out yet, go play with our [preview](http://preview.crocodoc.com/) and see how we’re converting the pages of your documents to embeddable SVG and HTML.

What does the new converter mean to those of you building web applications using Crocodoc? Simple: your documents will load faster, look sharper, and be much easier to customize. Our preview page is full of interactive examples designed to help provide inspiration and showcase what is possible with the new Crocodoc: everything from a 3D page demo, showing off the many layers in a document, to a magnified view of an uploaded document, and a thumbnail that expands into a full-size inline document.

For this post, I’d like to focus on the 3D demo. I’ll explain how it was built, the various issues we ran into, and the workarounds we used to fix them.

*Note: the demos in this blog post require IE 9+ (preferably 10), Firefox, or any WebKit browser. If you’re on a mobile device, you might need to click the open in new page button to view the demos properly.*

Building the 3D Demo
--------------------

The Crocodoc 3D Demo separates a document into discrete layers, which you can rotate so that you can see how the document is constructed. Not only does it look pretty cool, but it gives some insight into the processing Crocodoc is doing on each document that we convert to HTML.

For the purposes of this blog post, I broke the demo down into three steps: building a basic proof of concept (without SVG), getting it to work in our target browsers, and finally, adding the SVG layers.

*Just want to play with some code? Fork the [3d-demo repo](https://github.com/crocodoc/3d-demo)!*

**Step One: Proof of Concept**

When starting the Preview project, the first step was to create a proof-of-concept for the 3D demo. The project needed to work on the target browsers (IE 9+, Firefox, WebKit, and most mobile browsers). I have worked with CSS 3D transforms in the past, but not to the extent that this project required, so it was time to do a bit of research! The WebKitCSSMatrix class that I discovered seemed like a good place to start. Fortunately, the proof-of-concept demo was actually quite simple to build using the WebKitCSSMatrix class. Let’s take a look at some code!


**_The Page_**

The Page class inserts layer divs into a jQuery-wrapped container element, and exposes a rotate(dx, dy, dz) method, which rotates the page in 3D space. Each layer div is positioned LAYER\_SPACING apart on the z axis. From here, it’s easy to take it a step further, and add mouse/touch controls for rotating the page.

##### [Demo \#1: The Page](http://lakenen.com/blog-demos/demo1.html)
<p><iframe class="demo" src="http://lakenen.com/blog-demos/demo1.html"></iframe></p>
{% gist 5251877 %}
   

**_Adding Transitions_**

In browsers that support CSS transitions, adding transitions is very simple, and done almost entirely in CSS. The JavaScript changes are mainly adding and removing CSS classes. We can also add a nice implode/explode effect.

##### [Demo \#2: Transitions](http://lakenen.com/blog-demos/demo2.html)
<p><iframe class="demo" src="http://lakenen.com/blog-demos/demo2.html"></iframe></p>
{% gist 5251958 %}
  

**Step Two: Browser Support**

Awesome, we have a working, nice-looking proof-of-concept! Now, how do we get it to work on all browsers? It turns out, currently the only two implementations of W3C CSSMatrix interface ([2D](http://dev.w3.org/csswg/css3-2d-transforms/#cssmatrix-interface) and [3D](http://dev.w3.org/csswg/css3-3d-transforms/#cssmatrix-interface)) are [WebKitCSSMatrix](http://developer.apple.com/library/safari/#documentation/AudioVideo/Reference/WebKitCSSMatrixClassReference/WebKitCSSMatrix/WebKitCSSMatrix.html "WebKitCSSMatrix Class Reference") (supported by at least Chrome, Safari, iOS, and Android) and [MSCSSMatrix](http://msdn.microsoft.com/en-us/library/windows/apps/hh453593.aspx "MSCSSMatrix Class Reference") (IE 10 only). What now? Some googling lead me to a [CSSMatrix shim](https://github.com/arian/CSSMatrix), which looked promising, so I gave it a go. Unfortunately, the shim was actually broken when I found it, but hey–a chance to contribute to a useful open-source project? Sign me up! Long story short, after several hours of poring through [Wikipedia articles](http://en.wikipedia.org/wiki/Rotation_matrix) and even the [WebKit source](http://www.opensource.apple.com/source/WebCore/WebCore-514/platform/graphics/transforms/TransformationMatrix.cpp), I finally got it working properly (for a browser-ready version to play with, check out the [browserified branch](https://github.com/camupod/CSSMatrix/tree/browserified) and add CSSMatrix.js to your page).

##### CSSMatrix Example
{% gist 5257218 %}
   

With Firefox support handled, it’s time to tackle IE 9. Even though IE 9 can’t render matrix3d values in CSS, we can compute them in JavaScript and truncate a few numbers… which is exactly what we did. And it looks way better than you might expect! Check out the demo below to compare how it looks in IE 9 (which we’ll call affine mode) vs proper 3D mode.

##### [Demo \#3: Affine Mode](http://lakenen.com/blog-demos/demo3.html)

<p><iframe class="demo" src="http://lakenen.com/blog-demos/demo3.html"></iframe></p>
*(NOTE: can’t tell the difference between the two modes? you’re probably using IE 9. No? It’s also possible that hardware acceleration is disabled on your machine, which you can check in **chrome://gpu/.**)*
{% gist 5252004 %}
   

IE 9 also doesn’t support CSS transitions, but I’m not going to go into detail about how we did that, because there are already a lot of jQuery plugins out there that make it pretty seamless. (Here are a few places to start: [jQuery.transition.js](https://github.com/louisremi/jquery.transition.js), [jQuery-Animate-Enhanced](https://github.com/benbarnett/jQuery-Animate-Enhanced).)

Now that the demo works well in all the browsers we’re targeting, we can just throw in the SVG file and we’re done! Right? Well, not exactly.
   

**Step Three: Make it Work with SVG**

After a bit of (failed) experimenting, I realized that, out of the box, SVG does not support CSS transforms with perspective. My solution was to load the SVG with AJAX (really CORS from AWS S3, which required yet [another IE shim](https://gist.github.com/camupod/5252086), and a cache-related hack for Chrome), fix linked assets (Crocodoc SVG links to assets relatively), apply a filter to split the SVG into several distinct SVG objects, and embed each object using HTML5 Inline SVG in a separate div elements. Voila, we have our layers!

##### [Demo \#4: SVG Layers](http://lakenen.com/blog-demos/demo4.html)
<p><iframe class="demo" src="http://lakenen.com/blog-demos/demo4.html"></iframe></p>
{% gist 5252136 %}
   

Since the SVG is now contained in HTML elements, we can apply CSS 3D transforms to the containers, and everything works swimmingly. Except in IE 9. Yep, IE 9 supports inline SVG, but it doesn’t support document.importNode(), which is necessary for creating the inline SVG in the first place. I found a [nice shim](http://stackoverflow.com/a/9883539/494954), which I [modified slightly](https://gist.github.com/camupod/5165619#file-importnode-js), because it was producing [some issues](http://stackoverflow.com/questions/14593520/ie9-importing-inline-svg-image-elements-broken) with namespaced attributes.

If you’re interested, also check out demos [5](http://lakenen.com/blog-demos/demo5.html), [6](http://lakenen.com/blog-demos/demo6.html), and [7](http://lakenen.com/blog-demos/demo7.html) (warning, you might need a fast machine).

*Note: I also ran into some very troubling issues with Firefox.  Rotating the 3D demo would reproducibly cause a kernel panic on my MBP (Retina, Mid 2012). I unfortunately haven’t been able to create a reduced test case for this, but I’ll be sure to post about it when I do. It has something to do with applying 3D transforms to elements (possibly specifically SVG) in a overflow:hidden container.*
   

Wrap Up
-------

It’s true that SVG is still in its infancy in HTML, but I think our preview page alone proves that it’s already possible to do some incredible stuff. I hope this post inspires some great applications built on the new Crocodoc.

In my next post, I’ll explain the other demos on our preview page, as well as a few other issues we ran into.
