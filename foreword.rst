Foreword
========

The Internet gave the world's population unprecedented access to
information. Regrettably, it also connected the world's miscreants to
the world's end systems. Not long thereafter, remote services relied
on by users for critical tasks like banking, healthcare, and
government concentrated vast troves of personal information on
Internet-connected servers. The multi-decade steady drip of
attacker-driven data compromises and service outages attests that
these essential services and their data are attractive targets—and
have been vulnerable ones.

Pervasive, fast connectivity and diverse, sophisticated services are
some of the Internet's greatest successes. Internet security, by
contrast, remains a uniquely enduring challenge. It was not a priority
in early designs of network protocols and networked applications. It
is also inherently adversarial: as one improves a system's robustness
to attack, attackers adapt their "workload" specifically to target
any remaining omissions or oversights. What's more, while users and
service operators alike often value visibly new functionalities as
"features" justifying increased complexity or cost, they may not as
readily perceive security—roughly, the absence of undesired
unauthorized operations—as a feature.

How should one approach building secure networks and networked
applications that are robust and practical to deploy? A neophyte might
expect that building secure networks is a matter of "just add
cryptography." Surely, if one just chooses a good cryptography
library and updates the implementations of existing network protocols
to encrypt and sign messages, that's all that's needed.

Let's consider just a few of the many ways this simplistic view falls
short of capturing the scope and complexity of secure networked system
design:

* Cryptography requires keys typically associated with individual
  users, servers, and/or server operators. How does one distribute the
  right keys robustly and efficiently to billions of Internet users
  and servers that share no explicit prior pairwise trust
  relationship (e.g., where an online shopper has no prior
  relationship with the store's management or employees)?
* Suppose an adversary, while unable to view an encrypted
  message's cleartext content, records the encrypted message and sends
  an extra copy of it to the message's receiver. Any signature by the
  sender within the repeated message will still be valid. If the
  repeated message invokes an operation at the receiver (e.g., "Order
  an expensive item and charge the customer"), the consequences of
  the receiver's incorrectly acting on it may be dire. How can the
  original sender and receiver arrange for the receiver to detect and
  ignore such replayed messages?
* In the era of gigabit fiber to the home and 5G cellular
  connections, wide-area Internet latency often limits web and cloud
  app performance, rather than link speeds. Even a single added
  network round-trip in a communication can degrade user
  experience. As we secure communication protocols, how can we avoid
  adding network round-trips, so that server operators and users don't
  claim performance losses as a reason not to adopt secure designs?

It quickly becomes clear that designing systems that secure users'
data and preserve service availability in networks like the Internet
is a technically far broader and more subtle enterprise than "just add
cryptography." In network security, not least because an adversary
will actively seek out and leverage design lapses in any system
component, a true systems approach is essential: one must reason
precisely about the intended end-to-end security properties a
networked application requires; consider how threats to those
properties can plausibly manifest across the many components and
layers of the full, end-to-end network system and in their
interactions (on end hosts, at routers, and in the distributed
Internet infrastructure, e.g., for route dissemination, name
resolution, and TLS certificate distribution); and design robust defenses
against those manifestations across all those components and layers.

This book fills a vital niche: it comprehensively initiates the reader
into a systems approach to reasoning about network security, in all
its depth and nuance. Peterson and Davie do so by introducing the
fundamental design principles and toolkit of cryptographic building
blocks that underlie secure systems, and then illustrating their
application in a series of real-world, widely used secure network
systems. At every step they lucidly explain from first principles why
these designs take the form they do. Their treatment of recent
developments of note in network security, such as 0-RTT session
resumption in TLS 1.3 (a key technique for reducing handshake latency
in performance-sensitive applications, already widely deployed),
Passkeys and WebAuthn authentication in browsers (promising methods
for moving user authentication beyond the pain of passwords, also
already widely deployed), and ASPA for path validation in BGP routing
(so recent that it is still an Internet-Draft) is particularly
welcome. Their retrospectives on factors influencing uptake of secure
designs for Internet name resolution (DNS) and routing (BGP) add rare
insight into designing with an eye to incentive for adoption, beyond
protocol technical correctness.

In an area as complex and subtle as network system security, it is
unusual for a single volume to start with a discussion of elementary
principles and progress all the way through to recent developments in
the field, without compromising on essential detail or
clarity. Peterson and Davie's meticulous explanations of a judiciously
selected set of practically relevant secure network system designs
achieve that, and are what make this book so rewarding.

| Brad Karp
| University College London (UCL)
