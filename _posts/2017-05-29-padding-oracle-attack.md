---
layout: post
section-type: post
title: Recovering plaintexts with Padding Oracle Attacks 🔮
category: tech
tags: [ 'crypto', 'redteam' ]
---
[Last time]({% post_url 2017-05-21-length-extention-attack %}) we saw how to forge a valid signature by knowing
a signed message, its authentic signature and the length of the key that was used to sign it.
Today we'll see how to decrypt ciphertexts, without knowing the key that was used to encrypt the original plaintext.
The only mistake that needs to be made, is for a decryption module to leak whether the padding of the ciphertext that is decrypting, has a valid padding or not!
Yes, an innocent looking papercut like this, can make your crypto fall apart.

In this case, the decryption module that leaks this information is called the Padding Oracle.
The Padding Oracle Attack was initially published by [Vaudenay](http://www.iacr.org/cryptodb/archive/2002/EUROCRYPT/2850/2850.pdf) and it's a side-channel chosen-ciphertext attack that works against the Cipher Block Chaining (CBC) mode and the Public Key Cryptography Standards \#7 (PKCS7) padding scheme.
Side-channel attacks are those that are based on the implementation of a cryptosystem.
Chosen-ciphertext attacks on the other hand, are those that enable the adversary to submit chosen ciphertexts and decrypt them using a cryptosystem.
In order to understand how the attack works, we need first to understand how CBC and PKCS7 work.

### CBC

Encryption and decryption work with Block Ciphers in their core.
Imagine the Block Ciphers as black boxes, that get as an input a fixed length key, a fixed length block of plaintext/ciphertext and they spit out the corresponding block of ciphertext/plaintext.
Since these blackboxes have a fixed length input, we need to somehow combine them, so we can enable the encryption/decryption of arbitrary sized inputs.
This is what Block Cipher Modes are about with CBC being the most popular among them.

When encrypting a plaintext with a block cipher in CBC mode, then the plaintext input of each block is XOR'ed with the ciphertext output of the previous block cipher.
That way the slightest change in the plaintext input, will affect all the following blocks of its block, apart from the block itself.
In the case of the first block, then a random block (per plaintext encryption) called Initialization Vector is used to XOR the plaintext of the first block before it's encrypted.

Here's a visualization of the process:

![CBC](https://upload.wikimedia.org/wikipedia/commons/d/d3/Cbc_encryption.png)

In order to decrypt a ciphertext that was produced using CBC, you need to XOR the ciphertext of the previous block, with the output of the current block cipher.
That way you nullify the encryption's XOR operation of the previous' block cipher's ciphertext:

C<sub>i - 1</sub> XOR P<sub>i</sub> XOR C<sub>i - 1</sub> -> (C<sub>i - 1</sub> XOR C<sub>i - 1</sub>) XOR P<sub>i</sub> -> 0 XOR P<sub>i</sub> -> P<sub>i</sub>

![CBC](https://upload.wikimedia.org/wikipedia/commons/6/66/Cbc_decryption.png)

### PKCS7 padding

We need a padding scheme in order to construct inputs that have a length that is divisible by the block size, since the block ciphers operate strictly on blocks.
The PKCS7 padding is simple, the last N bytes are padded with the value N.
For example the padding of *"Hello, world"*, for a block size of 16 bytes will be four 4s appended at its end:

<pre><code data-trim class="bash">
#  H    e    l    l    o    ,         w    o    r    l    d    4    4    4    4
0x48 0x65 0x6c 0x6c 0x6f 0x2c 0x20 0x77 0x6f 0x72 0x6c 0x64 0x04 0x04 0x04 0x04
</code></pre>

### The vulnerable decryption

Now that we know what CBC and PKCS7 are, let's see the vulnerable Ruby code that encrypts and decrypts data using the Advanced Encryption Standard (AES) block cipher, which operates on blocks of 128 bits (or 16 bytes):

<pre><code data-trim class="ruby">
require 'openssl'

class PaddingOracle

  def encrypt(plaintext)

    cipher = OpenSSL::Cipher::AES.new(256, :CBC)
    cipher.encrypt
    @key = cipher.random_key
    iv = cipher.random_iv
    ciphertext = cipher.update(plaintext) + cipher.final

    return iv + ciphertext

  end

  def decrypt(ciphertext)

    decipher = OpenSSL::Cipher::AES.new(256, :CBC)
    decipher.decrypt
    decipher.key = @key
    decipher.iv = ciphertext[0..15]

    # The Oracle will leak if whether the padding is correct or not in the .final method
    plaintext = decipher.update(ciphertext[16..(ciphertext.length - 1)]) + decipher.final

    # No plaintext returned
  end

end
</code></pre>

The call to the final method in the decryption above, will also check if the padding of the result plaintext is valid, before removing it.
If the padding is not valid, then a *OpenSSL::Cipher::CipherError* will be thrown and this information will leak to the caller.
As a result, this is the information that we will use in order to use the decrypt method as the Padding Oracle.
By submitting ciphertexts that we construct to the Oracle, we'll manage to recover the plaintext, without knowing the key that was used to encrypt the ciphertext that we intercepted.

### The exploit

Let's imagine that we intercepted the following ciphertext that has the length of two blocks (32 bytes):

C<sub>0</sub> \| C<sub>1</sub>

Now let's construct a ciphertext C'<sub>0</sub> like this:

C'<sub>0</sub> = C<sub>0</sub> XOR 00000001 XOR 0000000X

Where X is a random byte between 0 and 255.
Now let's submit C'<sub>0</sub> | C<sub>1</sub> to the Oracle and let's see what will compute:

C'<sub>0</sub> XOR D(C<sub>1</sub>) -> <br/>
C<sub>0</sub> XOR 00000001 XOR 0000000X XOR (P<sub>1</sub> XOR C<sub>0</sub>) -> <br/>
(C<sub>0</sub> XOR C<sub>0</sub>) XOR 00000001 XOR 0000000X XOR P<sub>1</sub> -> <br/>
0 XOR 00000001 XOR 0000000X XOR P<sub>1</sub> -> <br/>
00000001 XOR 0000000X XOR P<sub>1</sub>

Let's assume that *X* is the correct guess of the last byte of P<sub>1</sub>, what will happen in this case?
The last byte of P<sub>1</sub> will be nullified by the XOR operation and 1 will end up in the end plaintext.
Then the end plaintext will be a valid PKCS7 padding and the Oracle will not throw.
On the other hand, if *X* doesn't match the last byte of P<sub>1</sub>, then the padding of the computed plaintext will not be valid, and the Oracle will throw!

We have successfully recovered the last byte of C<sub>1</sub>, how can we continue to the next byte?
By simply computing the following C<sub>0</sub> like this:

C'<sub>0</sub> = C<sub>0</sub> XOR 00000022 XOR 000000YX

Where *Y* is again a value between 0 and 255.
Now by submitting C'<sub>0</sub> | C<sub>1</sub> to the Oracle, we'll get the same behavior as before and at some point guess the correct value of *Y*.
Like this we can continue and recover all the bytes of the block and of course this can be applied for every block of the ciphertext, except the first one.
But, the first block is (usually) the Initialization Vector so we don't even need to recover it :smile:

Here is the Ruby code that intercepts a ciphertext and then performs the attack on the *PaddingOracle* that we saw earlier:

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
