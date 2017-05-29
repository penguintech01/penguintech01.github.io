---
layout: post
section-type: post
title: Recovering plaintexts with Padding Oracle Attacks :crystal_ball:
category: tech
tags: [ 'redteam', 'crypto' ]
---
[Last time]({% post_url 2017-05-21-length-extention-attack %}) we saw how to forge a valid signature by knowing
a signed message, its authentic signature and the length of the key that was used to sign it.
Today we'll see how to decrypt ciphertexts, without knowing the key that was used to encrypt the original plaintext.
The only mistake that needs to be made, is for a decryption module to leak whether the padding of the ciphertext that is decrypting, has a valid padding or not!
Yes, an innocent looking papercut like this, can make your crypto fall apart :smile:

In this case, the decryption module that leaks this information is called the Padding Oracle
The Padding Oracle Attack was initially published by [Vaudenay](http://www.iacr.org/cryptodb/archive/2002/EUROCRYPT/2850/2850.pdf) and it's a side-channel chosen-ciphertext attack that works against the Cipher Block Chaining (CBC) mode and the PKCS7 padding scheme.
Side-channel attacks are the attacks that are based on the physical implementation of a cryptosystem.
Chosen-ciphertext attacks, are the enabled when an adversary has the ability to submit chosen ciphertext and decrypt them using a cryptosystem.
In order to understand how the attack works, we need first to understand how CBC and PKCS7 work.

## CBC

Encryption and decryption work with Block Ciphers in their core.
Imagine the Block Ciphers as black boxes, that get as an input a fixed length key and a fixed length block of plaintext/ciphertext and they spit out the corresponding ciphertext/plaintext block.
Since these blackboxes have a fixed length input, we need to somehow combine them, so we can enable the encryption/decryption of arbitrary sized inputs.
This is what Block Cipher Modes are about with CBC being the most popular of them.

When encrypting a plaintext with a block cipher in CBC mode, then the plaintext input of each block is XOR'ed with the ciphertext output of the previous block cipher.
That way the slightest change in the plaintext input, will affect all the following blocks on top of its block.
In the case of the first block, then a random block called Initialization Vector is used to XOR the plaintext of the first block before it's encrypted.

Here's a visualization of the process:

![CBC](https://upload.wikimedia.org/wikipedia/commons/d/d3/Cbc_encryption.png)

In order to decrypt a ciphertext that was produced using CBC, you need to XOR the ciphertext of the previous block, with the output of the current block cipher.
That way you nullify the encryption's XOR operation of the previous' block cipher's ciphertext:

<pre><code data-trim class="bash">
C XOR P XOR C -> 0 XOR P -> P
</code></pre>

![CBC](https://upload.wikimedia.org/wikipedia/commons/6/66/Cbc_decryption.png)

## PKCS7 padding

We need a padding scheme in order to construct inputs that end in a multiple of a block size, since the block ciphers operate strictly on a fixed sized input.
The PKCS7 padding is simple, the last N bytes are padded with the value N.
For example the padding of "Hello, world", for a block size of 16 bytes will be four 4s at its end:

<pre><code data-trim class="bash">
#  H    e    l    l    o    ,         w    o    r    l    d    4    4    4    4
0x48 0x65 0x6c 0x6c 0x6f 0x2c 0x20 0x77 0x6f 0x72 0x6c 0x64 0x04 0x04 0x04 0x04
</code></pre>

Now that we know what CBC and PKCS7 are, let's see the (vulnerable) Ruby code that encrypts and decrypts data using AES as the block cipher, which operates on inputs of 127 bits (or 16 bytes):

<pre><code data-trim class="ruby">
require 'openssl'

class PaddingOracle

  def initialize()

    # Hardcoded key and IV for demonstration purposes
    @key = '9f5421af4c33a92583ceec37efe0e058'
    @iv = '9f5421af4c33a925'

  end

  def encrypt(plaintext)

    cipher = OpenSSL::Cipher::AES.new(256, :CBC)
    cipher.encrypt
    cipher.key = @key
    cipher.iv = @iv
    ciphertext = cipher.update(plaintext) + cipher.final

    return @iv + ciphertext

  end

  def decrypt(ciphertext)

    decipher = OpenSSL::Cipher::AES.new(256, :CBC)
    decipher.decrypt
    decipher.key = @key
    decipher.iv = ciphertext[0..15]

    # The Oracle will leak if whether the padding is correct or not in the .final method
    plaintext = decipher.update(ciphertext[16..(ciphertext.length - 1)]) + decipher.final

    return plaintext

  end

end
</code></pre>

Let's use the above code to encrypt and then decrypt a message:

<pre><code data-trim class="ruby">
plaintext = 'This is a top secret message!!!'

oracle = PaddingOracle.new()
ciphertext = oracle.encrypt(plaintext)
plantext = oracle.decrypt(ciphertext)

puts plaintext
# This is a top secret message!!!
</code></pre>

Now let's see how the Padding Oracle Attack works

<pre><code data-trim class="ruby">
plaintext = 'This is a top secret message!!!'

oracle = PaddingOracle.new()
ciphertext = oracle.encrypt(plaintext)

recovered_plaintext = ''

to = ciphertext.length - 1
from = to - 31

while from >= 0

  target_blocks = ciphertext[from..to]

  i = 15
  padding = 0x01
  recovered_block = ''

  while i >= 0
    # For each byte of the block

    for c in 0x00..0xff
      # For each possible byte value

      chosen_ciphertext = target_blocks.dup

      # Set the bytes that we have already recovered in the block
      j = recovered_block.length - 1
      ii = 15

      while j >= 0

        chosen_ciphertext[ii] = (chosen_ciphertext.bytes[ii] ^ recovered_block.bytes[j] ^ padding).chr
        j -= 1
        ii -= 1

      end

      # Guess the i-th byte of the block
      chosen_ciphertext[i] = (chosen_ciphertext.bytes[i] ^ c ^ padding).chr

      begin
        # Ask the Oracle
        oracle.decrypt(chosen_ciphertext)

        # The Oracle said Yes, move to the next byte
        recovered_block = c.chr + recovered_block
        next

        rescue OpenSSL::Cipher::CipherError
          # The Oracle said No, try the next possible value of the byte

      end

    end

    i -= 1
    padding += 0x01

  end

  recovered_plaintext = recovered_block + recovered_plaintext

  # Move to the next block
  from -= 16
  to -= 16

end

puts recovered_plaintext
# This is a top secret message!!!
</code></pre>

Here is the full source code:

<script src="https://gist.github.com/PanosSakkos/02c225e4ebe6c596a7519ebead84091c.js"></script>

Happy decrypting!
