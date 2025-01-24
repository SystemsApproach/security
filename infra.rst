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
fairly large set of criteria to decide what is "best". For example, a
path to a given prefix that is shorter (as measured by the number of autonomous systems
it contains) may be preferred to one that is longer. However, there are
many other criteria, notably the business relationship between the
peers, which are used to determine the ultimate choice of path. For a
more full discussion of how BGP works, refer to the section on
Inter-domain routing in our main textbook.

.. _reading_bgp:
.. admonition::  Further Reading

   Peterson, L. and Davie, B. `Computer Networks: A Systems Approach. Interdomain
   Routing <https://book.systemsapproach.org/scaling/global.html#interdomain-routing-bgp>`__.

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

.. _reading_threat:
.. admonition::  Further Reading

   Geoff Huston. `A Survey on Securing Inter-Domain Routing Part 1 –
   BGP: Design, Threats and Security Requirements
   <https://labs.apnic.net/index.php/2021/08/03/a-survey-on-securing-inter-domain-routing-part-1-bgp-design-threats-and-security-requirements/>`__.
   APNIC Blog, August 2021.

Since BGP runs over a TCP connection, it was recommended for many
years that the TCP connection be authenticated and integrity-protected
using MD5 authentication. The MD5 authentication option for TCP is now
viewed as insufficiently secure (due to known attacks on the MD5
algorithm). It also lacks dynamic key management and the ability to update the
cryptographic algorithm, so it is now deprecated in favor of a more
general TCP authentication option which is described in RFC 5925.

RFC 5925

RFC 7454

.. _reading_BGPTLS:
.. admonition::  Further Reading

   Thomas Wirtgen, Nicolas Rybowski, Cristel Pelsser, Olivier
   Bonaventure. `The Multiple Benefits of Secure Transport for BGP <https://conferences.sigcomm.org/co-next/2024/files/papers/p186.pdf/>`__.
   ACM Conext, December 2024.

.. _reading_rpki:
.. admonition::  Further Reading

   Sharon Goldberg. `Why Is It Taking So Long to Secure Internet
   Routing? <https://dl.acm.org/doi/pdf/10.1145/2668152.2668966/>`__.
   ACM Queue, August 2014.

8.2 DNS
----------

DNS over HTTP (DoH)

DNSSEC

   Geoff Huston. `Calling Time on DNSSEC?
   <https://labs.apnic.net/index.php/2024/05/27/calling-time-on-dnssec/>`__.
   APNIC Blog, May 2024.
