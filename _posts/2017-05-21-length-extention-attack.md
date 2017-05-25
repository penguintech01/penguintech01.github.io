---
layout: post
section-type: post
title: Forging signatures with length extension attacks
category: tech
tags: [ 'crypto', 'redteam' ]
---
Digital signatures are used in the digital world in order to prove the authenticity and integrity of a message.
In other words it's the way to prove that you sent a piece of data and that the data was not tampered.
Today we'll see what can go wrong when this scheme is not designed correctly.

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
The padding scheme is to append a *1* bit right after the message, then *0*s and the length of the original message, until the message becomes divisible by 512.
For example the message *"Hello world"* is padded to:

<pre><code data-trim class="bash">
# H    e    l    l    o         w    o
0x48 0x65 0x6c 0x6c 0x6f 0x20 0x77 0x6f
#  r    l    d   1
0x72 0x6c 0x64 0x80 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x58
                                   # 88
</code></pre>

After the padding pre-processing is done, we enter the stage of compression.
The compression is a compression function *f*, applied on each 256 bits block of the message and the previous result for the compression function.
For the first round, a fixed value, called Initialization Vector, is used as the output of the "previous" compression function.
When the compression is performed on all the blocks of the message, then the result is the digest of the hash function:

![merkle](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ed/Merkle-Damgard_hash_big.svg/800px-Merkle-Damgard_hash_big.svg.png)

So given:

 1. the signing scheme that we mentioned earlier,
 2. the length of the secret used,
 3. the message that is signed,
 4. its digest and
 5. a hash function that behaves like the one we described above,

what prevents us from forging an authentic signature by continuing the compression rounds on an arbitrary message that we will append on the known message?

Nothing. And this is what length extension attacks are about.

Let's assume we have the authentic signature of the message *"message"* when the secret *"secret"* is applied:

<pre><code data-trim class="bash">
SHA256('secret' | 'message') -> '33dd93031495b1e73b345ef5b7f494146d6c361908b4f2ad9cf7bbd35cffaa26'
</code></pre>

Our goal is to construct a new signature on a message that is appended on the 'message' string, without knowing the secret that was used to sign the message.
For that we'll need to construct a version of SHA-256 that allows to inject the Initialization Vector and the length of the input message:

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
    # Padding and compression rounds of SHA-256
  end

</code></pre>

By calling the *digest* function, we get back the expected SHA-256 digest of the input.
But, by calling the *inner_digest* function with the intercepted digest as the initialization vector and the length of the known message the data that we want to append, we can continue the computations of the compression functions, just like if we were the one who knows the secret as well!

Let's compute the signature of *"messageforged"* by injecting the signature that we intercepted as the Initialization Vector of our first compression round:

<pre><code data-trim class="ruby">
input = 'forged'.force_encoding('US-ASCII')
puts 'Forged signature:       ' + SHA256.inner_digest(input, [0x33dd9303, 0x1495b1e7, 0x3b345ef5, 0xb7f49414, 0x6d6c3619, 0x08b4f2ad, 0x9cf7bbd3, 0x5cffaa26], 70)
# Forged signature:       f9f333d547088763f8767a241baae7b50532f95a5ad75071a8e2960bc430fd37
</code></pre>

Now we need to construct the message that its authentic signing will match the above forged signature, by computing the padding that would be applied by the original signer of the forged message:

<pre><code data-trim class="ruby">
input = 'message'.force_encoding('US-ASCII')

# Construct the padding so when our message is appended on the secret, then our 'forged' string is pushed to the next block message
length = (input.length + 6) * 8
input << 0x80
input << 0x00 while (input.size + 6) % 64 != 56
input += [length].pack('Q').reverse
input += 'forged'
</code></pre>

The input is:

<pre><code data-trim class="bash">
#  m    e    s    s    a    g    e   1
0x6d 0x65 0x73 0x73 0x61 0x67 0x65 0x80
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
#     104    f    o    r    g    e    d
0x00 0x68 0x66 0x6f 0x72 0x67 0x65 0x64
</code></pre>

And the authentic signing of this input is:

<pre><code data-trim class="bash">
SHA256('secret' | input) -> 'f9f333d547088763f8767a241baae7b50532f95a5ad75071a8e2960bc430fd37'
</code></pre>

Just like we computed!
Which means that our message will be authenticated and pass the integrity check without any problems :smile:

Here is the full source code:

<script src="https://gist.github.com/PanosSakkos/58fda8b16f12a4b52790b0011322d4c9.js"></script>

Happy forging!
