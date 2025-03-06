Chapter 8. Securing Network Infrastructure
==========================================
.. Brad notes
   I really enjoyed the CCR paper with anonymous authors on collateral
   damage of China’s censorship (IIRC, causing DNS lookup failures in
   other countries).
   That paper is not exactly current now, but it is a nice example of
   how a state actor can deploy things that break infrastructure
   outside its own state boundaries.
   My gut feeling is that material on why stock DNS is vulnerable to
   attack, what DNSSEC is, how it’s supposed to make things better,
   and why it’s hard to deploy would definitely be useful.
   And probably the same for BGP and the RPKI. Goldberg has a paper on
   why it’s so hard to secure routing; I think it was in Queue.
   I wonder if a synthesis of any sort is possible on this
   topic. Certainly certificate chains and delegated signature authority
   are at the core of both DNSSEC and the RPKI.
   Perhaps there is a unifying theme of securing infrastructure with distributed domains of control.
   In a way CAs fit this model, too.


In the preceding chapters we have focused on the security of
end-systems connected to the Internet and on securing communication
between parties as their traffic traverses the Internet. But the
Internet itself consists of infrastructure, such as routers, that must
also be defended against attacks. Two areas of particular concern are
the interdomain routing system and the domain name system. Both
systems routinely come under attack. There are efforts under way to
make them more resistant to attacks as we discuss in the following
sections.


8.1 BGP
----------

In most respects, a router is just a special-purpose computer with
some high-speed interfaces and specialized software to perform tasks
such as route computation and advertisement. So they need to be
protected like end-systems, e.g., with strong passwords and
multi-factor authentication, using access control lists and firewalls,
etc.  That is only a starting point, however, because the actual
routing protocols themselves represent an opportunity for attack. BGP,
the Border Gateway Protocol, is vulnerable to a wide range of
attacks, and is the only protocol that is expected to cross the
boundaries of a single administrative domain, so we focus our
attention here.

The primary challenge in securing the Internet's routing
infrastructure boils down to this: how can you trust a routing
announcement received via BGP? At first glance, this looks similar to
the problem solved by TLS: how do we know that we're talking to the
web site we wanted to connect to? But there are multiple levels to
this problem when it comes to inter-domain routing.  When you have a
secure, encrypted connection to your bank, you probably trust them to
show you accurate information about your account (mostly-banks do make
mistakes on occasions) and the secure connection protects if from
modification by an attacker. A secure, encrypted connection to the
website of the New York Times, however, doesn't mean you should
believe every word published by the New York Times. Similarly, a
secure connection to a BGP speaker doesn't imply that every route
advertisement provided by that speaker is reliable. We need to look a
bit more closely at how BGP works to see where the challenges lie.

BGP speakers advertise *paths* to reach *prefixes*. When a BGP speaker
receives a set of path advertisements from its peers, it runs a route
selection process to determine the "best" path to any prefix, using a
fairly large and flexible set of criteria to decide what is
"best". For example, a path to a given prefix that is shorter (as
measured by the number of autonomous systems it contains) may be
preferred to one that is longer. However, there are many other
criteria, notably the business relationship between the peers, which
are used to determine the ultimate choice of path. When a BGP speaker
has chosen a path to reach a particular prefix, it may choose to
re-advertise that path to other BGP speakers, either in the same AS or
in another AS. For a more full discussion of how BGP works, refer to
the section on Inter-domain routing in our main textbook.

A BGP speaker needs to trust that the paths that it is receiving from
its peers are correct, and this turns out to be a multi-faceted
challenge. Geoff Huston, the Chief Scientist at APNIC, has written a
useful taxonomy of the threats that BGP faces, all of which relate to
the trust among peers.

The taxonomy asks the following questions of the communication between
two BGP speakers:

* How is the BGP session protected from
  modification or disruption?
* How does a speaker verify the identity of its peer?
* How does a speaker verify the authenticity and completeness of the
  routing information received from a peer?
* How does a speaker know that the advertisements received actually
  represent the true forwarding state of the peer?
* How current is the information received, and is it still valid?

The first two bullets are versions of the authentication and integrity
issues that we addressed in Chapter 5. There are some details specific
to BGP that we cover below. The remaining challenges all relate to the
correctness of routing information carried by BGP and turn out to be
more difficult to address.

.. _reading_threat:
.. admonition::  Further Reading

   Geoff Huston. `A Survey on Securing Inter-Domain Routing Part 1 –
   BGP: Design, Threats and Security Requirements
   <https://labs.apnic.net/index.php/2021/08/03/a-survey-on-securing-inter-domain-routing-part-1-bgp-design-threats-and-security-requirements/>`__.
   APNIC Blog, August 2021.

   
   Peterson, L. and Davie, B. `Computer Networks: A Systems Approach. Interdomain
   Routing <https://book.systemsapproach.org/scaling/global.html#interdomain-routing-bgp>`__.


8.1.1 Authentication and Integrity
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Since BGP runs over a TCP connection, it was recommended for many
years that the TCP connection be authenticated and integrity-protected
using MD5 authentication. The MD5 authentication option for TCP is now
viewed as insufficiently secure (due to known attacks on the MD5
algorithm). It also lacks dynamic key management and the ability to update the
cryptographic algorithm, so it is now deprecated in favor of a more
general TCP authentication option which is described in RFC 5925.

It might seem somewhat counter-intuitive that BGP does not currently run
over TLS, given the maturity and widespread adoption of that
technology today. The explanation for this comes down to a combination
of factors including history, inertia, and the different operational
model of running BGP versus connecting to a remote website. For
example, BGP sessions often run between directly connected routers at
a peering point or internet exchange points (IXPs) which allows for a
simple TTL-based method to prevent spoofing. Privacy of BGP updates is
considerably less important than authenticity. And as we shall see,
there is a lot more to establishing the authenticity of a BGP
advertisement that just authenticating the messages from a peer.

When BGP was being developed in the 1980s and 1990s, TLS was still far
in the future, and packet encryption and decryption operations were
generally quite computationally expensive. So it made sense that the
initial focus was on authenticating messages rather than providing the
greater protection of encryption that TLS offers.

With this background, the idea of running BGP over TLS is an area of
current research, and would offer potential benefits beyond simply
adding privacy to BGP advertisements. Most notably, certificate
management for TLS is now highly automated, which contrasts with the
manual provisioning that must be performed when using MD5
or TCP-based authentication for BGP. The paper below on secure
transport for BGP outlines this approach and suggests further
enhancements that might further improve the security of BGP. However,
in terms of the current state of the Internet, the recommended best
practice for BGP sessions (described in RFC 7454) does not extend to
running BGP over TLS.

Whether TLS or a more basic authentication mechanism is used, the
effect is only to verify that the information came from the intended
peer and has not been modified in transit. The much more challenging
part of Internet routing security is in the validation of the routing
information itself. When a peer announces that they have a path to a
certain prefix, how do we know that they really do have this path?


.. _reading_BGPTLS:
.. admonition::  Further Reading

   Thomas Wirtgen, Nicolas Rybowski, Cristel Pelsser, Olivier
   Bonaventure. `The Multiple Benefits of Secure Transport for
   BGP <https://conferences.sigcomm.org/co-next/2024/files/papers/p186.pdf/>`__.
   ACM CONEXT, December 2024.

   J. Durand, I. Pepelnjak and G. Dorering. `BGP Operations and
   Security <https://www.rfc-editor.org/info/rfc7454>`__. RFC 7454,
   February 2015.

8.1.2 Correctness of Routing Information
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a BGP speaker announces a path to a particular prefix, how do we
know that they really have such a path? The short answer is that we
don't, but there has been a multi-decade quest to build mechanisms that
enable greater confidence in the correctness of such
announcements. This quest, and the slowness of its progress, was well
documented by Sharon Goldberg in 2014, and the progress continues
today.

Let's start with a simple and well-studied example. 

.. _reading_rpki:
.. admonition::  Further Reading

   Sharon Goldberg. `Why Is It Taking So Long to Secure Internet
   Routing? <https://dl.acm.org/doi/pdf/10.1145/2668152.2668966/>`__.
   ACM Queue, August 2014.

8.2 DNS
----------

Threat model (RFC 3833) - explain cache poisoning

DNS over HTTP (DoH)

DNSSEC

   Geoff Huston. `Calling Time on DNSSEC?
   <https://labs.apnic.net/index.php/2024/05/27/calling-time-on-dnssec/>`__.
   APNIC Blog, May 2024.

   RFC 3833

   RFCs 4033-4035


   ----
   adoption of RPKI vs DNSSEC - the difference between detecting
   corrupt info vs. preventing spread of corrupt info

   compare infra mechanisms vs e2e, notably TLS


8.3 DOS-preventing infra (Cloudflare et al)
