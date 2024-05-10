Chapter 1:  Introduction
=========================


.. New effort from Bruce


Security of computer systems has been a focus for system designers for
as long as we have had time-shared computers. If two users can share a
computer, then it is necessary to have protections in place to limit
the impact one user can have on another. For example, one user should
not generally be able to read the data of another user just because
they run code on the same system. At the same time, when one user
*wants* to share data with another, the operating system needs to
support that in a controlled way. Similarly, multi-user systems ensure
that malicious or poorly written code from one user cannot interfere
with the operation of another user's programs. 

Computer networks are, like multi-user computers, shared
resources, and similar requirements apply. One network user should not
be able to interfere with another user's traffic. Networks are often
used so that specific resources can be shared among users. And in general,
a user sending data across a network wants that data to be protected
from unauthorized modification or eavesdropping.

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

Ensuring the security of end systems does not come close to addressing the entire set of
security issues that exist in a computer network. For example, an
attacker with access to a link, switch or router somewhere in the network
has the potential to read or modify packets passing
through that point. Furthermore, by
connecting computers to a global network, the opportunity to exploit
vulnerabilities in the code running on those end systems is opened up
to a much greater---potentially global---set of actors.

Thus we can think of network security as having two main
thrusts. First, we need to address the security challenges of a
shared, globally distributed network. Second, we need to address the
challenges of connecting end systems, which run imperfect software, to
a global set of actors, some of whom are bound to be malicious.

For an interesting retrospective view on system security, and some
commentary on how far we still have to go, we recommend
the paper on Multics from Karger and Schell.

.. admonition:: Further Reading

   P. Karger and R. Schell. `Thirty Years Later: Lessons from the
   Multics Security
   Evaluation. <https://www.acsac.org/2002/papers/classic-multics.pdf>`__.
   Annual Computer Security Applications Conference (ACSAC) 2002.


An early example of a security failure in the Internet serves to
highlight the breadth of the challenges included in the term "network
security". The Morris worm was the first large-scale attack on the
Internet, launched in 1988 when the Internet was largely limited to
universities and research institutions. While it was made possible by
the fact that the Internet of that era generally allowed packets from any source
to any destination, it was also dependent on a number of
vulnerabilities in the software running on the end systems connected
to the Internet. Like many future attacks, the Morris worm exploited
multiple vulnerabilities, including weak or default passwords, a buffer
overflow bug in a then widely-used software tool, and a security hole in
the sendmail program. There is a comprehensive analysis of the worm's
operation in the report from Donn Seeley written soon afterwards.

.. admonition:: Further Reading

  Donn Seeley. `A Tour of the
  Worm. <http://www.cs.unc.edu/~jeffay/courses/nidsS05/attacks/seely-RTMworm-89.html>`__.

What we present in this book is a systems perspective on the
security of computer networks. Our focus is on how to create networks
that meet certain security objectives, such as protection against
eavesdropping and modification of data in transit. The systems
approach requires us to look at the entire system: the network
components and the end systems connected by the network, both hardware
and software. 

A Short History of Internet Security
------------------------------------

The Internet architecture was initially created with essentially no
security features. This was not because the inventors, implementors and
architects were unaware of security issues, but rather because there
were other, more pressing goals. As Vint Cerf, the co-inventor of
TCP/IP said: "getting this thing to work at all was
non-trivial”. David Clark, the architect of the Internet, has said
"it’s not that we didn’t think about security…we thought we could
exclude [untrustworthy people].”

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
was. By the early 1990s the first firewalls had appeared, allowing the
default "accept any packet from anywhere" behavior of the Internet to
be changed. These devices filtered packets based on information in the
TCP and IP headers, and could be implemented in both hosts and
routers. By 1994 they were common enough that applications such as FTP
(the file transfer protocol) adapted to work with them.

Also in the early 1990s, the Internet was growing quickly enough to make it clear
that IP version 4 (IPv4), with 32-bit addresses, would eventually run
out of address space. The effort to create a new version of IP, known
as IPng (next generation) before being officially labeled as IPv6, had
a much larger scope than a simple increase in the address space. There
was a sense that this was perhaps the last opportunity to
significantly change the IP layer, and thus the time to address
perceived shortcomings of the Internet. High on the list of such
shortcomings to be addressed was security.

The security features that were proposed for IPv6 included headers to
support encryption, message integrity and authentication. However it
became clear that such features did not require a new version of IP,
only a way to add optional information to the packet
header, and so these capabilities also made their way into IPv4. These
extensions became known collectively as IPSEC (IP security) and are
described in several dozen RFCs.

It is worth noting that even if IPSEC had
existed in 1988, it would probably have had minimal impact on the
spread of the Morris Worm, because the worm spread among
hosts that were *supposed* to connect to each other (e.g., to exchange
email using the sendmail program). Encrypting and authenticating traffic
between hosts doesn't prevent the spread of malware
if the end-systems have the sorts of
weaknesses exploited by the Morris worm.

The rise in popularity of the World Wide Web in the 1990s created the
demand for security features at the transport layer to support
applications such as e-commerce. This lead to the creation of SSL
(secure sockets layer) which was superseded by TLS (transport layer
security), both of which provided confidentiality and authentication
at the transport layer.

Another aspect of securing the Internet that started to receive
attention in this period was the security of its infrastructure. One
such piece of infrastructure is the domain name system (DNS). DNS
replaced static host-to-address mapping files in the 1980s and subsequently
become critical to the operation of the Internet. Clearly the
information served up by DNS needs to be robust against manipulation
by adversaries, and hence there has been an multi-decade effort to add
security to the DNS. The fact that this continues to roll on
illustrates some of the challenges in making incremental updates to
the distributed infrastructure of the Internet.

The Internet's routing system is at least as important as DNS, and
similarly lacked any security provisions in its original design. Not
only do we need to be concerned about modification of routing messages
in transit, but it has historically been all too easy to simply send
incorrect routing updates in BGP, e.g., advertising a good route to
some prefix from an autonomous system that has no such route. Thus,
securing BGP has likewise proven to be a multi-decade, incremental task.

This is by no means a complete history of Internet security but it
gives some sense of the scope of the problems faced. Some further
perspective on this history, and the factors that contributed to
Internet's lack of security, can be found in the following series
of articles from the Washington Post.


.. admonition:: Further Reading

  C. Timberg. `A Net of Insecurity
  <https://www.washingtonpost.com/sf/business/2015/05/30/net-of-insecurity-part-1/>`__.
  The Washington Post, May 30, 2015. 

Trust and Threats
-----------------

A discussion of security often begins with an analysis of the *threat
landscape*. That is, what are the threats that our system is likely to
be exposed to and which we hope to mitigate. This is one of the great
challenges in developing a security strategy: how do we know when we
have identified all the likely threats? Some may be obvious, such as
eavesdropping on unencrypted traffic sent over a shared medium, but
less obvious threats are constantly being identified.

It is common to talk about security as a "negative goal". That is, we
are trying to ensure that a set of undesirable things cannot
happen. That set of undesirable things is large, making security
particularly challenging. Over the years, a number of principles have
been developed to try to manage this challenging landscape; we will
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
that security techniques impose on our system, we can also consider
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
security in a broad context (e.g., airports), not just systems and
networks. Nevertheless it provides some useful guidelines that are
applicable to system security.

  
.. admonition:: Further Reading

  B. Schneier. Beyond Fear: Thinking Sensibly About Security in an
     Uncertain World. Copernicus Books, 2003.



.. from the original book - need some cleanup to splice with the above text

Computer networks are typically a shared resource used by many
applications representing different interests. The Internet is
particularly widely shared, being used by competing businesses, mutually
antagonistic governments, and opportunistic criminals. Unless security
measures are taken, a network conversation or a distributed application
may be compromised by an adversary.

Consider, for example, some threats to secure use of the web. Suppose
you are a customer using a credit card to order an item from a website.
An obvious threat is that an adversary would eavesdrop on your network
communication, reading your messages to obtain your credit card
information. How might that eavesdropping be accomplished? It is trivial
on a broadcast network such as an Ethernet or Wi-Fi, where any node can
be configured to receive all the message traffic on that network. More
elaborate approaches include wiretapping and planting spy software on
any of the chain of nodes involved. Only in the most extreme cases
(e.g.,national security) are serious measures taken to prevent such
monitoring, and the Internet is not one of those cases. It is possible
and practical, however, to encrypt messages so as to prevent an
adversary from understanding the message contents. A protocol that does
so is said to provide *confidentiality*. Taking the concept a step
farther, concealing the quantity or destination of communication is
called *traffic confidentiality*—because merely knowing how much
communication is going where can be useful to an adversary in some
situations.

Even with confidentiality there still remains threats for the website
customer. An adversary who can’t read the contents of your encrypted
message might still be able to change a few bits in it, resulting in a
valid order for, say, a completely different item or perhaps 1000 units
of the item. There are techniques to detect, if not prevent, such
tampering. A protocol that detects such message tampering is said to
provide *integrity*.

Another threat to the customer is unknowingly being directed to a false
website. This can result from a Domain Name System (DNS) attack, in
which false information is entered in a DNS server or the name service
cache of the customer’s computer. This leads to translating a correct
URL into an incorrect IP address—the address of a false website. A
protocol that ensures that you really are talking to whom you think
you’re talking is said to provide *authentication*. Authentication
entails integrity, since it is meaningless to say that a message came
from a certain participant if it is no longer the same message.

The owner of the website can be attacked as well. Some websites have
been defaced; the files that make up the website content have been
remotely accessed and modified without authorization. That is an issue
of *access control*: enforcing the rules regarding who is allowed to do
what. Websites have also been subject to denial of service (DoS)
attacks, during which would-be customers are unable to access the
website because it is being overwhelmed by bogus requests. Ensuring a
degree of access is called *availability*.

In addition to these issues, the Internet has notably been used as a
means for deploying malicious code, generally called *malware*, that
exploits vulnerabilities in end systems. *Worms*, pieces of
self-replicating code that spread over networks, have been known for
several decades and continue to cause problems, as do their relatives,
*viruses*, which are spread by the transmission of infected files.
Infected machines can then be arranged into *botnets*, which can be used
to inflict further harm, such as launching DoS attacks.
