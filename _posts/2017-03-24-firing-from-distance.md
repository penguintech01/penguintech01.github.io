---
layout: post
section-type: post
title: Firing from distance with proxychains! 👀 🔫
category: tech
tags: [ 'redteam', 'kali', 'tor', 'proxychains' ]
---

I decided to start a [#redteam](/tags/redteam.html) series of posts, so here we are!
The setup we'll use? Just a [Kali linux](https://www.kali.org/) installation, since it has everything we'll ever need.
I suggest setting it up in a live USB with persistence (and encrypted), so you can have your workstation always with you.
You'll be able to stick your USB on any computer you find and fire up Kali.
If you're using a mac, follow [this](https://www.youtube.com/watch?v=mDRbTHCoj8U) guide.

So, what's your goal? To get access to a resource that you are not authorized to do so.
The caveat? The resource owner shouldn't discover that you accessed it!

Let's start by hiding your MAC address. Why hiding it? To avoid your physical unique machine identifier from being logged by your network provider (for example Starbucks's WiFi network). Why can you spoof this unique identifier? Because your network provider has no way to verify that your MAC address is the one you're claiming to be. In other words, it's like someone asking what's your name, without asking for a government issued document to verify that this is your real name! :smile:

Here's how to change your MAC address if you're using your WiFi:

<pre><code data-trim class="bash">
ifconfig wlan0 down
macchanger -r wlan0 # -r for asking for a random MAC
ifconfig wlan0 up
</code></pre>

If you're on ethernet, then you need to specify the eth0 interface instead of the wlan0.

I suggest you put these commands in your bash profile, so you never forget to run them :wink:

Next, let's hide your IP address. Your IP address can't be spoofed.
It's like expecting to receive a mail, while you gave a fake address to the sender.
What you can do instead, is to send the mail to another recipient and expect that recipient to send that mail to you.
That's how proxies work.
You redirect all your traffic through another server.
Let's use the proxychains tool to hide behind a proxy easily.

First, let's find our IP address:

<pre><code data-trim class="bash">
curl https://canhazip.com/  
193.71.106.208
</code></pre>

Now let's use proxychains to issue the same command.
By default proxychains uses the Tor network as a proxy, so let's start the Tor service:

<pre><code data-trim class="bash">
service tor start
</code></pre>

In order to use the proxychains, you just need to add it before the command you want to execute.
Normally you'll want to perform a network scan (e.g.), like this:

<pre><code data-trim class="bash">
proxychains nmap -sS 192.168.1.0/24
</code></pre>

But for practical reasons (actually seeing our IP address changing), let's GET the canhazip page through proxychains:

![proxychains](/img/posts/proxychains/proxychains-0.png)

You can verify that your IP is one of Tor's exit nodes by searching for it [here](https://check.torproject.org/exit-addresses).

Now, let's add one more proxy (located in Venezuela), after Tor in our chain:

<pre><code data-trim class="bash">
nano /etc/proxychains.conf
</code></pre>

![proxychains](/img/posts/proxychains/proxychains-1.png)

Of course you can chain as many proxies as you want, making it more and more difficult to trace you.

And now my favorite part, let's randomize the chain order!
Comment out the "strict_chain" option and uncomment the "random_chain":

![proxychains](/img/posts/proxychains/proxychains-2.png)

Now my traffic is going through either the three Tor nodes and then the proxy in Venezuela, or it will start from the proxy in Venezuela and then go through the three Tor nodes.

Have fun scanning!

Disclaimer: You should be poking only systems that you have explicit approval to do so.
