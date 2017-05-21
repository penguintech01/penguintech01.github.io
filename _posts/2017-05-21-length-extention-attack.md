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
In order to generate the signature, you need to apply the mathematical function on the secret and the data to generate the signature which later will be shared along with the transmitted message.
When the message and the signature are received, then the receiver will perform the same computation and then compare the result signature with the transmitted one.
If this signature matches the one that was received, then she knows that the message is coming from you and that the data is not tampered.

The wrong way to do the above is by using a mathematical function based on [Merkle–Damgård construction](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction), like MD5, SHA1 and SHA2 in the following way:

<pre><code data-trim class="bash">
H(secret | message)
</code></pre>

This scheme is vulnerable to length extension attacks, and this practically means that when a message, the length of the secret used to sign it and the result digest are known, then a signature of a message that is appended to the original message can be constructed, without knowing the secret!
To understand how the attack works, we need first to discuss how the Merkle–Damgård construction works.

Initially the message is padded, in order to be divisible by a specific value, i.e. in the case of SHA-256 this value is 512.
The padding scheme is to append a *1* bit right after the message, then *0* s and the length of the original message, until the message becomes divisible by 512.
For example the message "Hello world" is padded to:

<pre><code data-trim class="bash">
# Message in hex-------------| 1 after the message
0x66 0x6f 0x72 0x67 0x65 0x64 0x80 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x02 0x30 # Length in bits
</code></pre>

After the padding pre-processing is done, we enter the stage of compression.
The compression is a compression function f applied on each 256 block of the message and the previous result for the compression function.
For the first round, a fixed value, called initialization vector, is used as the output of the "previous" compression function.
When the compression is performed on all the blocks of the message, then the result is the digest of the hash function:

![merkle](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ed/Merkle-Damgard_hash_big.svg/800px-Merkle-Damgard_hash_big.svg.png)

So, given the signing scheme that we mentioned earlier, the length of the secret used, the message that is signed, its digest the above hashing function, and a hash function that behaves like we described above, what prevents us from forging a message that will continue on the last compression function that was applied?

Nothing. And this is what length extension attacks are about.

Let's assume we have the authentic signature of the following message *message* when the secreat *secret* is applied:

<pre><code data-trim class="bash">
SHA256('secret' | 'message') -> '33dd93031495b1e73b345ef5b7f494146d6c361908b4f2ad9cf7bbd35cffaa26'
</code></pre>

Our goal is to construct a new signature on a message that is appended on the 'message' string, without knowing the secret that was used to sign the message.
For that we'll need to construct a version of SHA256 that allows to inject the initialization Vector and the length of the input message:

<pre><code data-trim class="ruby">
class SHA256

  def self.digest(input)

    input = input.force_encoding('US-ASCII')

    return self.inner_digest(
            input,
            # Original initialization vector of SHA-256
            [0x6a09e667, 0xbb67ae85, 0x3c6ef372, 0xa54ff53a, 0x510e527f, 0x9b05688c, 0x1f83d9ab, 0x5be0cd19],
            input.length)
  end
  
  def self.inner_digest(input, z, length)

  end

</code></pre>