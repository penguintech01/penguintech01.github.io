---
layout: post
section-type: post
title: Prisma owns your photos
category: tech
tags: [ 'privacy' ]
---
Prisma is an app that applies filters on photos that got a lot of hype recently <small>(my profile picture actually was made with the app)</small>.
When I visited the app's [website](http://prisma-ai.com/) I noticed that they were stating that the filter
is applied online on their service, which means that it's actually not applied on the user's device.
So I thought "Great, they have all the photos that I have applied a filter on them", and I started reading their [privacy policy](http://prisma-ai.com/privacy.html).

The privacy policy, between others, states the following:

> We collect the following types of information.
> Information you provide us directly:
> User Content (e.g., photos and other materials) that you post through the Service. Communications between you and Prisma.

> We may also share certain information such as cookie data with third-party advertising partners. This information would allow third-party ad networks to, among other things, deliver targeted advertisements that they believe will be of most interest to you.

The conversation started with <small>(typos in the original mails are preserved)</small>:

<pre><code data-trim class="md">
From: Panos Sakkos <panos.sakkos@gmail.com>
To: privacy@prisma-ai.com
Date: Fri, Jul 15, 2016 at 10:47 PM
Subject: Question

Hello,

in your privacy policy the following is stated:

> We collect the following types of information.
> Information you provide us directly:
> User Content (e.g., photos and other materials) that you post through the Service.

Does this mean that your are storing the photos that are being sent to you for processing?

Thanks,
:panos
</code></pre>

I didn't get a reply back within 6 days so I added the mail account of the "Investor Relations", assuming that they care more about investors than user concerns about their privacy policy:

<pre><code data-trim class="md">
From: Panos Sakkos <panos.sakkos@gmail.com>
To: contact@prisma-ai.com, ir@prisma-ai.com
Date: Thu, 21 Jul 2016 10:45:17 +0200
Subject: Fwd: Question

Hello,

Is the privacy@prisma-ai.com account being monitored?
I've asked a very straight forward questions and still haven't got a reply
back.

Thanks,
:panos
</code></pre>

And got the following answer:

<pre><code data-trim class="md">
From: Alexey Moiseenkov <contact@prisma-ai.com>
To: Panos Sakkos <panos.sakkos@gmail.com>
Date: Thu, 21 Jul 2016 12:47:04 +0300
Subject: Re: Question

Hello! Sorry for a delay.

We stored photos encrypted for one day and then delete all of them. We are
also do not collect any information from your device. We need to store
photos for one day to keep sure that the client app will get the result.

Thank you for question and feel free to ask more.
</code></pre>

Which of course made me wonder about the ownership of the encryption key.
Just saying that my photos are encrypted, means nothing if Prisma owns the key, because that means that they can see my data and share it with third parties, which the privacy policy clearly states.

The reply was:

<pre><code data-trim class="md">
From: Panos Sakkos <panos.sakkos@gmail.com>
To: Alexey Moiseenkov <contact@prisma-ai.com>
Date: Thu, 21 Jul 2016 12:58:51 +0200
Subject: Re: Question

- ir@prisma-ai.com

> Hello! Sorry for a delay.

No worries :)

> We stored photos encrypted for one day and then delete all of them.

And who owns the encryption key?
Do you keep it or is it owned only by the user (meaning that is stored at
the device only).

> We need to store photos for one day to keep sure that the client app will
get the result.

I don't have any knowledge of your system, but from an outsider's
perspective it doesn't seem like this is necessary.
If the client doesn't get a response with the processed image, why can't it
just retry by uploading the photo again?
That way you can reduce your storage costs and avoid leaking user photos
(which would be terrible PR for your app).

Thanks,
:panos
</code></pre>

No reply back, I bumped the conversation 5 days later:

<pre><code data-trim class="md">
From: Panos Sakkos <panos.sakkos@gmail.com>
To: Alexey Moiseenkov <contact@prisma-ai.com>, privacy@prisma-ai.com
Date: Tue, 26 Jul 2016 14:57:29 +0200
Subject: Re: Question

+ privacy@prisma-ai.com

Alexey mentioned that you are storing the photos that are uploaded for one
day and that they are encrypted.
My question is how is the owner of the key?
Is it prisma or is the key being stored only on the user's device?

Thanks,
:panos
</code></pre>

No reply on that either.
If there is a reply in the future I will update the post, but bottom line,
Prisma is an app that creates a false perception of security because you naturally assume that the filter is applied locally on your device, but it's actually not.
That means that your photos are sent to the Prisma servers and Prisma owns them and can do whatever they state in their privacy policy that no one ever reads.

That means you should take that under consideration before deciding in which photos to apply your filter on.
