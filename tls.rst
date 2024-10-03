Chapter 6. Transport Layer Security (TLS, SSL, HTTPS)
=======================================================

To understand the design goals and requirements for the Transport Layer
Security (TLS) standard and the Secure Socket Layer (SSL) on which TLS
is based, it is helpful to consider one of the main problems that they
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
specific to Web transactions (i.e., those using HTTP) and instead built
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
when performanced is considered. This eventually led to a rethinking
of the layering and a new transport protocol, QUIC, was developed with
the benefit of decades of experience with TLS and HTTP. We return to
this development below.

In the following discussion we focus
on TLS. SSL is now something of a historical artifact. TLS continues
to evolve as new weaknesses are identified and fixed, and new
cryptographic algorithms continue to be added, often replacing older
ones now considered insufficiently strong.

Like TCP, TLS has a setup phase involving handshakes and a connected
phase in which data is exchanged between the endpoints. We discuss
each in turn, beginning with the *handshake protocol* that supports
intial setup. Once the connection properties are established by the
handshake, the *record protocol* is used to protect the data sent
between the endpoints.


6.1 Handshake Protocol
-----------------------

The main job of the handshake protocol is to allow a pair of TLS
participants to establish a shared secret key and negotiate at runtime
the set of cryptographic algorithms to use. It also allows for version
negotiation, so that, for example, a client running TLS version 1.2
can talk to a server that supports both version 1.2 and 1.3. It also
alllows the client to authenticate the server, and, optionally, for
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
session quickly, as discussed below.

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
of the offered cipher suites hase been selected. With Diffie-Hellman
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
   handshake was tampered with.

6. The client sends a similar "handshake finished" message.

At this point the client knows that it is talking to the intended
server, and both parties know that they have succesfully completed the
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
   
Encryption of data between client and server is
is performed by TLS’s *record protocol*, described in more detail
below. Because the handshake protocol in TLS 1.3 requires encryption
after the first two messages, the record protocol actually comes into
play at step 3 above, even before we get to sending any application
data. 


:numref:`Figure %s <fig-tls-hand>` shows the handshake protocol at a
high level.  When the client and server have each received a
"handshake finished" message from their respective peer, the handshake
is complete and application data can start to flow.

.. _fig-tls-hand:
.. figure:: figures/TLS-handshake.png
   :width: 400px
   :align: center

   Handshake protocol to establish TLS session.



We glossed over some of the details of how the client and server
encrypt messages after the initial part of the exchange. These details
are part of the record protocol, which we describe next.

6.2 Record Protocol
--------------------

.. WIP

In TLS, the confidentiality cipher uses two keys, one for each
direction, and similarly two initialization vectors. The HMACs are
likewise keyed with different keys for the two participants. Thus,
regardless of the choice of cipher and hash, a TLS session requires
effectively six keys. TLS derives all of them from a single shared
*master secret*. The master secret is a 384-bit (48-byte) value that in
turn is derived in part from the “session key” that results from TLS’s
session key establishment protocol.


Within a session established by the handshake protocol, TLS’s record
protocol adds confidentiality and integrity to the underlying transport
service. Messages handed down from the application layer are:

1. Fragmented or coalesced into blocks of a convenient size for the
   following steps

2. Optionally compressed

3. Integrity-protected using an HMAC

4. Encrypted using a secret-key cipher

5. Passed to the transport layer (normally TCP) for transmission

The record protocol uses an HMAC as an authenticator. The HMAC uses
whichever hash algorithm (MD5, SHA-1, etc.) was negotiated by the
participants. The client and server have different keys to use when
computing HMACs, making them even harder to break. Furthermore, each
record protocol message is assigned a sequence number, which is included
when the HMAC is computed—even though the sequence number is never
explicit in the message. This implicit sequence number prevents replays
or reorderings of messages. This is needed because, although TCP can
deliver sequential, unduplicated messages to the layer above it under
normal assumptions, those assumptions do not include an adversary that
can intercept TCP messages, modify messages, or send bogus ones. On the
other hand, it is TCP’s delivery guarantees that make it possible for
TLS to rely on a legitimate TLS message having the next implicit
sequence number in order.

Another interesting feature of the TLS protocol is the ability to resume
a session. To understand the original motivation for this, it is helpful
to understand how HTTP originally mades use of TCP connections. (The
details of HTTP are presented in the next chapter.) Each HTTP operation,
such as getting a page from a server, required a new TCP connection to
be opened. Retrieving a single page with a number of embedded graphical
objects might take many TCP connections. Opening a TCP connection
requires a three-way handshake before data transmission can start. Once
the TCP connection is ready to accept data, the client would then need
to start the TLS handshake protocol, taking at least another two
round-trip times (and consuming some amount of processing resources and
network bandwidth) before actual application data could be sent. The
resumption capability of TLS was designed to alleviate this problem.

The idea of session resumption is to optimize away the handshake in
those cases where the client and the server have already established
some shared state in the past. The client simply includes the session ID
from a previously established session in its initial handshake message.
If the server finds that it still has state for that session, and the
resumption option was negotiated when that session was originally
created, then the server can reply to the client with an indication of
success, and data transmission can begin using the algorithms and
parameters previously negotiated. If the session ID does not match any
session state cached at the server, or if resumption was not allowed for
the session, then the server will fall back to the normal handshake
process.

The reason the preceeding discussion emphasized the *original*
motivation is that having to do a TCP handshake for every embedded
object in a web page led to so much overhead, independent of TLS, that
HTTP was eventually optimized to support *persistent connections* (also
discussed in the next chapter). Because optimizing HTTP mitigated the
value of session resumption in TLS (plus the realization that reusing
the same session IDs and master secret key in a series of resumed
sessions is a security risk), TLS changed its approach to resumption in
the latest version (1.3).

In TLS 1.3, the client sends an opaque, server-encrypted *session
ticket* to the server upon resumption. This ticket contains all the
information required to resume the session. The same master secret is
used across handshakes, but the default behavior is to perform a session
key exchange upon resumption.

.. _key-layering:
.. admonition:: Key Takeaway

   We call attention to this change in TLS because it illustrates the
   challenge of knowing which layer should solve a given problem. In
   isolation, session resumption as implemented in the earlier version
   of TLS seems like a good idea, but it needs to be considered in the
   context of the dominate use case, which is HTTP. Once the overhead of
   doing multiple TCP connections was addressed by HTTP, the equation
   for how resumption should be implemented by TLS changed. The bigger
   lesson is that we need to avoid rigid thinking about the right
   layer to implement a given function—the answer changes over time
   as the network evolves—where a holistic/cross-layer analysis is
   required to get the design right.

