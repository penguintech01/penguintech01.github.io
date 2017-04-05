---
layout: post
section-type: post
title: Creating a Tor Hidden Service
category: tech
tags: [ 'tor', 'privacy' ]
---

Let's talk about Tor!

I will not spend time on why Tor is changing and saving lives, because there
are already so many stories [reported](https://www.torproject.org/about/torusers.html.en)
about this. I will just say that Tor is a tool, just like a screwdriver is a tool.
And screwdrivers can be used for screwing screws, but also for hurting people.
Would you ban screwdrivers?

A [Tor hidden service](https://www.torproject.org/docs/hidden-services.html.en)
is any web service, accessible only from within Tor.
So any tricks that you know already about the web, they apply for the tor hidden services.
All you have to do, is to just replace the service url with the onion address.

First of all, you will need [Tor](https://www.torproject.org/docs/installguide.html.en).
Then you have to declare the path where you want to place your service's private and public keys
and its listening port in the torrc configuration file:

<pre><code data-trim class="bash">
HiddenServiceDir /home/username/hidden-service/
HiddenServicePort 80 127.0.0.1:4000
</code></pre>

Restart the tor service and you will find your service's onion address (public key) in HiddenServiceDir/hostname.

<small>Note the obvious, that if someone gets his hands on your private key, then he becomes you, your hidden service.</small>

Fire up a [Tor browser](https://www.torproject.org/projects/torbrowser.html.en), paste the onion address and you are live!

The only thing you should be worried except from [giving away](https://www.youtube.com/watch?v=7G1LjQSYM5Q)
 your Personal Identifiable Information due to human error, is that the speculations about NSA decrypting TLS were
[true](https://www.schneier.com/blog/archives/2015/05/the_logjam_and_.html).

P.S.: First time that you will access the service it will take a few minutes to respond.
