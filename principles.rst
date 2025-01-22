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

2.1 Security Requirements
----------------------------

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
our credit card to some impostor.

Closely related to authentication is *integrity*. It is important not only that
we know who we are talking to, but that we can verify
that the data sent across our connection has not been modified by some
adversary in transit.

The preceding requirements also suggest that we must have a concept of
*identity*. That is, we need a system by which the entities involved
in communication, often called *principals*, can be securely
identified. As we will discuss later, this problem is harder to solve
than it might first appear. How can we know that a website we are
communicating with actually represents the business with whom we wish
to communicate? Or how does a banking system know that the person
behind a particular request is actually the account holder?

Just as we are concerned that an adversary might access our data in
transit to eavesdrop on or modify it, we also need to be concerned
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
rendered those computers unable to function. Networks provide a means
by which data can be amplified by replication, allowing large volumes
of traffic to be sent to the target of a DoS attack; thus it has
become necessary to develop means to mitigate such attacks.

2.2 Broader System Requirements
-------------------------------------

Understanding the requirements of a secure system is important, but
security is just one of many requirements placed on any system.  It's
also important for systems to be reliable, available, scalable, performant,
usable, manageable, evolvable, observable, and so on.

These system requirements are sometimes indistinguishable from
security. For example, a retail system brought down by a security
breach is just as unavailable as one brought down by a hardware
failure or a software bug. Similarly, a video delivery system designed
to scale up to handle a flash crowd may also able to scale to fend off
a DoS attack. In practice, it can be difficult to pinpoint the root
cause of a failure (was it malicious or benign) and the root source of
workload (was it malicious or legitimate). The main difference is that
it is generally possible to provision for legitimate workload and
benign failures, which can be measured and predicted (although not
always accurately), whereas malicious attacks are often less
predictable (although they should be expected).

Where the broader set of requirements become interesting is when they
are in conflict with each other, and tradeoffs become necessary.  For
example, making a system secure can add latency to every transaction.
Similarly, keeping a system secure puts additional burden on users
(making it less usable), with the flip side being that those users are
often the weakest link in the security strategy. These tradeoffs do
not need to force an either-or scenario—clearly, designing for both
security and usability is desirable—but no requirement is absolute,
or we'd simply secure a system by denying access to all users.

For our purposes, the main takeaway is that security is unique in that
it is not just a requirement *for* a system, but it is often best
viewed as a system in an of itself, one that must be usable,
scalable, manageable, evolvable, observable, available, reliable, and
so on.


2.3 Principles for Secure Systems
---------------------------------

Given how long people have been trying to build secure computing
systems, including networked systems, there has been plenty of
time to develop some principles that improve the chances that the
system remains secure. We will outline some of the most well-known
principles here and the following chapters contain examples of how
those principles have been applied in practice.

2.3.1 Defense in Depth
~~~~~~~~~~~~~~~~~~~~~~
As we have noted, one of the central challenges in security is that we
never know if we have done enough. Much as we try to defend against
all possible attacks, there is no way to be sure that we've thought of
everything. This is what we mean by saying that security is a negative
goal: we aim to be sure that a set of things cannot happen, but we can
never quite be sure that all vulnerabilities have been found and
mitigated. This leads to the idea of *defense in depth*: layer upon
layer of defense, so that even if one layer is penetrated, the next
layer is unlikely to be. Only by getting through all the layers of
defense will an attacker be able to achieve their goal (of stealing
our data, for example). The hope is that with enough layers of
defense, the odds of an attacker penetrating all of them becomes
vanishingly small.

As a simple example, a corporation might make use of a VPN (virtual
private network) to ensure that only authorized users can access
corporate servers, and that when they do so over the Internet, their
traffic is encrypted. However, this single layer of defense is prone
to several forms of attack, such as the presence of malware on the
remote user's computer, or compromise of the remote user's
credentials. Thus, additional layers of defense are needed, such as
internal firewalls between different corporate systems; tools to
detect, remediate, and prevent malware on the remote users' systems;
and multi-factor authentication to protect against compromised user
credentials. This is just a short list of defensive measures that are
commonly used, and would not on their own be considered
sufficient. But the point to note here is the use of many overlapping
layers of defense to raise the bar high enough to thwart the majority
of attacks.

The fact that we read about breaches in which attackers succeed in
gaining access to corporate systems and data on a regular basis might
suggest that the battle is being lost. Certainly the challenges in
defeating determined attackers are substantial. However, it is surprising how
frequently it turns out that a well-publicized attack has succeeded
because some relatively common defensive measure, such as multi-factor
authentication, was not put in place correctly.

2.3.2 Principle of Least Privilege
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The principle of least privilege has a long history in computer
science, having been proposed by Saltzer and Schroeder in 1975. The
principle states:

  "Every program and every user of the system should operate using the
  least set of privileges necessary to complete the job."

A common example of this principle in practice is to avoid running
anything as root on Unix-like systems unless absolutely necessary.

In the context of networking, this principle implies that applications
which access the network should only have access to the set of
resources needed to do their jobs.

.. feel like there is more detail to provide here.

Interestingly, Saltzer and Schroeder explicitly mention "firewalls" in
the section of their paper on least privilege, using the analogy from
the physical world (a wall to prevent the spread of fire) before the
concept of network firewalls had been invented. As we will discuss
later, it turns out that the widespread use of network firewalls for
most of their history *failed* to follow the principle of least
privilege, in that it is common to find large "zones" of a network
where all machines have access to each other, even though this access
is not actually required for the machines to do their jobs. Addressing
this shortcoming required some innovations in the design of firewalls
that arrived only in the last decade or so.

2.3.3 Open Design
~~~~~~~~~~~~~~~~~

Another principle codified by Saltzer and Schroeder is that of open
design. It states that the mechanisms and
algorithms that are used to implement security should be open, not
secret. The idea is that rather than trying to keep something as large
and complex as an encryption algorithm secret, it is better for that
algorithm to be published and only the key(s) be secret. There are two
reasons for this principle:

* It is hard to keep an algorithm secret, especially if it is in
  widespread use as is the case with encryption on the Internet;
* Making security mechanisms robust against all forms of attack is, as
  we have discussed, difficult. Thus it is better to have wide
  scrutiny of these mechanisms to expose weaknesses that may then be
  rectified.

The history of computer security is filled with cautionary tales
related to this principle. In the cases where the principle is
followed, subtle bugs in protocol design or implementation have been
exposed and patches rolled out to mitigate them. Heartbleed, a bug in
the widely used open source implementation of SSL, is a famous
example. The consequences of the bug were serious, with as many as
half a million Web servers being impacted, but it was a positive thing
that the bug was found, reported, and remediated quickly.

If this principle is not followed, a design that is believed to be
secret may in fact have been compromised (e.g. by reverse
engineering), or may have flaws that have gone unreported but are
nevertheless being exploited.

Another way to state this principle is "minimize secrets".  For
example, rather than trying to keep an entire algorithm secret, only
keep secret the key that is used to decrypt with the algorithm. It is
much easier to replace a key that has been compromised than to replace
an entire algorithm.

2.3.4 Fail-Safe Defaults
~~~~~~~~~~~~~~~~~~~~~~~~

The idea behind this principle is the default settings of a system are
the ones most likely to be used, so by default, undesired access
should be disabled. It then takes an explicit action to enable
access. This is a principle that dates back at least to 1965 according to
the Saltzer and Schroeder paper.

It turns out that the design of the Internet really doesn't follow
this approach. The datagram delivery model of the Internet, by
default, allows packets from anywhere to be sent anywhere. So to the
extent that sending a packet to a system can be defined as accessing
the system, the Internet's default behavior does not provide fail-safe
defaults. Efforts to revert to a more secure default behavior include
such old ideas as network firewalls and virtual private networks,
along with more modern approaches such as microsegmentation and
zero-trust architectures.  We will discuss these developments in a later chapter.

2.3.5 Least Common Mechanism
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This principle states that the amount of mechanism that is common to
more than one user should be minimized. The principle was originally
articulated by Gerald Popek in 1974, where in the context of operating
system design, he argued in favor of minimizing the number of
mechanisms put in the OS kernel. A failure (or security breach) of
such a shared mechanism would impact all users, and so it is better to
provide it in some other way, such as a library running the user's
address space. The failure of such a mechanism then negatively impacts
only that user, and not all users.

The counter argument is that any mechanism that requires privileged
access—e.g., accessing an I/O device or modifying a shared data
structure—must be implemented in a common layer (i.e., the kernel),
but even then, care should be taken to decompose the mechanism so as
to minimize the sub-pieces that are common. Library OSs designed during
the 1990s took this argument to its logical conclusion, reducing the
"kernel" to its bare minimum, with most functionality one typically
expects from an OS running as a library co-located with the
application.\ [#]_

.. [#] While strictly adhering to the principle of least common
   mechanism, Library OSs are not widely adopted in practice,
   illustrating that security is only one of many system requirements
   taken into consideration.

.. admonition:: Further Reading

  G. Popek. `A Principle of Kernel Design
  <https://dl.acm.org/doi/pdf/10.1145/1500175.1500361>`__.  Security
  Kernel Panel at the AFIPS '74 National Computer Security Conference
  and Exposition. May 1974.

  D. Engler, F. Kaashoek, and J. O'Toole Jr. `Exokernel: an Operating
  System Architecture for Application-level Resource Management
  <https://dl.acm.org/doi/pdf/10.1145/224057.224076>`__.  SIGOPS
  Symposium on Operating System Principles. December 1995.

  J. Saltzer, D. Reed, and D. Clark. `End-to-End Arguments in System
  Design <https://dl.acm.org/doi/abs/10.1145/357401.357402>`__.  ACM
  Transactions on Computer Systems. November 1984.

In an OS setting, the principle of least common mechanism is related to
the principle of least privilege since the common platform (kernel)
runs with greater privilege. This is because minimizing the number of
mechanisms that require elevated kernel privilege also minimizes the
privilege required across all mechanisms that make up a system. In a
network setting, the principle is related to the
end-to-end-argument. That is, it is best to avoid putting functions
such as encryption into the network when the user is likely to need
end-to-end encryption anyway.

2.3.6 Design for Iteration
~~~~~~~~~~~~~~~~~~~~~~~~~~

Given what we have said about the difficulty of knowing that a system
is secure, a useful design principle is to accept that we will need to
iterate, and design for it. A good example of this is in the choice of
particular cryptographic algorithms for integrity protection or
encryption. These algorithms are often found to be insufficiently
strong after some number of years, perhaps due to a weakness in the
underlying mathematics, or breakthroughs in algorithms, or just the
steady improvement in computing power that happens over time. Thus,
any protocol that is developed that depends on such an algorithm
should be designed such that a change of algorithm is an expected
behavior. We see this in protocols such as Transport Layer Security
(TLS) which includes a set of procedures by which two participants
negotiate the cryptographic algorithms to be used.

Recent developments in quantum computing have raised the issue that
many existing forms of cryptographic algorithm may need to be
replaced. While the timeframe in which such a change will be needed
remains a subject of debate, the safe choice is to accept that
cryptographic algorithms will periodically need to be replaced.

2.3.7 Audit Trails
~~~~~~~~~~~~~~~~~~

Part of dealing with the impossibility of covering all possible
security threats is to accept that sometimes we need to analyze what
has gone wrong. This leads to the idea that security needs to be
auditable (observable). For example, it will be easier to conduct a
post-mortem of a breach involving compromised login credentials if
every login attempt is logged, along with information such as whether
the login came over a VPN, what IP address was used, and so
on. Similarly it is very hard to prevent insider attacks, but suitable
logging might both make it easier to detect such attacks quickly and
to deter those who might undertake them.

In a different vein, consider the design of secure protocols. The
specification for TLS (transport layer security) describes a large
number of error conditions that may trigger alerts, and recommends the
logging of all such alerts. Such logging would help in understanding
if the protocol was subject to an attack that involved incorrect or
unexpected messages. Given the complexity of negotiations that go on
in security protocols (to establish cryptographic algorithms and
parameters, for example) it is wise to assume that these may have
subtle bugs, and a good set of audit tools will enable any such bugs
to be detected and then remedied.

Of course, the audit mechanisms themselves must be designed to be
secure. A determined attacker will, in all likelihood, try to erase their tracks,
so logging for audit purposes cannot just be an afterthought; it has
to be part of the design of a secure system.

2.4 Best Practices
---------------------

Design principles help guide how a system is architected to be secure,
but an architecture has to be implemented by an engineering team and
run on a day-to-day basis by an operations team. Mistakes in the
implementation or operational practices render the best intentions
impotent. To this end, software companies typically establish a set of
best practices for the the entire software lifecycle, from design to
coding, deployment, and operation.

These practices start by establishing the security metrics that the
end result is to be judged by, along with the processes used to review
designs and test compliance. On the development side, emphasis is
placed on using cryptography standards and the best available
protocols, platforms, and languages. It is **not** an engineer's job
to reinvent the security mechanisms described in this book, but
rather, to know what mechanisms are available and how to use
them. Once a system is deployed and operational, the emphasis is on
proactively monitoring the system for anomalous behavior and
establishing an incident response plan to deal with suspected attacks.
Continuously collecting data about a running system is commonplace,
and using AI/ML to respond to detect (and potentially respond to)
attacks is becoming increasingly common.

The details of these procedures is beyond the scope of this book,
except as we note, that they depend on engineers and operators being
well-informed about available security mechanisms. For an example of
industry practices, we recommend Microsoft's Security Development
Lifecycle (SDL) practices.

.. admonition:: Further Reading

  `Microsoft Security Development Lifecycle (SDL)
  <https://www.microsoft.com/en-us/securityengineering/sdl>`__.


2.5 Summary
-----------

Just as we can never be quite sure that we have covered all possible
vectors of attack against a system, there is no hard limit to the set of
principles that can be applied to developing secure systems. The
principles covered above include several that were drawn from the
influential paper by Saltzer and Schroeder from 1975. That is the same
Saltzer whose book (with Kaashoek) we referred to in Chapter 1. The
fact that many of the principles from the 1975 paper reappear in the
2009 is probably a sign that Saltzer had some confidence that these
principles have stood the test of time. We recommend reading the
entire paper.

.. admonition:: Further Reading

  Jerome Saltzer and Michael Schroeder. `The Protection of Information
  in Computer Systems
  <http://web.mit.edu/Saltzer/www/publications/protection/index.html>`__. In
  Proceedings of the IEEE, 1975.







