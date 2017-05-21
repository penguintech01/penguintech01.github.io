---
layout: post
section-type: post
title: Forging signatures with length extension attacks
category: tech
tags: [ 'crypto', 'redteam' ]
---
Digital signatures are used in the digital world in order to prove the authenticity and integrity of a message.
In other words it's the way to prove that you sent a piece of data and that the data was not tampered
Today we'll see what can go wrong when this scheme is not implemented correctly.

Digital signatures operate on a secret key, which is shared across the parties that need the authentication scheme, the data that needs to be signed and a mathematical function.
In order to generate the signature, you need to apply the mathematical function on the secret and the data and in order to generate the signature which later will be shared along with the transmitted message.
When the message and the signature are received, then the receiver will perform the same computation and then compare the result signature with the transmitted one.
If this signature matches the one that was received, then she knows that the message is coming from you and that the data is not tampered.
