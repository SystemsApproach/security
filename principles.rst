Chapter 2. Design Principles
============================

In the preceding chapter we looked at some reasons why network
security is a never-ending challenge. As security is a negative goal, we can never
be sure that we have foreseen and prevented every possible
threat. Nevertheless, there are a number of general principles that
have been developed over the decades that we can apply to improve the
overall security of our networks and systems. We will discuss these in
this chapter and see them in action in subsequent chapters.

In order to take a systematic approach to security it helps to have a
clear view of our requirements. Once we have defined those we can consider the
principles that can be applied to help us meet them.

2.1 Requirements
--------------------

The first requirement that comes to mind when thinking about sending
data across a shared network is likely to be *confidentiality*. As
noted in Chapter 1, there are many ways that an adversary might gain
access to data in transit through a network. Confidentiality is the
ability to prevent data from being read by anyone other than its
intended recipient. This, for example, is what is required to prevent
your credit card details being exposed when you enter then on a
website.

Related to the confidentiality of data is *traffic
confidentiality*. This is the idea that we don't want an adversary to
be able to determine where or to whom we are sending traffic, or in
what quantity. This presents some distinct challenges from data
confidentiality; there is no need for network devices to see our data,
but they generally must look at packet headers, which contain
destination information, to determine where to
send traffic.  

An equally important requirement in many cases is
*authentication*. This is the ability to verify that an item of data
was sent by the entity that claimed to have sent it. In the example of
e-commerce, this is what allows us to know we are connected to, say,
the website of the vendor we wish to patronize and not handing over
our credit card to some imposter.

Closely related to authentication is *integrity*. It is not only important that
we know who we are talking to, but that we can verify
that the data sent across our connection has not been modified by some
adversary in transit.

The preceding requirements also suggest a fourth: *identity*. That is,
we need a system by which the entities involved in communication,
often called *principals*, can be securely identified. As we will
discuss later,
this problem is harder to solve than it might first appear. How can we
know that a website we are communicating with actually represents the
business with whom we wish to communicate?

Just as we are concerned that an adversary might access our data in
transit to eavesdrop on or modifiy it, we also need to be concened
about *replay attacks* in which data is captured and then
retransmitted at some later time. For example, we would want to
protect against an attack in which an item added to a shopping cart
was repeatedly added again by an attacker. Thus it is a common
requirement to have some form of *replay prevention*.

A common requirement in computer system security is *access control*:
the ability to limit who has access to a system and what operations
they may perform on it. This applies not only to end systems but to
network devices such as routers and infrastructure components such
as name servers.

Finally, it is important that networks and the systems attached to
them can be protected against *denial-of-service* (DoS) attacks. The
Morris Worm was an early example of an unintentional DoS attack: as
the worm spread to more and more computers, and reinfected computers
on which it was already present, the resources consumed by the worm
rendered those computers unable to function. Networks provide a meands
by which data can be amplified by replication, allowing large volumes
of traffic to be sent to the target of a DoS attack; thus it has
become necessary to develop means to mitigate such attacks.

2.2 Principles for Secure Systems
---------------------------------

Given how long people have been trying to build secure computing
systems, including networked systems, there has been plenty of
time to develop some principles that improve the chances that the
system remains secure. We will outline some of the most well-known
principles here and the following chapters contain examples of how
those principles have been applied in practice.


.. working list of ideas
   
Least Privilege

Defense in Depth/safety net

Be Explicit

Design for Iteration

Audit

Open Design

Minimize secrets

economy of mechanism

fail-safe defaults


