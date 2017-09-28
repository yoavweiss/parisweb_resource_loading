TechSummit
SLOW INTERNET!! - How is Web Performance Impacting Our Users? (and what
can we do about it)
As a CDN, we’re very good at passing bytes along from one end of the
planet to another as fast as possible. Unfortunately, that’s not always
enough to make sure users perceive the “internet” as fast when they are
loading their favorite web page. So, how do users perceive web
performance? What does a browser do when it loads a page? And what makes
a web page “fast”? 

In this talk, Yoav Weiss will go over the various processing modes that
a browser goes through when it loads web pages. He will explain the
connection between the various networking protocols that we all know and
love and web performance as perceived by the user.
He will talk about the different metrics we use when measuring the
performance of a page, and discuss how changes in the web’s content
layer are impacting those metrics and the user experience. Finally,
he’ll detail what the different Akamai web performance products are
doing to remedy the situation and make our customer sites faster. 

Edge
Faster Bytes is Not Always Enough - Why is The Web Slow? (and what can
we do about it)
Abstract
Moving bytes fast across the planet doesn't always lead to fast Web
experiences. So how do browsers load pages? And how do we measure web
site speed and make sure ours is fast? 
Yoav Weiss will discuss browser internals, web metrics, and common perf
issues and what's being done to resolve them. 

Description

CDNs are very good at passing bytes along from one end of the planet to
another as fast as possible. Unfortunately, that’s not always enough to
make sure users perceive their experience as fast when they are loading
your web site. So, how do users perceive web performance? What does a
browser do when it loads a page? And what makes a web page “fast”? 

In this talk, Yoav Weiss will go over the various processing modes that
a browser goes through when it loads web pages. He will explain the
connection between the various networking protocols and web performance
as perceived by the user.
He will talk about the different metrics we use when measuring the
performance of a page, and discuss how changes in the web’s content
layer are impacting those metrics and the user experience. Finally,
he’ll detail what the different Akamai web performance products are
doing to remedy the situation and make our customer sites faster.




Outline
What’s the Web? What’s a browser?
Web is slow == Internet is slow
Even if DNS/SureRoute/<your favorite protocol> works fine
How browsers work
Components
Critical rendering path
Major events - DCL, onload
Waterfalls
Performance metrics
Onload, DCL
SpeedIndex
First paint
User timing
Hero element timing
Time to interactive
Long tasks
Input event timing
“Modern Web”
JS as the bottleneck
Loss of traditional metrics and need to create new ones in RUM
Third party content
How we’re making things faster
FEO
A2
3PO
mPulse
CriticalJS

Script
Bla bla bla statment
I’m kinda old. When I first started in the industry, I was working on a
web optimization proxy, as one does. My very first task was to build a
proprietary SPDY-like client-server protocol in order to be able to send
multiple requests and receive multiple responses at once over a single
connection. My second task was to integrate an image compression library
into the server. The image compression server was usually getting 2x
size reduction on the images. And network bandwidth at the time was a
serious issue, with top-notch mobile networks clocking at a cool
33kbps!! But, we soon noticed something odd. Even with those awful
speeds, a 2x reduction in image sizes did not give us similar
performance improvements, unless they were deployed at the same time as
the SPDY-like client-server protocol (which could only be deployed for
users that actively installed the client, which was very few of them).
The less technical folks were baffled by that. If we’re cutting down the
bytes we’re sending down by a factor of 2, why aren’t we seeing 2x
performance improvements???
The answer to that, as you probably guessed, is latency.
Sure the fancy networks at the time only delivered 33Kbps, but they also
did that with round trip times of 700ms, making the page’s loading
performance dominated by the latency. Requests in HTTP/1.0 and HTTP/1.1
were going out over multiple connections (2 connections per host for 1.1
at the time), but each connection could handle a single request at a
time, which meant that if you have to bring in 50 images to the page, it
would take you at a minimum 25 RTTs (even if all your images fit in a
single packet), or 17.5 seconds!!
But, that was a long time ago, right?
The industry has moved on, and those problems are all gone, *right*?

Well, we have advanced a lot in the last… almost 20 years. I’m really
old...
But the Web is still weird. We have newer protocols such as HTTP2 and
QUIC, we have improved image formats, and we have significantly faster
computers.
But we also have slower computers (pull out phone)
And content on the web has evolved significantly over the last 20 years.
From simple text sites with images and an occasional JS based form
validation, it turned into a full fledged application development
platform, and one where you *download your entire app every time you
want to use it*.

But first things first, let’s start with some definitions.
Some people consider the words “web” and “internet” interchangeable. But
they have very distinct meanings.
As our friends at Google tell us, the internet is a global computer
network providing a variety of information and communication facilities,
consisting of interconnected networks using standardized communication
protocols.
The web however, is “a network of fine threads constructed by a spider
from flui..” no
The web is a network of interlinked documents, identified by their URLs,
and accessible via the internet. In other words, it’s a network of
documents, applications and content that is delivered over a network of
computers, the internet.
A browser is a computer programs that is used to display HTML in order
to navigate the web.
So if we’d take the route of a metaphor, the internet is the roads, the
web is a network of interconnected attractions on those roads, URLs is
the road signs (stretching it, I know) and the browser is the vehicle we
can use to move from one location to the next.

But many people fail to make a distinction. Your users fails to make
that distinction, and often when they download a series of slow web
sites, their reaction would be “the internet is slow”.
They perceive slow networks the same way they perceive slow websites -
as a blank page that’s staring them in the face for seconds on end.
So if you are an internet provider, slow web sites mean that people will
think the service you provide is bad, even if the network you provide
has excellent bandwidth and latency characteristics. And if you’re a CDN
provider, “shorter roads” in our metaphor won’t necessarily help the end
user a whole lot if the slowest component is actually the “attraction”
itself.

So how does the attraction work. How do browsers load web pages?
TODO: a less shitty metaphor


<How browsers work bit>
(Also, mention the 3 seconds goal)

So, what are the problems with that:

* Discovery

* Bloat

* Server side processing

* Out of control third parties

* Client side processing

Let's go over these problems in detail and see how we can reduce the
pain they cause.

# Server side processing
Historically, before the seminal web performance work by Steve Souders,
server side processing was considered *the* problem in web performance,
and it was what people were optimizing for. While Steve showed that
front end performance is responsible for 80% of the time people are
waiting for web pages, 20% is nothing to scoff at.
So server side performance is still very much something to be concerned
with.
At Akamai we see median server side performance times (or TTFB between the edge server and the origin server) of ~450ms, which
is a lot, especially if you consider the 3 seconds performance budget.

Early advice on server side performance had the advice to "flush early"
- you don't have to wait for your DB in order to let the browser start
  processing your most-probably-static HTML head and start fetching some
resources there. Only that this is often not the case - if your DB calls
can fail, they can impact your entire document (e.g. respond with a
completely different 404 page vs. your 200 page), and it's not always
trivial to "switch" pages on the user after an error has happened (and
without reloading the whole thing).

# Discovery
_show a bandwidth graph with holes in it_
Because of the way browsers work, because the browser has to download
the HTML, in order to find subresources to download, which in turn,
especially CSS and JS download more resources that they depend on, a lot
of the resources the browser needs to download are discovered at a late
phase.
Fonts and background images are notorious in that sense, since the
browser has to download all the CSS, and process the DOM in order to
find what's needed to be downloaded. But Javascript that loads resources
is just as guilty, as the entire script has to be processed before the
resource download can start.
And this creates a dependency tree that's deep and contains many nodes,
each layer in that tree depends upon the downloading and processing of
the layer above it.
Now some resources are better at this than others: HTML and SVG can be
processed as they come, so a resource dependency in their first 100
bytes doesn't need to wait until the entire resource is downloaded
before the download can start.
But, CSS and JS are not the same. They have to be processed in their
entirety (and for CSS, all the blocking CSS has to be downloaed and
processed) before any resource download can start.
And that's not a bug, it's inherent to the way these formats work.

And there's inherent tension here between flattenning the dependency
tree which would help the browser load the page earlier, and the fact
that humans write websites, and for the humans, it's often better to
have smaller modules, which they can reason about without looking at the
larger picture.
So we have the old-fashioned CSS `@import`s and the newly fangled JS modules. They make
writing and managing code dependencies significantly easier. They also
make it harder for the browser, adding another layer to that dependency
tree.

One solution for that tension is build scripts that take your
dependencies and mush them into a single blob.

That can be better, but:
* You have to parse the whole thing before anything is requested, so
  making a large unified resource can sometimes turn up to be slower.
* You're messing up the caching granularity

What are the solutions to that problem:
* H2 push can be used to push critical resources before the browser
  discovers them and realizes that it needs them.
* Link rel preload can be used to preload critical


What's the difference between preload and push?
<bring in slide from other talk>

Does push and preload solve all discoverability problems?
Not really, sometimes actual JS needs to run on the client in order to
figure out what resource the browser will need
<show code that randomally pick either img>

But as web developers, we can try to limit that variablity and make sure
we push and preload everything that's needed.

Aside on preload and priorities.

Aside on early hints & server side delays.

# Bloat
Traditionally we have been building web sites for desktops. 
Also traditinally, we compressed our images to avoid that kind of
experience
<autoplaying video of an image downloading slowly>
As broadband got better, we compressed our images less and less (HTTP
archive graph), and added more of them. And for our mobile version of
the site, we kept it slim.
But as more and more devices came to the market, each with its own
viewport dimensions, building a site version became simply not feasible. 
As a result, responsive design took over web design by storm, but in its
most naive version, we kept sending the same images we needed for our
high-end, high-bandwidth user experience to all devices.

The web performance community formed the Responsive Images Community
Group, and after a long while and numerous arguments, we managed to
convince browser vendors that we need to a way to specify multiple
images for browsers to download, based on their device dimensions,
design and the browser's environment.

And we came up with `<picture>`, srcset and sizes, specified them and
shipped them in browsers.

That, along with Client Hints, gives us the tools to manage image bloat
on the Web, and reduce it to a minimum, reduce its impact to a minimum.

So we made some progress with image bloat, but we're not there just yet.

There's also much progress on bloat in terms of better compression
algorithms, better image formats, etc.

Aside on shared dictionaries.

# Client side processing

But the problem with bloat is not just the bandwidth waste. These
devices (points to phone) are not just downloading resources on an often
flakier network than our desktops. They also often have less CPU power
to process larger assets and less memory to deal with the consequences.

And even if we solved bloat for images, there's still a lot of CSS and
JS bloat to go around...
CSS frameworks are great to develop to, but often mean that you're
downloading way more CSS than you actually need *as a blocking
resource*.
JS frameworks and libraries often have a similar impact only that they
also tend to peg down the mobile device's CPU as their JS is being
processed (show timeline with CPU).

Critical CSS build step (or automatic processing) can help, but it may
be hard configure and get in place.

Build steps can help (webpack, etc), but they're not always
applicable...

# Third parties
Everyone like to complain about third party performance...
It feels good to complain about that, cause by definition, it's someone
else that's doing a bad thing, and you're all good.


Feature plan
A2 nextgen
Preload + priorities
Smarter/more RUM data 1
Cache Digests
Early Hints
Third Parties
Akamized 3P rewrite (optimizely)
Certificate frame
3PM runtime reporting 1
3PM Service Worker
JS optimizations
Critical JS
JS framework integration
AMP integration
Image optimizations
Video comp. for images 2
Image lazy load
Client Hints 2
Universal caching
Service Worker
Protocols
H2
Shared dictionaries
QUIC


Well, some of them.
HTTP/1.0 which had 4 non-persistent connections gave way to HTTP/1.1
with 2, then 4, then 6 persistent connections, significantly improving
the parallelism problem.
SPDY and then HTTP/2 came along and added parallelism capabilities,
enabling the browser and the server to send multiple requests and
responses at the same time over a single connection.
But HTTP2 has not solved everything: Many sites are not hosted on a
single domain, but have several domains. They do this either because in
HTTP/1.1 that used to help them by increasing the parallelism (2
connections per host turn into overall 8 connections when you have 4
different hosts delivering your images).
They also do this because it used to be a best practice to deliver
static assets from a cookie-less domain, as cookies had a tendency to
blow out requests.
Those 2 concerns are no longer valid in HTTP/2, but people tend to take
a long to change thier existing sites, so when moving them over to H2
they don’t necessarily move them over to a single H2 connection. Which
leaves us with an internet full of H2 connections that are sent to
different domains. And that’s even before we talked about third parties.

 

