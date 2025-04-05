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


8.1 BGP and Routing Security
----------------------------

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

Let's start with a simple and well-studied example. In 2008, ISPs in
Pakistan were ordered by the government to block access to YouTube for
users in the country. One ISP (Pakistan Telecom) chose to do this by
advertising a route to a prefix that was within the range allocated to
YouTube. In effect, the ISP announced "I have a good path to YouTube"
so that it could then redirect traffic that would try to follow that
path. The problem was that not only was this path not a viable way to
reach YouTube, it was also a *more specific* path, that is, it was for
a longer prefix than the true path to YouTube that was being
advertised by other ASes. This turned into a problem well beyond the
boundaries of Pakistan when the ISP advertised the route upstream to a
larger ISP.  The upstream ISP now saw the more specific route as a
distinct piece of routing information from the true, less specific
route, and so it re-advertised this (false) path to its peers. Repeated
application of this decision to accept the more specific path and
re-advertise it caused much of the Internet to view the small ISP in
Pakistan as a true path to YouTube. Within minutes a large percentage
of the Internet was sending YouTube traffic to Pakistan, causing a
global outage for YouTube. Resolution was achieved by manual
intervention at multiple ISPs to stop the global advertisement of the
false path.

There are many other forms of attack possible on BGP, but they mostly
take the form of a route being advertised and then propagated when it
should not be. There is a relatively simple measure that should have
prevented the incident described above: the provider AS immediately
upstream from Pakistan Telecom  should not have accepted the
advertisement that said "I have a route to YouTube". How would it know
not to accept this? After all, BGP needs to be dynamic, so a newly
advertised prefix is sometimes going to be correct. One solution to
this problem is the use of Internet Routing Registries, which serve as
databases mapping address prefixes to the ASes that are authorized to
advertise them. In the prior example, since YouTube is not a customer
of Pakistan Telecom, the IRR would show that the YouTube prefix should
not be advertised by this AS. The responsibility to filter out the
false announcement falls on the *upstream* ISP, who would need to
periodically query one or more IRRs in order to maintain an up-to-date
set of filters to apply to its downstream peers.

There are numerous issues with the IRR approach, including the fact
that this sort of filtering gets much more difficult the closer you
get to the "core" of the Internet. It's one thing to filter prefixes
from an ISP that serves a modest number of customers in a single
country; it's another to filter prefixes coming from a large peer with
global presence. Some obviously bad routes can be filtered but it's
very hard to get a complete picture sufficient to rule out anything
incorrect that could be advertised. The set of rules that need to be
configured on a BGP router for an ISP that carries hundreds of
thousands of routes can also get very large. 

Furthermore, as noted by Sharon Goldberg in her article "*Why Is It
Taking So Long to Secure Internet Routing?*", the incentives for
prefix filtering are somewhat misaligned. The cost of filtering falls
on the AS that is immediately upstream of the misbehaving ISP, while
the benefit accrues to some distant entity (YouTube in our example)
who avoids the impact to their traffic thanks to the work of a
provider with whom they have no relationship.

A more sophisticated approach relies on the use of cryptographically
signed statements authorizing a particular AS to advertise paths to a
particular prefix. This technology behind this is referred to as RPKI:
Resource Public Key Infrastructure.

RPKI provides a means by which entities involved in routing, such as
the operator of an AS, can prove that they are authorized to advertise
routing information such as address prefixes. A Route Origin
Authorization (ROA), for example, contains a certificate, an AS
number, and a set of prefixes that the AS is authorized to
advertise. The ROA is cryptographically signed by an entity that is
itself trusted to provide this authorization, generally the AS to
which this address prefix has been allocated.

Because Regional Internet Registries (RIRs) are at the top of the hierarchy
for address allocation,  they are a logical place to place the root of
trust, known as a trust anchor, for the
RPKI. There are five RIRs globally and each has a root certificate in
the RPKI.

Address allocation is a hierarchical process. For example, an RIR can
allocate a chunk of address space to an ISP, and the ISP can
sub-allocate from that chunk to one of its customers. A hierarchy of
certificates can be used that follows this hierarchy of address
allocation.  The RIRs form trust anchors from which chains of trust
can be built, much the way a modern browser comes with a set of
trusted root certification authorities (CAs) so that the certificates
issued by web sites, which are signed by CAs, can be checked for validity.

A key distinction between RPKI and the certificates that we are
familiar with from TLS is this: the certificates in TLS are used to
validate the *identity* of a web site (e.g., a certificate for cnn.com
tells your browser that it is actually talking to cnn.com), whereas
RPKI certificates are used to validate the *resources* allocated to an
entity such as an ISP or an end customer. The resources in particular
include IP address prefixes. As IP address allocation starts with the RIRs
and proceeds down through ISPs to end customers, resource certificates
are generated at each level in the hierarchy.

.. _fig-rpki:
.. figure:: figures/rpki.png
   :width: 600px
   :align: center

   Chain of trust for RPKI

:numref:`Figure %s <fig-rpki>` shows how the
certificates are arranged for a simple example of an ISP *A* with
customer *C*. There is a chain of trust from the root certificate to
the customer, much like the sort of certification hierarchy we have seen
used for TLS. However, because the goal here is ultimately to certify the
authority of a certain AS to advertise a prefix, the details of the
certificates are different from those used in TLS. For example, the
certificate that ISP *A* issues, on the far right of the picture, says
that some address prefix has been allocated to customer *C*, and
includes the public key of customer C. This certificate is signed by
ISP *A* using the private key of *A*. So if we can trust *A*, we learn
two things about *C*: its public key and the set of addresses
allocated to the holder of that public key.

One level higher in the chain, the Local Internet Registry (LIR) has
issued a certificate that states ISP *A* has authority to allocate
addresses out of some prefix. The prefix that *A* has allocated to *C*
must be a subprefix within the allocation made by the LIR.
By following the chain back to the root certificate, it is possible to
establish that *C* is legitimately able to advertise the prefix
allocated to it by *A*.

*C* can now create a Route Origin Authorization (ROA) which it signs
with its private key. The ROA contains the AS number of *C* and the
prefix or prefixes that it wishes to advertise. Anyone who looks at
the ROA and the resource certificate chain that leads from the root CA to *C*
can validate that it has been signed with the private key
belonging to the entity authorized to advertise the prefixes in the
ROA. Because the ROA also contains the AS number for *C*, we now know
that we should trust advertisements of this prefix if they originate
from the stated AS.


Rather than being passed around in real time like certificates in TLS,
the RPKI certificates are stored in repositories, which are typically
operated by the RIRs. Address allocations happen at a relatively long
timescale, and certificates can be issued at the same time. Thus it is
feasible to fetch the entire contents of the RPKI repository to build up a
complete picture of the chains of certificates that have been
issued. With this information, a router running BGP can determine *in advance* which
ASes could originate routing advertisements for which prefixes and use
this to configure filtering rules that specify which advertisements they are
willing to accept. There is a well-established set of software tools
to automate this process for popular operating systems and commercial
routing platforms. Notably, the routers running BGP do not perform
cryptographic operations in real time when processing route
advertisements; all the cryptographic operations happen in advance
when setting up the filtering rules based on information from the RPKI
repository. 

With the RPKI in place it is now possible to perform Route Origin
Validation. That is, if a given AS claims to be the originator of a
certain prefix, that claim can be checked against the information in
the RPKI. So, for example, if Pakistan Telecom were now to claim to be the
origin AS for a subprefix of YouTube, that could immediately be
detected as false information and discarded by any router receiving
such an advertiment, not just the neighbors of the offending ISP.

While there are many forms of attack or misconfiguration that would
not be caught by ROV (such as an AS falsely claiming a shorter path that
doesn't actually exist, or a BGP speaker falsely claiming an AS number
not allocated to it) it does prevent a large number of issues,
especially those caused by misconfiguration. To more fully combat the
advertisement of false information in BGP, it is necessary to adopt
some sort of path validation, as discussed below.

The adoption of RPKI for route origin validation has been moving along
steadily for many years now. According to monitoring performed by
NIST, more than 55% of IPv4 prefixes advertised in the Internet (as
of the time of writing in 2025) have
a validated route origin. Just over 1% are invalid with the remainder
(43%) are not found, i.e., no ROA is found in the repository for those
prefixes. 

.. rubric:: path validation






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

   
   adoption of RPKI vs DNSSEC - the difference between detecting
   corrupt info vs. preventing spread of corrupt info

   compare infra mechanisms vs e2e, notably TLS


8.3 DOS-preventing infra (Cloudflare et al)
