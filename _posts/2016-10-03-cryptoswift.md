---
layout: post
section-type: post
title: Don't use CryptoSwift
category: tech
tags: [ 'crypto', 'opensource' ]
---

[CryptoSwift](https://github.com/krzyzanowskim/CryptoSwift)
is a Swift component that provides
cryptographic algorithms, implemented from scratch, and maintained by one person
[because he can](https://github.com/krzyzanowskim/CryptoSwift/issues/5).
What can go wrong? Like you guessed, a lot.

When you use cryptography it will be for securing information, which means that you can't
really afford bugs. So the rule is that you should be very careful
on which crypto-libraries to take a dependency on, since they should be maintained by
domain experts and being tested in the wild successfully and for a long period of time.

That said, I had to use PBKDF2 for a side-project in different languages <small>(Swift, Java and C#)</small>,
and while prototyping in the early days, I chose CryptoSwift <small>(for the Swift implementation)</small>
because at that point it was the easiest one to use and it was very popular on Github.
I created a mental backlog item for replacing it with something more reliable (like
openssl) before the release, but to be honest, I wasn't expecting any hassle till then.
That turned out to not be the case, and I quickly found that PBKDF2 <small>,a very simple RFC,</small>
was [not](https://github.com/krzyzanowskim/CryptoSwift/issues/270)
 implemented correctly. After a while, I also found that the derivation for 50,000 iterations
was taking 19.5 seconds, where the BouncyCastle and SpongyCastle equivalents were
finishing the computation in a few milliseconds. Yesterday I managed to replace the dependency
with Apple's CommonCrypto library and the derivation times got down to the ones that
 BouncyCastle and SpongyCastle were giving:

![cryptoswift](/img/posts/cryptoswift/cryptoswift.png)

<img alt="commoncrypto" src="/img/posts/cryptoswift/commoncrypto.png"/>

I don't regret using it for the prototype back then, since it bought me time, but
CryptoSwift is dangerous and it shouldn't be used in production environment.
Choose between openssl, CommonCrypto, BouncyCastle or an equivalent trustworthy library.
