Chapter 1:  Introduction
=========================


.. New effort from Bruce


Security of computer systems has been a focus for system designers for
as long as we have had time-shared computers. If two users can share a
computer, then it is necessary to have protections in place to limit
the impact one user can have on another. For example, one user should
not generally be able to read the data of another user just because
they run code on the same system. A multi-user system should ensure
that malicious or poorly written code from one user cannot interfere
with the operation of another user's programs.

Computer networks are, like multi-user computers, shared
resources, and similar requirements apply. One network user should not
be able to interfere with another user's traffic. And in general,
a user sending data across a network wants that data to be protected
from unauthorised modification or eavesdropping.

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
to log in to a remote computer, you would need to authenticate
yourself to that remote system (via user name and password) before
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

  Donn Seeley. `A Tour of the
  Worm. <http://www.cs.unc.edu/~jeffay/courses/nidsS05/attacks/seely-RTMworm-89.html>`__.

What we aim to cover in this book is a systems perspective on the
  security of computer networks. 


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
