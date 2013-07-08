---
layout: post
title: "Node Man's Land"
date: 2013-07-05 09:00
comments: true
categories: [node.js, javascript, conferences]
---

I went to NodeConf last week, and I wanted to share what I learned! This post is basically a brain dump of what I remember from each session and my overall impression of the conference.

But first, a little about the structure of NodeConf. When we arrived, everyone received a [map of the camp](http://i.imgur.com/GRoK0hw.png) and a randomly chosen, unique [schedule](http://i.imgur.com/28Ar19l.jpg) of times with numbers that correspond to locations on the map. The idea was that for each time slot, you’d go to the session indicated by the number on your schedule. In each session, there were about 30 attendees and 2-3 presenters. The sessions were very interactive; almost all of them included coding workshops of some sort. Each session only lasted 50-55 minutes, but considering the amount of information covered, I found them amazingly rewarding and useful.

Read more about the format of NodeConf on the [github repo](https://github.com/mikeal/nodeconf2013#format).

## Sessions

<br/>
**Contributing to Node Core** ([@izs](https://twitter.com/izs), [@piscisaureus](https://twitter.com/piscisaureus))

In this session, Isaac explained that you don’t have to be a superstar C++ or low level OS developer to contribute to node. He went over how to write good tests and a very brief overview of node’s structure. We also learned how to add methods to core node modules using C++ bindings and how to expose them in JS.

My [commit](https://github.com/camupod/node/commit/41d028b2bc7184af10de5ae3087dfc311e99e068) from the workshop:

* added simple hello world C++ binding to net module, including a simple test case (see: lib/net.js, test/simple/test-net-hello-world.js, src/tcp_wrap.cc, src/tcp_wrap.h)
* added json method (with pretty print) to responses in http module (see: lib/_http_outgoing.js)

*Takeaway: it’s super easy to contribute to node core! Do it!*

<br/>
**Web Services** ([@eranhammer](https://twitter.com/eranhammer), [@mcantelon](https://twitter.com/mcantelon))

From what I gathered, the web services talk seemed basically like a comparison of hapi.js and express.js, two popular node web frameworks. It started out with a brief overview of web services in node, then continued on about how to use express and hapi and the differences between them. There was not too much of an interactive portion to this talk, though we were provided some data and code samples to play around with on our own time.

*Takeaway: express offers a very general framework with a lot of 3rd party middleware available, where hapi is more opinionated and offers more built in stuff. Basic node can do web, but frameworks are more powerful.*

<br/>
**Stream Adventure!** ([@substack](https://twitter.com/substack), [@maxogden](https://twitter.com/maxogden))

In the streams session, we played\* an awesome [stream-adventure](https://npmjs.org/package/stream-adventure) game\* created by [@substack](https://twitter.com/substack), which taught us a ton about how streams work, various use cases, and different modules that work really well with and implement streams. You should play\* it! This was one of my favorite sessions.

<sup>\*term used loosely</sup>

*Takeaway: streams are awesome and we should use them for everything always.*

<br/>
**Distributed Chat** ([@raynos](https://twitter.com/Raynos), [@dominictarr](https://twitter.com/dominictarr))

In this session, we built a distributed chat system where every client is also a server. We incrementally added functionality throughout the session (you can find the steps here: [https://github.com/Raynos/distributed](https://github.com/Raynos/distributed)). The session was awesome, but went very fast, so it was a little hard to follow. If you find this topic interesting, you should also check out Dominic Tarr’s [scuttlebutt](https://github.com/dominictarr/scuttlebutt) module.

*Takeaway: holy crap, this is cool!*

<br/>
**Domains** ([@domenic](https://twitter.com/domenic), [@othiym23](https://twitter.com/othiym23))

In the domains session, we implemented domains for various types of (crashy) servers. If I understood everything correctly, this is how we used domains:

* a simple http server that crashes at various endpoints, which are each wrapped in a domain
* domainified express middleware that uses the default error handler where the domain runs at the beginning of middleware chain, which basically wraps all requests in a domain
* domains are created explicitly in each route, then middleware (to be placed at the end of the middleware chain) is used as a wrapper around the express error handler, which emits errors to the active domain if it exists
* proof of work server with domains... this one kinda lost me ¯\\_(ツ)_/¯

There was a bit about domain.intercept() as well, which is used to reduce boilerplate error passing in callbacks. One *very *important thing to note about domains: [don’t ignore errors](http://nodejs.org/api/domain.html#domain_warning_don_t_ignore_errors). Errors can leave the process in an undefined state, so it’s better to use domains to capture the error for the users’ benefit (eg. send a useful error message or do a graceful shutdown), and then restart the app.

*Takeaway: domains are like try/catch that works in async environments and we should use them... carefully.*

<br/>
**Node Bots** ([@rwaldron](https://twitter.com/rwaldron), [@rockbot](https://twitter.com/rockbot), [@tmpvar](https://twitter.com/tmpvar))

This session was basically like: "here is a bunch of awesome robot hardware... go build something." I built a... ravebot? It was basically a multi-color LED that pulsed various colors at different intervals. *Cut me some slack, it was dark in the room, and I don’t know how to breadboard! *Man I could have so much fun with this stuff. I have an Arduino from JSConf... time to go shopping for parts!

*Takeaway: you can use node to (really, ridiculously easily) control an Arduino!*

<br/>
**Node Copters** ([@nexxylove](https://twitter.com/nexxylove), [@s5fs](https://twitter.com/s5fs))

The node copters session was very similar to the node bots session: make it do something cool. I attempted to build a browser-based controller and realtime video feed with web sockets, but didn’t have enough time (it looks like at least one person beat me to it anyways: [https://github.com/functino/drone-browser](https://github.com/functino/drone-browser)). I’m going to explore this a little more with the two AR Drones we have from the Crocodoc office.

Side note: [@jonmarkgo](https://twitter.com/jonmarkgo) and [@swiftalphaone](https://twitter.com/SwiftAlphaOne) were somehow able to create a tool that takes over all drones in wifi range. It was amazing, and you should watch it right now: [https://www.dropbox.com/s/onsiqfxk8n12qvv/dronevirus.MOV](https://www.dropbox.com/s/onsiqfxk8n12qvv/dronevirus.MOV).

*Takeaway: what’s next, node boats?*

<br/>
**DTracing Node** ([@mrbruning](https://twitter.com/mrbruning))

In this session we learned about using DTrace with your node apps. DTrace is a dynamic tracing framework intended to be used in (and not interfere with) production environments. We connected to a SmartOS VM and ran DTrace against various running apps. The session was a bit over my head and complicated, but I think it could be very useful when debugging performance issues in production node apps.

*Takeaway: you can’t use it on linux...*

## Materials

If you want to download the session materials and try stuff out, here’s the instructions:

1. install the latest stable (0.10.x) release of node.js from http://www.nodejs.org/
2. npm install nodeconf2013
3. login to GitHub and fork https://github.com/joyent/node and follow the instructions to clone it locally
4. verify that you can build node locally by running  ./configure && make

Hopefully some of this will be useful to someone. Please feel free to contact me with questions, but I definitely encourage you to contact the presenters directly–they are very helpful!

## Everything Else

Aside from an amazing lineup of presenters, there was a lot of other things about NodeConf that made it a very special experience. The remote and beautiful location, Walker Creek Ranch in Marin County, CA was the perfect place for a bunch of developers to get together for a few days and geek out. I stayed in a bunkhouse with several other developers, ate great food (not the mediocre camp food I remember from summer camp as a child), went hiking in the hills and swimming in the pond, and enjoyed great craft beer, music, star gazing, and conversation with the other campers in the evenings. After the evening annoucements and presentations, we even got to hear some amazing original songs by Paul Campbell ([http://vimeo.com/69667425](http://vimeo.com/69667425)).

I loved the fact that I didn’t know what session I was going to next and how small and interactive the sessions were. The structure of the sessions made it easier to socialize and collaborate, offering everyone a chance to come out of their cabins and tents.

Overall, I really enjoyed the conference, and I hope other conferences follow NodeConf’s lead. I think I learned more, coded more, met more awesome people, and had more fun at NodeConf than any other conference I’ve been to in the past. Huge thanks to [@mikeal](https://twitter.com/mikeal) for putting on such an amazing conference!
