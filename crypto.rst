Chapter 3:  Cryptographic Primitives
======================================

In the first two chapters we have seen that network security is a
multi-faceted problem. One of the major threats that must be addressed
is that data traversing a network is prone to being read or modified by
an attacker. The foundational technology for tackling such threats is
cryptography. So we begin our study of technical approaches to network
security with an overview of cryptography.

We introduce the concepts of cryptography-based security step by step.
The first step is the cryptographic algorithms—ciphers and cryptographic
hashes—that are introduced in this chapter. Such algorithms are not a solution in
themselves, but provide building blocks from which a solution can be
built. Cryptographic algorithms are parameterized by *keys*. The
distribution of keys is in itself a challenge which we tackle in a later
chapter.

Once we have a set of cryptographic algorithms and a way to distribute
keys, we are in a position to build protocols that enable secure
communication between participants. The final chapter describes several
complete security protocols and systems in current use.

3.1 Principles of Ciphers
---------------------------

Encryption transforms a message in such a way that it becomes
unintelligible to any party that does not have the secret of how to
reverse the transformation. The sender applies an *encryption*
function to the original *plaintext* message, resulting in a
*ciphertext* message that is sent over the network, as shown in
:numref:`Figure %s <fig-genericCrypto>`. The receiver applies a
*decryption* function—the inverse of the encryption function—to
recover the original plaintext. The ciphertext transmitted across the
network is unintelligible to any eavesdropper, assuming the
eavesdropper doesn’t have the means to perform the decryption
function. The transformation represented by an encryption function and
its corresponding decryption function is called a *cipher*.

.. _fig-genericCrypto:
.. figure:: figures/f08-01-9780123850591.png
   :width: 500px
   :align: center

   Secret-key encryption and decryption.

As we noted in the previous chapter, the principle of open design has
been proposed for security technologies since at
least 1975. Cryptography has a much longer history than that, however,
going back thousands of years. One of the leading cryptographers of
the 19th century, Auguste Kerckhoffs, stated in 1883 that
cryptographic system designs themselves should not be secret, but
should be parameterized by an easily changeable *key*; only the key
should need to be secret. This also follows the principle from Chapter 2 of minimizing secrets.

One reason for open design is that, if you were to depend on the
cipher being kept secret, then you would have to retire the cipher
(not just the keys) when you believe it to be no longer secret. This
means potentially frequent changes of cipher, which is problematic
since it takes a lot of work to develop (and establish the security
of) a new cipher. In the pre-computer era, ciphers were often
implemented by specialized hardware such as the Enigma machine; these
could not be easily replaced, but were re-keyed frequently. Today's
algorithms, while implemented in software, are the result of length
processes of development, testing, analysis and standardization; all
of this makes the expensive to replace.

Also, one of the best ways to know that a cipher is secure is to use
it for a long time—the longer it goes unbroken, the better the chance
that is is secure. As with security in general, proving that a
cryptographic algorithm cannot be broken is a bit of a negative goal,
and thus hard to do with complete confidence.  Sometimes we may be
able to prove that breaking a cipher is as hard as solving some
well-studied mathematical problem such as factoring large numbers.
Even then there may be problems of implementation that will only get
discovered by the scrutiny of those looking at and using the
algorithm. And in many cases, we just have to rely on the fact that
no-ne has yet found a viable way to break the cipher. Fortunately,
there are plenty of people who will try to break ciphers and who will
let it be widely known when they have succeeded.

Parameterizing a cipher with keys provides us with what is in effect a
very large family of ciphers; by switching keys, we are 
switching to another cipher in the family. It is common to limit the amount
of data that a *cryptanalyst* (code-breaker) can access before the key
changes. This provides the attacker with less ability to break the cipher
(for reasons discussed below) and limits the damage done if the code is
broken. 

The basic requirement for an encryption algorithm is that it turns
plaintext into ciphertext in such a way that only the intended
recipient—the holder of the decryption key—can recover the plaintext.
What this means is that encrypted messages cannot be read by people
who do not hold the key. When we say "cannot be read" we are going to
make some assumptions about the computational capabilities and time
available to the attacker; more on this in a moment.

It is important to realize that when a potential attacker receives a
piece of ciphertext, he may have more information at his disposal than
just the ciphertext itself. For example, he may know that the
plaintext was written in English, which means that the letter *e*
occurs more often in the plaintext that any other letter; the
frequency of many other letters and common letter combinations can
also be predicted. With simple ciphers, this information could greatly
simplify the task of finding the key. Similarly, the attacker may know
something about the likely contents of the message; for example, the
word “login” is likely to occur at the start of a remote login
session. Common headers appear at the start of HTTP messages. This may
enable a *known plaintext* attack, which has a much higher chance of
success than a *ciphertext only* attack. Even better is a *chosen
plaintext* attack, which may be enabled by feeding some information to
the sender that you know the sender is likely to transmit. 

The best cryptographic algorithms, therefore, can prevent the attacker
from deducing the key even when the individual knows both the
plaintext and the ciphertext. In the ideal case, the attacker has no
choice but to try all the possible keys—exhaustive, “brute-force”
search. If keys have *n* bits, then there are 2\ :sup:`n` possible
values for a key (each of the *n* bits could be either a zero or a
one).  An attacker could be so lucky as to try the correct value
immediately, or so unlucky as to try every incorrect value before
finally trying the correct value of the key, having tried all 2\
:sup:`n` possible values; the average number of guesses to discover
the correct value is halfway between those extremes, 2\ :sup:`n/2`.
This can be made computationally impractical by choosing a
sufficiently large key space and by making the operation of checking a
key reasonably costly. What makes this difficult is that computing
speeds keep increasing, making formerly infeasible computations
feasible. Furthermore, such brute force searches are easily
parallelized, meaning that an attacker can use general purpose GPUs
(GPGPUs) or other machines operating in parallel to speed up the
attack.

It turns out that it is not trivial to create cryptographic ciphers
that can be broken only by brute force. For example, the original DES
(data encryption standard) algorithm had a key of only 56 bits; when
it became clear that 56 bits was too small, triple DES was introduced, using three
rounds of DES each with its own key. It might seem that this 
increased the key size to 168 bits (:math:`3 \times 56`) but because
of the 3-round structure of triple DES, the attacker only has to
search a key space of 112 bits. This depends on something called a
"meet-in-the-middle attack". The details are not important here but it
illustrates why cryptographic algorithms need to be designed
carefully if they are not to contain surprising weaknesses.

Network security tends to focus on the security of data as it
moves through the network—that is, data that is vulnerable for only a
short period of time. In general, however, we should also consider the
possibility that data might be captured for later analysis, or that
some data might be stored in archives for tens of years. This
argues even more strongly for a generously large key size to prepare
for future computational advances.  However we do also need to
consider that larger keys tend to make encryption and decryption
slower.

.. sidebar on PQC

3.1.1 Block Ciphers
~~~~~~~~~~~~~~~~~~~~

Most ciphers in use today are *block ciphers*; they are defined to
take as input a plaintext block of a certain fixed size, typically 128
bits. Using a block cipher to encrypt each block independently—known
as *electronic codebook (ECB) mode* encryption—has the weakness that a
given plaintext block value will always result in the same ciphertext
block (as long as the key remains constant). Hence, recurring block
values in the plaintext are recognizable as such in the ciphertext,
making it much easier for a cryptanalyst to break the cipher.

To prevent this, block ciphers are always augmented to make the
ciphertext for a block vary depending on context. Ways in which a
block cipher may be augmented are called *modes of operation*. A
common mode of operation is *cipher block chaining* (CBC), in which
each plaintext block is XORed with the previous block’s ciphertext
before being encrypted. The result is that each block’s ciphertext
depends in part on the preceding blocks (i.e., on its context). Since
the first plaintext block has no preceding block, it is XORed with a
random number. That random number, called an *initialization vector*
(IV), is included with the series of ciphertext blocks so that the
first ciphertext block can be decrypted. This mode is illustrated in
:numref:`Figure %s <fig-cbc>`. Another mode of operation is *counter
mode*, in which successive values of a counter (e.g., 1, 2, 3,
:math:`\ldots`) are incorporated into the encryption of successive
blocks of plaintext.

.. _fig-cbc:
.. figure:: figures/f08-02-9780123850591.png
   :width: 500px
   :align: center

   Cipher Block Chaining.

Block size, like key size, is a design tradeoff. 64-bit blocks were
used for many years but as computer and network speeds increased and
storage costs dropped, 64-bit blocks became vulnerable to a type of
attack based on the *birthday problem* (the odds of 2 people in a
group of size *n* having the same birthday being surprisingly
high). If an attacker can manage to keep an encrypted session open
long enough to receive two identical 64-bit blocks, they can gain
useful information on the plaintext that produced the blocks. This was
known to be an issue in theory for a long time but the exploitation of
it was proven in 2016 leading to the deprecation of 64-bit blocks.

Block ciphers imply the padding of messages up to the next block
boundary, which wastes some network bandwidth, so there is a cost to
overly large blocks. For this reason most ciphers today have settled
on 128-bit blocks. Some details on how the birthday attacks were shown
to be an issue is available at the "Sweet32" website.



.. admonition:: Further Reading

   Sweet32. `Birthday attacks on 64-bit block ciphers in TLS and OpenVPN
   <https://sweet32.info>`__. 






3.2 Secret-Key Ciphers
------------------------

In a secret-key cipher, both participants in a communication share the
same key.\ [#]_ In other words, if a message is encrypted using a particular
key, the same key is required for decrypting the message. If the
cipher illustrated in :numref:`Figure %s <fig-genericCrypto>` were a
secret-key cipher, then the encryption and decryption keys would be
identical. Secret-key ciphers are also known as symmetric-key ciphers
since the secret is shared with both participants. We’ll take a look
at the alternative, public-key ciphers, shortly. (Public-key ciphers
are known as also asymmetric-key ciphers, since as we’ll soon see, the
two participants use different keys.)

.. [#] We use the term *participant* for the parties involved in a
       secure communication since that is a common networking term to
       identify the two endpoints of a communication channel. In the
       security world, the parties are often called *principals*.
       
The U.S. National Institute of Standards and Technology (NIST) has
issued standards for a series of secret-key ciphers. *Data Encryption
Standard* (DES) was the first, and it survived for several decades
before being deprecated. 

DES keys have 56 independent bits (although they have 64 bits
in total; the last bit of every byte is a parity bit). As noted above,
you would, on average, have to search half of the space of 2\
:sup:`56` possible keys to find the right one, giving 2\ :sup:`55` =
3.6 × 10\ :sup:`16` keys.  That may sound like a lot, but  by the late 1990s, it was
already possible to recover a DES key after a few hours. Consequently,
NIST updated the DES standard in 1999 to indicate that DES should only
be used for legacy systems. Importantly, DES was never shown to be
vulnerable to any attack other than brute force.

DES was initially replaced by *Triple DES* (3DES). A 3DES key has 168
(= 3 × 56) independent bits. As noted above, the computational cost of
launching a brute-force attack was not, as expected, the cost of
searching a 2\ :sup:`168` key space, but rather a 2\ :sup:`112` key
space search due to the "meet in the middle" attack. At the same time, the
computational cost to perform encryption and decryption was still three
times higher than with single DES. This ultimately led to a process to
replace DES with newer algorithms.


3DES is now deprecated in favor of the *Advanced Encryption Standard*
(AES) issued by NIST. The cipher underlying AES (with a few
minor modifications) was originally named Rijndael (pronounced roughly
like “Rhine dahl”) based on the names of its inventors, Daemen and
Rijmen.  AES supports key lengths of 128, 192, or 256 bits, and the
block length is 128 bits. AES permits fast implementations in both
software and hardware, being somewhat more efficient than triple
DES. It doesn’t require much memory, which makes it suitable for small
mobile devices. AES has some mathematically strong security properties
and, as of the time of writing, has not suffered from any significant
practical attacks.

We won't go further into the details of secret-key ciphers, since it
really is a field for specialist cryptographers. The security expert
Bruce Schneier puts it this way:

  Anyone, from the most clueless amateur to the best cryptographer,
  can create an algorithm that he himself can’t break. It’s not even
  hard. What is hard is creating an algorithm that no one else can
  break, even after years of analysis. And the only way to prove that
  is to subject the algorithm to years of analysis by the best
  cryptographers around. 

3.3 Public-Key Ciphers
------------------------

An alternative to secret-key ciphers is public-key ciphers. Instead of
a single key shared by two participants, a public-key cipher uses a pair
of related keys, one for encryption and a different one for decryption.
The pair of keys is “owned” by just one participant. The owner keeps
one key of the pair secret. That key is the *private key*. The owner makes the second key
public; that key is called the *public key*.
Obviously, it must
not be possible to deduce the private key from the public key.

Anyone can get the public key and then use it to encrypt a message
before sending it to the owner of the keys. Only the owner has the
private key necessary to decrypt such a message. This scenario is depicted in
:numref:`Figure %s <fig-public>`.

.. _fig-public:
.. figure:: figures/f08-03-9780123850591.png
   :width: 500px
   :align: center

   Public-key encryption.

Because it is somewhat unintuitive, we emphasize that the public
encryption key is useless for decrypting a message—you couldn’t even
decrypt a message that you yourself had just encrypted unless you had
the private decryption key. If we think of keys as defining a
communication channel between participants, then an important difference
between public-key and secret-key ciphers is the topology of the
channels. A key for a secret-key cipher provides a channel that is
two-way between two participants—each participant holds the same
(symmetric) key that either one can use to encrypt or decrypt messages
in either direction. A public/private key pair, in contrast, provides
a channel that is one way and many-to-one: from everyone who has the
public key to the unique owner of the private key, as illustrated in
:numref:`Figure %s <fig-public>`.


Public-key ciphers are used  not just for encryption, but also for
authentication. The way this works is that the
private key can be used with the *encryption* algorithm to
encrypt messages so that they can then only be decrypted using the public
key. This property clearly wouldn’t be useful for
confidentiality since anyone with the public key could decrypt such a
message. This property is,
however, useful for authentication since it tells the receiver of such
a message that it could only have been created by the owner of the
keys (subject to certain assumptions that we will get into
later). This is illustrated in :numref:`Figure %s <fig-pksign>`.

It
should be clear from the figure that anyone with the public key can
decrypt the encrypted message, and, assuming that the result of the
decryption matches the expected result, it can be concluded that the
private key must have been used to perform the encryption. Exactly how
this operation is used to provide authentication is the topic of a
later chapter.

To summarize, public-key ciphers may be used both for authentication
and to support confidential communications by encryption. In the
latter case, they are mostly used to *bootstrap* confidential
communications. Rather than encrypting the majority of messages sent
between participants, public-key ciphers are used to confidentially
distribute secret (symmetric) keys, leaving the rest of
confidentiality to secret-key ciphers. The symmetric key sent over
this confidential channel is called a *session key*. The reasons for this two-step
approach include the higher efficiency of secret-key ciphers, and the need
for reasonably frequent changing of encryption keys as described
above. 

.. _fig-pksign:
.. figure:: figures/f08-04-9780123850591.png
   :width: 500px
   :align: center

   Authentication using public keys.

Public-key cryptograpy has an interesting history. The concept of
public-key ciphers was first published in 1976 by Diffie and
Hellman. Subsequently, however, documents have come to light proving
that Britain’s Communications-Electronics Security Group had
discovered public-key ciphers by 1970, and the U.S. National Security
Agency (NSA) claims to have discovered them in the mid-1960s.

The best-known public-key cipher is RSA, named after its inventors:
Rivest, Shamir, and Adleman. RSA relies on the high computational cost
of factoring large numbers. The problem of finding an efficient way to
factor numbers is one that mathematicians have worked on unsuccessfully
since long before RSA appeared in 1978, and RSA’s subsequent resistance
to cryptanalysis has further bolstered confidence in its security.
Unfortunately, RSA needs relatively large keys, at least 1024 bits, to
be secure. This is larger than keys for secret-key ciphers because it is
faster to break an RSA private key by factoring the large number on
which the pair of keys is based than by exhaustively searching the key
space.

Another public-key cipher is ElGamal. Like RSA, it relies on a
mathematical problem, the discrete logarithm problem, for which no
efficient solution has been found, and requires keys of at least 1024
bits. There is a variation of the discrete logarithm problem, arising
when the input is an elliptic curve, that is thought to be even more
difficult to compute; cryptographic schemes based on this problem are
referred to as *elliptic curve cryptography*.

Public-key ciphers are, unfortunately, several orders of magnitude
slower than secret-key ciphers. Consequently, secret-key ciphers are
used for the vast majority of encryption, while public-key ciphers are
reserved for use in authentication and session key establishment.

3.4 Message Authentication Codes
---------------------------------

Encryption alone does not provide data integrity. For example, just
randomly modifying a ciphertext message could turn it into something
that decrypts into valid-looking plaintext, in which case the tampering
would be undetectable by the receiver. Nor does encryption alone provide
authentication. In a sense, integrity and authentication are
fundamentally inseparable. It is not much use to say that a message came from a
certain participant if the contents of the message have been modified
after that participant created it.

An *message authentication code* is a value, to be included in a transmitted message,
that can be used to verify simultaneously the authenticity and the data
integrity of a message. We will see later how such codes can be used in
protocols. For now, we focus on the algorithms that produce
authentication codes. 

When data is stored or transmitted, it is routine to use
error-detecting or error-correcting codes. These are pieces of
information added to a stored or transmitted data object so the
receiver detects when the data has been inadvertently modified by bit
errors. Error-correcting codes are used on CDs and DVDs, for example, to deal with
data corruption from scratches or dust. A similar concept applies to
authenticators, with the added challenge that the corruption of the
message is likely to be deliberately performed by someone who wants
the corruption to go undetected. To support authentication, an
authenticator includes some proof that whoever created the
authenticator knows a secret that is known only to the alleged sender
of the message; for example, the secret could be a key, and the proof
could be some value encrypted using the key. There is a mutual
dependency between the form of the redundant information and the form
of the proof of secret knowledge. We will discuss several workable
combinations.

For simplicity, let's assume initially that the original message need not be
confidential—that a transmitted message will consist of the plaintext of
the original message plus an authentication code. Later we will consider the
case where confidentiality is desired.

One kind of message authentication combines encryption and a
*cryptographic hash function*. Cryptographic hash algorithms are
treated as public knowledge, as with cipher algorithms. A
cryptographic hash function is a function that outputs sufficient
information about a message to expose any tampering. Just as a
checksum or error-detecting code exposes bit errors introduced by
noisy links or scratched disks, a cryptographic checksum is designed
to expose deliberate corruption of messages by an adversary. The value
it outputs is called a *message digest* and, like an ordinary
checksum, is appended to the message. All the message digests produced
by a given hash have the same number of bits regardless of the length
of the original message. Since the space of possible input messages is
larger than the space of possible message digests, there will be
different input messages that produce the same message digest, like
collisions in a hash table. An important property of hash functions is
that such collisions may not be produced deliberately under
the control of the attacker.

A message authentication code can be created by encrypting the message digest. The
receiver computes a digest of the plaintext part of the message and
compares that to the decrypted message digest. If they are equal, then
the receiver would conclude that the message is indeed from its alleged
sender (since it would have to have been encrypted with the right key)
and has not been tampered with. No adversary could get away with sending
a bogus message with a matching bogus digest because she would not have
the key to encrypt the bogus digest correctly. An adversary could,
however, obtain the plaintext original message and its encrypted digest
by eavesdropping. The adversary could then (since the hash function is
public knowledge) compute the digest of the original message and
generate alternative messages looking for one with the same message
digest. If she finds one, she could undetectably send the new message
with the old authentication code. Therefore, security requires that the hash
function have the *one-way* property: it must be computationally
infeasible for an adversary to find any plaintext message that has the
same digest as the original.

For a hash function to meet this requirement, its outputs must be
fairly randomly distributed. For example, if digests are 128 bits long
and truly randomly distributed, then you would need to try 2\ :sup:`127`
messages, on average, before finding a second message whose digest
matches that of a given message. If the outputs are not randomly
distributed—that is, if some outputs are much more likely than
others—then for some messages you could find another message with the
same digest much more easily than this, which would reduce the
security of the algorithm. If you were instead just trying to find any
*collision*—any two messages that produce the same digest—then you
would need to compute the digests of only 2\ :sup:`64` messages, on
average.  This surprising fact is the basis of the “birthday
attack” mentioned above.

There have been several common cryptographic hash algorithms over the
years, including Message Digest 5 (MD5) and the Secure Hash Algorithm
(SHA) family. Weaknesses of MD5 and earlier versions of SHA have been
known for some time, which led NIST to recommend a family of
algorithms known as SHA-3 in 2015.

When generating an encrypted message digest, the digest encryption could use
either a secret-key cipher or a public-key cipher. If a public-key
cipher is used, the digest would be encrypted using the sender’s private
key (the one we normally think of as being used for decryption), and the
receiver—or anyone else—could decrypt the digest using the sender’s
public key.

A digest encrypted with a public key algorithm but using the private
key of the sender
is called a *digital signature* because it provides nonrepudiation
similar to that of
a written signature. The receiver of a message with a digital signature
can prove to any third party that the sender really sent that message,
because the third party can use the sender’s public key to check for
herself. Secret-key encryption of a digest does not have this property
because only the two participants know the key; furthermore, since both
participants know the key, the alleged receiver could have created the
message herself. Any public-key cipher can be used for digital
signatures. *Digital Signature Standard* (DSS) is a digital signature
format that has been standardized by NIST. DSS signatures may use any
one of three public-key ciphers, one based on RSA, another on ElGamal,
and a third called the *Elliptic Curve Digital Signature Algorithm*.

Another kind of authenticator is similar, but instead of encrypting a
hash it uses a hash-like function that takes a secret value (known
only to the sender and the receiver) as a parameter, as illustrated in
:numref:`Figure %s <fig-macAndHmac>`. Such a function outputs an
authenticator called a *message authentication code* (MAC). The sender
appends the MAC to her plaintext message. The receiver recomputes the
MAC using the plaintext and the secret value and compares that
recomputed MAC to the received MAC.

.. _fig-macAndHmac:
.. figure:: figures/f08-05-9780123850591.png
   :width: 600px
   :align: center

   Computing a MAC (a) versus computing an HMAC (b).

A common variation on MACs is to apply a cryptographic hash (such as
MD5 or SHA-1) to the concatenation of the plaintext message and the
secret value, as illustrated in :numref:`Figure %s
<fig-macAndHmac>`. The resulting digest is called a *hashed message
authentication code* (HMAC) since it is essentially a MAC. The HMAC,
but not the secret value, is appended to the plaintext. Only a receiver
who knows the secret value can compute the correct HMAC to compare
with the received HMAC. If it weren’t for the one-way property of the
hash, an adversary might be able to find the input that generated the
HMAC and compare it to the plaintext message to determine the secret
value.

Up to this point, we have been assuming that the message wasn’t
confidential, so the original message could be transmitted as plaintext.
To add confidentiality to a message with an authenticator, it suffices
to encrypt the concatenation of the entire message including its
authenticator—the MAC, HMAC, or encrypted digest. Remember that, in
practice, confidentiality is implemented using secret-key ciphers
because they are so much faster than public-key ciphers. Furthermore, it
costs little to include the authenticator in the encryption, and it
increases security. A common simplification is to encrypt the message
with its (raw) digest, such that the digest is only encrypted once; in
this case, the entire ciphertext message is considered to be an
authenticator.

Although authenticators may seem to solve the authentication problem, we
will see in a later chapter that they are only the foundation of a
solution. First, however, we address the issue of how participants
obtain keys in the first place.
