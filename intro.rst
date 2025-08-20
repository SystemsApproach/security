Chapter 1.  Introduction
=========================


Security has been a focus of system designers for
as long as we have had time-shared computers. If two users can share a
computer, then it is necessary to have protections in place to limit
the impact one user can have on another. For example, one user should
not generally be able to read the data of another user just because
they run code on the same system. At the same time, when one user
*wants* to share data with another, the operating system needs to
support that in a controlled way. Similarly, multi-user systems ensure
that malicious or poorly written code from one user cannot interfere
with the operation of another user's programs.

Computer networks are, like multi-user computers, shared resources,
and similar requirements apply. One network user should not be able to
interfere with another user's traffic, and in general, a user sending
data across a network wants that data to be protected from
unauthorized modification or eavesdropping. At the same time, networks
are typically used so that specific resources can be shared among
users.

As we will see, the security of computer systems and the security of
computer networks are closely related topics. And just as a
traditional networking book needs to pay attention not only to the
operation of switches and routers but also to a whole stack of
software that runs in end systems, so the topic of network security
demands that we look at both the devices that make up the network and
the end systems that connect to it.

The Internet was created to allow users in one location to
access computing resources in another. Those systems had their own
security measures in place. For example, if you wanted to use the Internet
to log in to a remote computer, you would (usually) need to authenticate
yourself to that remote system (e.g., with a user name and password) before
gaining access to any resources on that system.

Ensuring the security of end systems, while important, does not come
close to addressing the entire set of security issues that exist in a
computer network. For example, an attacker with access to a link,
switch or router somewhere in the network has the potential to read or
modify packets passing through that point. Furthermore, failures of
end system security are exposed when we connect them to networks. By
connecting computers to a global network such as the Internet, the
opportunity to exploit vulnerabilities in the code running on those
end systems is opened up to a much greater—potentially global—set
of actors.

Thus, we can think of network security as having two main
thrusts. First, we need to address the security challenges of a
shared, globally distributed network. Second, we need to address the
challenges of connecting end systems running imperfect software to
a global set of actors, some of whom are bound to be malicious.

For an interesting retrospective view on computer system security, and some
commentary on how far we still have to go, we recommend the paper
looking back on the history of security in Multics from Karger and
Schell.

.. admonition:: Further Reading

   P. Karger and R. Schell. `Thirty Years Later: Lessons from the
   Multics Security
   Evaluation. <https://www.acsac.org/2002/papers/classic-multics.pdf>`__.
   Annual Computer Security Applications Conference (ACSAC) 2002.


An early example of a security failure in the Internet serves to
highlight the breadth of the challenges included in the term "network
security". The Morris worm, the first large-scale attack on the
Internet, was launched in 1988 when the Internet was largely limited to
universities and research institutions. While it was made possible by
the fact that the Internet of that era generally allowed packets from
any source to any destination, it was also dependent on a number of
vulnerabilities in the software running on the end systems connected
to the Internet. Like many future attacks, the Morris worm exploited
multiple vulnerabilities. In this case they included weak or default
passwords, a buffer overflow bug in a widely-used software tool,
and a security hole in the Sendmail program. There is a comprehensive
analysis of the worm's operation in the report from Donn Seeley
written soon afterwards.

.. admonition:: Further Reading

  D. Seeley. `A Tour of the
  Worm <http://www.cs.unc.edu/~jeffay/courses/nidsS05/attacks/seely-RTMworm-89.html>`__.

What we present in this book is a systems perspective on the
security of computer networks. Our focus is on how to create networks
that meet certain security objectives, such as protection against
eavesdropping and modification of data in transit. The systems
approach requires us to look at the entire system: the network
components and the end systems connected by the network, as well as both hardware
and software.

1.1 A Short History of Internet Security
----------------------------------------

The Internet architecture was initially created with essentially no
security features. This was not because the inventors, implementers and
architects were unaware of security issues, but rather because there
were other, more pressing goals. As Vint Cerf, the co-inventor of
TCP/IP said: *"getting this thing to work at all was
non-trivial.”* David Clark, the architect of the Internet, has said
*"it’s not that we didn’t think about security…we thought we could
exclude [untrustworthy people].”*

The Internet's architecture is often characterized as an hourglass,
with IP being the narrow waist that sits between a wide range of
networks below and a wide range of upper layer protocols and
applications above. The narrowness of the waist refers to its limited
functionality: global addressing and a best-effort packet delivery
model. Conspicuously missing from that waist is any sort of security
capability. The best-effort packet delivery model included a "default
open" setting: if you knew the destination address of a host, you
could send it a packet.

.. could include something about decentralization

The Morris Worm served as something of a wake-up call to the early
developers of the Internet by highlighting just how vulnerable it
was. By the early 1990s the first *firewalls* had appeared, allowing the
default "accept any packet from anywhere" behavior of the Internet to
be changed in a controlled way. These devices, which remain common
today, filter packets based on information in the TCP and IP headers,
and can be implemented in both hosts and routers. By 1994 they were
common enough that applications such as FTP (the file transfer
protocol) were adapted to work with them.

Also in the early 1990s, the Internet was growing quickly enough to
make it clear that IP version 4 (IPv4), with 32-bit addresses, would
eventually run out of address space. The effort to create a new
version of IP, known as IPng (next generation) before being officially
labeled as IPv6, had a much larger scope than a simple increase in the
address space. Among those working on IPv6 requirements, some argued
that this was perhaps the last opportunity to significantly change the
IP layer, and thus the time to address perceived shortcomings of the
Internet. High on the list of such shortcomings to be addressed was
security.

The security features that were proposed for IPv6 included headers to
support encryption, message integrity and authentication. However, it
became clear that such features did not require a new version of IP,
only a way to add optional information to the packet
header, and so these capabilities also made their way into IPv4. These
extensions became known collectively as *IPsec (IP security)* and are
described in several dozen RFCs. We discuss them in a later chapter.

It is worth noting that, even if IPsec had
existed in 1988, it would probably have had minimal impact on the
spread of the Morris Worm. This is because the worm spread among
hosts that were *supposed* to connect to each other (e.g., to exchange
email using the Sendmail program). Encrypting and authenticating traffic
between hosts doesn't prevent the spread of malware
if the end-systems have the sorts of
weaknesses exploited by the Morris worm.

The rise in popularity of the World Wide Web in the 1990s created the
demand for security features at the transport layer to support
applications such as e-commerce. This lead to the creation of *SSL
(secure sockets layer)* which was superseded by *TLS (transport layer
security)*, both of which provided confidentiality and authentication
at the transport layer. TLS is an important case study for network
security, and we describe it in detail in a later chapter.

Another aspect of securing the Internet that started to receive
attention in this period was the security of its infrastructure. One
such piece of infrastructure is the *Domain Name System (DNS)*,
which replaced static hostname-to-address mapping files in the 1980s, and subsequently
become critical to the operation of the Internet. Clearly the
information served up by DNS needs to be robust against manipulation
by adversaries, and hence, there has been a multi-decade effort to add
security to the DNS. The fact that this continues to roll on
illustrates some of the challenges in making incremental updates to
the distributed infrastructure of the Internet.

The Internet's routing system is at least as important as DNS, and
similarly lacked any security provisions in its original design. Not
only do we need to be concerned about modification of routing messages
in transit, but it has historically been all too easy to simply send
incorrect routing updates in BGP, the *Border Gateway Protocol*.
For example, a router might advertise a good route to
some prefix from an autonomous system that has no such route.
Securing BGP has likewise proven to be a multi-decade, incremental task.

This is by no means a complete history of Internet security but it
gives some sense of the scope of the problems faced. Some further
perspective on this history, and the factors that contributed to
Internet's lack of security, can be found in the following series
of articles from the Washington Post, in which many of the Internet's
pioneers are interviewed.


.. admonition:: Further Reading

  C. Timberg. `A Net of Insecurity
  <https://www.washingtonpost.com/sf/business/2015/05/30/net-of-insecurity-part-1/>`__.
  The Washington Post, May 30, 2015.

1.2 Trust and Threats
----------------------

A discussion of security often begins with an analysis of the *threat
landscape*. That is, what are the threats that our system is likely to
be exposed to and which we hope to mitigate. This is one of the great
challenges in developing a security strategy: how do we know when we
have identified all the likely threats? Some may be obvious, such as
eavesdropping on unencrypted traffic sent over a shared medium, but
less obvious threats are constantly being identified. Furthermore,
there are different *threat actors* with different motivations,
ranging from those who enjoy the technical challenge of finding
vulnerabilities, to criminals looking to obtain valuable information
such as credit card details, to government actors looking to perform
surveillance or interfere in elections.

It is common to talk about security as a "negative goal". That is, we
are trying to ensure that a set of undesirable things cannot
happen. That set of undesirable things is large, making security
particularly challenging. Over the years, a number of principles have
been developed to help manage this challenging landscape; we will
consider many of them in subsequent chapters.

Noted security expert Bruce Schneier points out in his book
"Beyond Fear" that security is also a matter of making trade-offs. You not
only have to identify the threats that you wish to defend against, but
also to decide what costs you are willing to incur in mounting
that defense. For example, encrypting every packet sent by a computer
in 1970 imposed such a high computational cost as to be barely
practical or required special hardware; today it is routine that every
packet sent between a web browser and server is encrypted. Thus, the
trade-offs around encryption are different than they were when the
Internet was originally designed. And just as we consider the costs
that security techniques impose on our system, we must also consider
the costs they impose on adversaries. Much of security consists of
finding ways to make those costs highly asymmetric, so that they are
much higher for the adversary than for those seeking to protect their
systems and information.


Schneier describes a set of questions to be addressed in developing a
security strategy:

* Step 1: What assets are you trying to protect?
* Step 2: What are the risks to these assets?
* Step 3: How well does the security solution mitigate those risks?
* Step 4: What other risks does the security solution cause?
* Step 5: What costs and trade-offs does the security solution impose?

Schneier's book is targeted at a general audience, addressing
security in a broad context (e.g., airports), not just computing systems and
networks. Nevertheless, it provides some useful guidelines that are
applicable to system security.


.. admonition:: Further Reading

  B. Schneier. Beyond Fear: Thinking Sensibly About Security in an
     Uncertain World. Copernicus Books, 2003.

Finally, it is important to recognize that trust and threats are two
sides of the same coin. A threat is a potential failure scenario that
you design your system to avoid, and trust is an assumption you make
about how external actors and internal components you build upon will
behave. For example, if you plan to transmit messages over Wi-Fi on an
open campus, you would likely identify an eavesdropper that can
intercept messages as a threat (and adopt some of the methods
discussed in this book as a countermeasure). But if you are planning
to transmit messages over a fiber link between two machines in a
locked datacenter, you might trust that channel is secure, and so take
no additional steps.  Every system makes trust assumptions, even if it
as simple as trusting the computer you just bought from a reputable
vendor does not forward your data to an adversary. The key is
to be as explicit as possible about those assumptions, because they
may change over time.



1.3 Threats to Network Security
-------------------------------



.. from the original book chapter - somewhat edited to follow the above text

Computer networks are, as we noted above, invariably a shared
resource. They are used by many applications representing different
interests. The Internet is particularly widely shared, being used by
competing businesses, mutually antagonistic governments, and
opportunistic criminals. Most of the world's information is stored on
systems connected to the Internet. Unless effective security measures
are in place, a network conversation, a distributed application, or an
end-system storing sensitive data may be compromised by an
adversary. Critical systems ranging from healthcare delivery to the
power grid are at risk of disruption from various forms of attack.


A simple and familiar example of threats and mitigations is the secure
use of the web. Suppose you are a customer using a credit card to
order an item from a website.  An obvious threat is that an adversary
could eavesdrop on your network communication, reading your messages
to obtain your credit card information. How might that eavesdropping
be accomplished? It is trivial on a broadcast network such as an
Ethernet or Wi-Fi, where any node can be configured to receive all the
message traffic on that network. More elaborate approaches include
wiretapping or planting spy software on any of the chain of nodes
involved. The insertion of monitoring software might be performed by
an operator with physical or remote access to a router (e.g., an
employee of an Internet service provider). A vulnerability in the
router's software might be exploited by an attacker to gain remote
access. And in recent years there have been examples of "supply chain
attacks" in which malicious software is inserted in some code, either
open source or proprietary, that is subsequently used in another
vendor's products. In other words, there are a *lot* of ways that the
data in flight from your browser to the website might end up in the
hands of an attacker.

While various steps can be taken to secure the devices along the path
traveled by your data, it is relatively straightforward today to
encrypt all messages such that even if an adversary has access to the
data, they are unable to *understand* the message contents. A protocol that does
so is said to provide *confidentiality*. Taking the concept a step
farther, concealing the quantity and destination of communication is
called *traffic confidentiality*—because merely knowing where
traffic is going, and how much, can be useful to an adversary in some
situations.

Confidentiality alone is not sufficient. An adversary who can’t read
the contents of your encrypted message might nevertheless be able to
modify it. By changing a few bits, it might be possible to order a
completely different item or perhaps 1000 units of the item. There are
techniques to detect, if not prevent, such tampering. A protocol that
detects such message tampering is said to provide
*integrity*. Similarly, an attacker might capture a message and
send it again at another time, which might cause a duplicate purchase,
for example. This is called a *replay attack* and prevention of such
attacks is a common requirement of security protocols.

Another threat to the customer is unknowingly being directed to a
false website. This can result from a Domain Name System (DNS) attack,
in which false information is entered in a DNS server or the name
service cache of the customer’s computer. This leads to translating a
correct URL into an incorrect IP address—the address of a false
website.  It is also common to create websites with domain names that
look like they might be legitimate. A protocol that ensures that you
really are talking to whom you think you are is said to provide
*authentication*. Authentication is separate from but also requires integrity, since it is
meaningless to say that a message came from a certain participant if
it is no longer the same message.

The owner of the website can be attacked as well. Some websites have
been defaced; the files that make up the website content have been
remotely accessed and modified without authorization. That is an issue
of *access control*, enforcing the rules regarding who is allowed to do
what. Websites are also subject to *Denial-of-Service (DoS)*
attacks, during which would-be customers are unable to access the
website because it is being overwhelmed by bogus requests. Ensuring a
degree of access is called *availability*.

In addition to these issues, the Internet has notably been used as a
means for deploying malicious code, generally called *malware*, that
exploits vulnerabilities in end systems. *Worms*, of which the Morris
worm is a famous example, are pieces of
self-replicating code that spread over networks.
*Viruses* differ slightly from worms, in that they are spread by the transmission of infected files.
Once infected, machines can then be arranged into *botnets*, in which
a set of compromised machines are harnessed together
to inflict further harm, such as launching DoS attacks.

We will look further into these various classes of threats and the
measures developed to mitigate them in the following chapters. For a
solid introduction to system security we recommend the chapter below
from Saltzer and Kaashoek. Their perspective is grounded in *Operating
System (OS)* design, which is the context for much of the foundational
work in security. In that setting, finding the right balance between
sharing (so users can access each other's files) and security (so you
can keep users from accessing your private data) is a central
challenge. Security is easiest when the answer is always "no".


.. admonition:: Further Reading

  J. Saltzer and F. Kaashoek. `Principles of Computer System Design: An
     Introduction. Chapter 11
     <https://ocw.mit.edu/courses/res-6-004-principles-of-computer-system-design-an-introduction-spring-2009/pages/online-textbook/>`__. Morgan
     Kaufmann Publishers, 2009.

