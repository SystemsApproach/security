Chapter 6. Transport Layer Security (TLS, SSL, HTTPS)
=======================================================

To understand the design goals and requirements for the Transport Layer
Security (TLS) standard and the Secure Socket Layer (SSL) on which TLS
is based, it is helpful to consider the main problems that they
were invented to solve. As the World Wide Web became popular and
commercial enterprises began to take an interest in it, it became clear
that some level of security would be necessary for transactions on the
Web. The canonical example of this is making purchases by credit card.
There are several issues of concern when sending your credit card
information to a computer on the Web. First, you might worry that the
information would be intercepted in transit and subsequently used to
make unauthorized purchases. You might also worry about the details of a
transaction being modified, such as changing the purchase amount. And
you would certainly like to know that the computer to which you are
sending your credit card information is in fact one belonging to the
vendor in question and not some other party. Thus, we immediately see a
need for confidentiality, integrity, and authentication in Web
transactions. The first widely used solution to this problem was SSL,
originally developed by Netscape and subsequently the basis for the
IETF’s TLS standard.

The designers of SSL and TLS recognized that these problems were not
specific to Web transactions (i.e., those using HTTP) and thus they built
a general-purpose protocol that sits between an application protocol
such as HTTP and a transport protocol such as TCP. The reason for
calling this “transport layer security” is that, from the application’s
perspective, this protocol layer looks just like a normal transport
protocol except for the fact that it is secure. That is, the sender can
open connections and deliver bytes for transmission, and the secure
transport layer will get them to the receiver with the necessary
confidentiality, integrity, and authentication. By running the secure
transport layer on top of TCP, all of the normal features of TCP
(reliability, flow control, congestion control, etc.) are also provided
to the application. This arrangement of protocol layers is depicted in
:numref:`Figure %s <fig-tls-stack>`.

.. _fig-tls-stack:
.. figure:: figures/f08-15-9780123850591.png
   :width: 300px
   :align: center

   Secure transport layer inserted between application and TCP layers.

When HTTP is used in this way, it is known as HTTPS (Secure HTTP). In
fact, HTTP itself is unchanged. It simply delivers data to and accepts
data from the SSL/TLS layer rather than TCP. For convenience, a default
TCP port has been assigned to HTTPS (443). That is, if you try to
connect to a server on TCP port 443, you will likely find yourself
talking to the SSL/TLS protocol, which will pass your data through to
HTTP provided all goes well with authentication and decryption. Although
standalone implementations of SSL/TLS are available, it is more common
for an implementation to be bundled with applications that need it,
primarily web browsers and servers.

The layered approach, inserting TLS between the application protocol
and the transport protocol, is not without drawbacks, particularly
when performance is considered. This eventually led to a rethinking
of the layering and a new transport protocol, QUIC, was developed with
the benefit of decades of experience with TLS and HTTP. We return to
this development below.

In the following discussion we focus
on TLS. SSL is now something of a historical artifact. TLS continues
to evolve as new weaknesses are identified and fixed, and new
cryptographic algorithms continue to be added, often replacing older
ones now considered insufficiently strong.

Like TCP, TLS has a setup phase involving handshakes followed by a connected
phase in which data is exchanged between the endpoints. We discuss
each in turn, beginning with the *handshake protocol* that supports
initial setup. Once the connection properties are established by the
handshake, the *record protocol* is used to protect the data sent
between the endpoints.

Performance is always important when applying cryptographic operations
to data transfers, and it has a significant impact on end-to-end
latency for web interactions. One performance-related design choice is
the use of symmetric key cryptography for the encryption and
authentication of data after the handshake is complete. In addition,
there have been efforts to reduce the number of round-trip
times required to begin data transfer, thus lowering initial latency.
We examine these developments later in this chapter.


6.1 Handshake Protocol
-----------------------

The main job of the handshake protocol is to allow a pair of TLS
participants to establish a shared secret key and negotiate at runtime
the set of cryptographic algorithms to use. It also allows for version
negotiation, so that, for example, a client running TLS version 1.2
can talk to a server that supports both version 1.2 and 1.3. It also
allows the client to authenticate the server, and, optionally, for
the client to be authenticated as well.

The handshake protocol needs to be resistant to man-in-the-middle
(MITM) attacks, which we discussed in Chapter 4. At one point in its
history, TLS version negotiation could be subverted by a MITM in such
a way that the client and server settled on a lower version than
necessary, opening up the risk that old vulnerabilities in the lower
version could be exploited.

TLS takes multiple precautions to increase its resistance against MITM
attacks. TLS 1.3 encrypts most of the handshake protocol, as early in
the process as possible. To facilitate this, the first step of the
handshake entails the establishment of a shared secret between the
client and the server. This is most commonly achieved using an
ephemeral Diffie-Hellman key exchange as described in Chapter
4. Pre-shared keys are also supported and have a role in restarting a
session quickly, as discussed in Section 6.3.

When we described Diffie-Hellman in chapter 4 we explained the original
algorithm that operates on groups of integers using modular
arithmetic. This is now known as Finite Field Diffie-Hellman. It is
also possible to use elliptic curves rather than modular arithmetic
and both options are available in modern TLS.

There are a number of ways the handshake protocol can play out, but
a typical set of operations is as follows:

1. The client sends a "client hello" message which specifies which
   Diffie-Hellman groups or elliptic curves it supports, along with an
   ephemeral Diffie-Hellman key for each group/curve. The hello also
   contains a nonce, the set of cipher suites that the client can use
   for subsequent encryption and authentication, and the TLS version
   the client supports.

2. The server replies with a "server hello" message indicating which
   Diffie-Hellman group or curve it has chosen and a corresponding
   ephemeral Diffie-Hellman public key. The hello also states which
   TLS version the server supports, the cipher suite it has chosen
   among those offered by the client, and a nonce.

Recall that a simple Diffie-Hellman key exchange is not secure against
MITM attacks, and the remaining steps in the handshake protect against
this. From the first two messages, the server and the client are able
to agree on a shared secret using one of several Diffie-Hellman
algorithms. A choice of groups or curves were provided in the client
hello, and one of them has been selected by the server. Similarly, one
of the offered cipher suites has been selected. With Diffie-Hellman
allowing them to obtain a shared secret, all subsequent messages
between client and server will be encrypted. But we still have to rule
out the MITM attack.

3. The server now sends one or more certificates. In the simplest
   case, there is a single certificate signed by a certification
   authority (CA) that is trusted by the client.

4. The server sends a "certificate verify" message, which proves that
   the server has the private key that corresponds to the public key
   in the previously supplied certificate. The signature covers
   everything that has been sent in the handshake up to this point,
   which includes a pair of nonces, thus providing protection against
   replay attacks. And the signature along with the certificate is
   sufficient to prove to the client that it is talking to the
   intended server, not to some attacker in the middle, who would be
   unable to provide the signature.

5. The server sends a "handshake finished" message which contains a
   hash of everything sent so far, ensuring that nothing in the
   handshake was tampered with. This further protects against MITM attacks.

6. The client sends a similar "handshake finished" message.

At this point the client knows that it is talking to the intended
server, and both parties know that they have successfully completed the
handshake without any tampering of messages. The server in this case
does not know who the client is because there has been no client
authentication. TLS does support client authentication using client
certificates, but it is not the norm in today's Internet for clients
to authenticate in this way.
   

.. 0RTT needs coverage somewhere
   Something about compatibility with 1.2 middleboxes

Recall that public key cryptography is computationally more expensive
than symmetric key cryptography, so we limit the use of public key
operations to the handshake protocol. And when we said above that all
the messages after the first two are encrypted, this is done using
symmetric keys. The role of public keys in TLS are (a) the
Diffie-Hellman key exchange (b) the use of certificates to
authenticate servers and, optionally, clients. All of that is limited
to the handshake protocol.
   



:numref:`Figure %s <fig-tls-hand>` shows the handshake protocol at a
high level.  When the client and server have each received a
"handshake finished" message from their respective peer, the handshake
is complete and application data can start to flow.

.. _fig-tls-hand:
.. figure:: figures/TLS-handshake.png
   :width: 400px
   :align: center

   Handshake protocol to establish TLS session.

Encryption of data between client and server is performed by TLS’s
*record protocol*. Because the handshake protocol in TLS 1.3 requires
encryption after the first two messages, the record protocol actually
comes into play at step 3 above, even before we get to sending any
application data. We discuss the details of the record protocol below.

6.2 Record Protocol
--------------------

.. WIP

The task of the record protocol is to protect the data that is sent
over a TLS connection with both encryption and authentication.  
While TLS supports a wide range of encryption and authentication
methods, the set of options has actually become narrower in version
1.3 as weaknesses of older methods became clear and new cryptographic
algorithms have emerged. All the algorithms in TLS 1.3 provide both
encryption and authentication in a single cipher suite, using the
technique known as authenticated encryption with additional data
(AEAD) which was discussed in Chapter 3.


In TLS, the cipher that provides authentication and encryption uses
two keys, one for each direction. Similarly, two initialization
vectors are required.  Thus, regardless of the choice of cipher suite,
a TLS session requires effectively four keys to be agreed upon by the
end points. TLS derives all of them from a single shared secret that
was obtained during the handshake phase.

The step that derives the keys and initialization vectors from the
shared secret is called the "HMAC-based extract-and-expand key
derivation function (HKDF)". The goal is to produce enough keying
material for the record layer–two IVs and two symmetric keys of
appropriate length–and to do so in such a way that an attacker has no
better way of guessing them than a brute force attack. In other words,
we want the keys and IVs to be as close to random as possible. This is
a bit harder than it might first appear, because the shared secret
that is obtained via Diffie Hellman, which is our starting point, is
not itself completely random. The reason for this may not be obvious,
but the goal of the various Diffie Hellman algorithms is to generate a
shared secret, not that such secrets be randomly distributed.

There is some fairly serious mathematics underlying HKDF, but the
basic idea is called "extract and expand". The first step is to
"extract" the randomness from the shared secret. This is done by
calculating a HMAC (hash-based message authentication code, as described
in Chapter 3) over the shared secret. The resulting pseudorandom key
is input to the next stage, along with an additional source of
randomness: the hash of everything contained in the initial
handshake. Note that the handshake messages include two random
nonces. The "expand" step then applies the HMAC function using these
inputs and HMAC is reapplied as many times as needed to produce the
required amount of key and IV material.

When all the keys and IVs are available to client and server, the record
layer can now protect the underlying data with encryption and
authentication. The record layer also handles fragmentation and
reassembly–breaking the incoming stream of plaintext into chunks of up
to 2\ :sup:`14` bytes. 

To encrypt one block for transmission, the record layer takes as input
the encryption key, a nonce (which we explain below), the plaintext to
be encrypted, and "additional data" to be authenticated but not
encrypted. This additional data is the header for the record layer,
indicating the type of data being encrypted (e.g., application data or
handshake data) and its length. The nonce is calculated by computing
the XOR of the IV and a sequence number that increments with every
block. The AEAD cipher then computes the ciphertext that will follow
the record header, and the resulting block is passed to the transport
layer (normally TCP) for transmission.

On the receiving side, the process runs in the other direction, with
the appropriate key, nonce, ciphertext and additional data (headers)
being passed to the AEAD decryption function. If authentication is
successful, the plaintext is recovered and can be passed up to the
application. If authentication does not succeed, the connection is
terminated and an alert is generated. 



6.3 Session Resumption and Zero RTT Operation
----------------------------------------------

In our initial description of the TLS handshake, we described how
Diffie-Hellman is used to established a shared secret, but noted
that the option also exists to use a pre-shared key (PSK). While
out-of-band provisioning of a PSK is possible, a much more common use
of a PSK is to allow session resumption, thus removing the need to go
through another Diffie-Hellman exchange.

An important side-effect of using a pre-shared key is that it becomes
possible to start sending data earlier in the process. This operation
is referred to as "0-RTT Data" because it is possible to start sending
application data along with the handshake material without waiting for
the round trip time of the handshake to elapse. This is an important
step in improving the latency of HTTPS connection establishment and
thus the user experience when browsing the Web. 

The idea of session resumption predates TLS 1.3 but it has evolved
somewhat to become more secure. In TLS 1.3, the server may create a
*session ticket* after the completion of the handshake process. The ticket
contains an opaque identifier of the session and a ticket lifetime (as
well as some other fields). The ticket is sent after the handshake
which means it is encrypted much like application data. More than one
ticket can be sent.

A ticket is effectively a label for a previously established
session, which has a shared secret already. When a client
connects to a server to which it was previously connected, it can look
at its stored tickets and, if there are any that have not expired, it
can include one in the first message of a handshake. Along
with the ticket, the client includes something called a "binding",
which is a HMAC calculated over the current handshake message using a
key derived from the *previous* handshake. The effect of this binding
is to tie the new handshake back to the old one, since only a client
that successfully completed the prior handshake can have the key
required to calculate the HMAC. Thus, while an attacker might snoop on
the ticket, it can't do much with it and any attempt to modify the new
handshake message will fail.

When the server sees that the client has sent a ticket, it validates
the binding, and if the HMAC calculation succeeds, then the server and
client now have agreement that they can use a shared secret
established in the prior session. They use a "resumption master
secret" that was calculated and stored in the prior session to derive
a new set of keys for this session. The keys of the new
session are different from those of the prior session to support
forward secrecy (i.e., an attacker who learns the key for session N
doesn't immediately have the keys for session N+1).

When the server sends its "Finished" message, it calculates the HMAC
over the handshake messages using the agreed-upon new key, and thus
authenticates itself to the client.

On its own, session resumption as just described may not seem that
interesting. It avoids the need for another Diffie-Hellman exchange
but is still requires a round trip time to establish the session. But
because the new session keys are known to both sides before the first
handshake message is sent, session resumption opens up the possibility
of sending "0-RTT data" along with the handhsake. 0-RTT data can be
included along with the handshake messages, without waiting one RTT
for keys to be established. This is beneficial from a performance
perspective, especially for short-lived connections, but it comes with
some downsides in terms of security.

There are two main drawbacks to 0-RTT data. The first is that it is
prone to replay attacks in a way that other data transfers are not. If
an attacker can sit between a client and a server, they have the
opportunity to replay 0-RTT data. Exactly how much damage this does is
very much application dependent, so the TLS specifications dictate
that (a) 0-RTT data can only be sent when the application layer
explicitly requests it, i.e., it can't just be an optimization
provided by the socket layer (b) the application must know how to deal
with replays of data sent as 0-RTT, e.g., by only sending 0-RTT
data for operations that are idempotent.

The other drawback of 0-RTT data is that it depends on keys that are
derived from secrets used in an earlier transaction. If those secrets
were somehow compromised, the attacker would have the necessary
information to compromise the new session. Thus, 0-RTT data lacks
forward secrecy. For this reason, the option exists to generate a new
set of keys as part of the session resumption handshake with a new
Diffie-Hellman exchange. This means that only the data sent in the
first RTT lacks forward secrecy, and the rest of the session is
protected by the new, uncompromised keys.


All of this work to reduce the setup time of TLS by a single RTT might
seem surprising, but in fact the history of HTTP and HTTPS over TCP is
full of issues with excessive setup times. The very first
implementations of HTTP were quite wasteful of TCP connections,
setting up a new connection for every object on a requested web
page. The history of HTTP over TCP and the addition of TLS is full of
efforts to reduce the latency since the most simple approaches just
layered one handshake on top of another. The next step in the process
of reducing the latency of TLS session establishment involves
rethinking the choice of TCP as the underlying transport, as we
discuss below. 



6.4 QUIC, HTTP/3 and TLS
------------------------

6.5 User Experience with HTTPS
------------------------------

