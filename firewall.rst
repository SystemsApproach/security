Chapter 9. Firewalls
====================

Whereas much of this book has focused on the uses of cryptography to
provide such security features as authentication and confidentiality,
there is a whole set of security issues that are not readily addressed
by cryptographic means. For example, worms, viruses and other malware spread by
exploiting bugs in operating systems and application programs (and
sometimes human gullibility as well);  no amount of cryptography can
help you if your machine has unpatched vulnerabilities. So other
approaches are often used to keep out various forms of potentially
harmful traffic. Firewalls are one of the most common ways to do this.

To provide a little more context for this chapter, it is helpful to
understand that writing software that is not vulnerable to being
hacked is an important part of the overall security landscape. It is
also a broad topic, starting with questions about the programming
language you use (e.g., memory-safe languages like Rust are less
susceptible than, say, C), and also including a variety of OS problems
(e.g., how to efficiently enforce isolation between processes). Such
topics are outside the scope of this book, where we instead take a
network-centric view, and ask: *"What can we do in the network to
either minimize opportunities for malware to exploit vulnerable
software, or to mitigate the impact when such exploits succeed."*
Firewalls, and more generally *security appliances*, are part of the
answer. They are devices placed at strategic points throughout the
network that identify and respond to malicious traffic.

The other big-picture takeaway is that firewalls and other security
appliances illustrate the principle of defense in depth introduced in
Chapter 2. It would be ideal for all the software we run to be
bullet-proof, but for those times it isn't—e.g., when a new bug is
discovered—we need a second line of defense. Network appliances play
that role.

9.1 Basic Principles of Firewalls
-----------------------------------

The historical meaning of a firewall is a barrier to prevent the
spread of fire from one part of a building to another. Firewalls are also
present in many automotive vehicles to separate passengers from the
noisy (and possibly fire-prone) engine compartment. Saltzer and
Schroeder in 1975 applied the term, possibly for the first time in
the context of computer security, when discussing least privilege.

A network firewall is a system that typically sits between two regions
of a network, as illustrated in :numref:`Figure %s <fig-firewall>`,
protecting the flow of traffic from one region to another. A common
location would be a strategic point such as the ingress/egress point
to an enterprise network or data center. Of course, unlike a physical
firewall that blocks everything in its path, a network firewall is
really a sort of packet filter that makes decisions about exactly what
traffic is allowed to pass through it.

Historically, a firewall has been implemented as an “appliance”: a
dedicated system with one job, filtering packets. Firewalls have also
been provided as a set of features available on a router, and a
“personal firewall” may be implemented on an end-user machine. More
recently (in the last decade or so), *distributed firewalls* have
appeared in an effort to apply security policies to a greater amount
of traffic. We return to this class of firewall later in the chapter.

Firewall-based security depends on the firewall being at some sort of
choke-point. There should be no way to bypass the firewall via other
gateways, or alternate paths such as wireless connections or dial-up
connections. While the job of a firewall is to block potentially harmful traffic
("fire") for some definition of "harmful", a great deal of traffic passes through a firewall. One
common setting for a firewall is "default closed": by default it
blocks traffic unless that traffic is specifically allowed to pass
through. For example, it might filter out all incoming messages except
those addressed to a particular set of IP addresses and to particular
TCP port numbers. Given the complexity of the environments where
firewalls are used, with perhaps thousands of IP addresses and
hundreds of applications in use, the configuration of firewalls also
often becomes complex.

.. _fig-firewall:
.. figure:: figures/f08-20-9780123850591.png
   :width: 600px
   :align: center

   A firewall filters packets flowing between a site and the rest of the
   Internet.

When deployed at an ingress/egress point, a firewall divides a network
into a more-trusted zone on one side of the firewall (the "inside")
and a less-trusted zone that is "external" to the firewall. This is
useful if, for example, you do not want external users to access a particular host
or service within your site. Much of the complexity comes from the
fact that you want to allow different kinds of access to different
external users, ranging from the general public, to business partners,
to remotely located members of your organization. A firewall may also
impose restrictions on outgoing traffic to prevent certain attacks and
to limit losses if an adversary succeeds in getting access inside the
firewall (or to limit the damage of insider attacks).

The location of a firewall often happens to also be the dividing line
between globally addressable regions and those that use local
addresses. Hence, Network Address Translation (NAT) functionality and
firewall functionality often are found in the same device, even though
they are logically separate.


While the example above suggests a trusted and an untrusted network,
firewalls are often used to create multiple *zones of trust*, such as a
hierarchy of increasingly trusted zones. A common arrangement involves
three zones of trust: the internal network, the *DMZ* (“demilitarized
zone”); and the rest of the Internet. The DMZ is used to hold services
such as DNS and email servers that need to be accessible to the outside.
Both the internal network and the outside world can access the DMZ, but
hosts in the DMZ cannot access the internal network; therefore, an
adversary who succeeds in compromising a host in the exposed DMZ still
cannot access the internal network. The DMZ can be periodically restored
to a clean state.

Firewalls filter traffic based on information that can be found in the
packets passing through them. This is likely to include such fields as
IP addresses, the TCP or UDP port numbers, and so on. They are
configured with a table of values in these fields that characterize
the packets they will, and will not, forward.  Generally, each entry
in the table is a 4-tuple or 5-tuple: It gives the IP address and TCP (or UDP)
port number for both the source and destination (four fields), and it
may also include the specific value of the layer 4 protocol (TCP, UDP, etc.).

For example, a firewall might be configured to filter out (not forward)
all packets that match the following description:

.. literalinclude:: code/fwrule1


This pattern says to discard all TCP packets from port 1234 on host
198.51.100.14 addressed to port 80 on host 192.0.2.11. (Port 80 is the
well-known TCP port for HTTP.) Of course, it’s often not practical to
name every source host whose packets you want to filter, so the patterns
can include wildcards. For example,

.. literalinclude:: code/fwrule2


says to filter out all packets addressed to port 80 on 192.0.2.11,
regardless of what source host or port sent the packet. Notice that
address patterns like these require the firewall to make
forwarding/filtering decisions based on level 4 port numbers, in
addition to level 3 host addresses. For this reason, network
layer firewalls are sometimes called *level 4 switches*.

Linux has a firewall feature called ``ufw`` (uncomplicated firewall)
that can apply firewall rules on a host. We can implement the policy
described above with the following command:

.. literalinclude:: code/fwsnip1


Then we can check that our rule was applied correctly:

.. literalinclude:: code/fwsnip2

In the preceding discussion, the firewall forwards everything except
where specifically instructed to filter out certain kinds of packets. A
firewall could also filter out everything unless explicitly instructed
to forward it, or use a mix of the two strategies. For example, instead
of blocking access to port 80 on host 192.0.2.11, the firewall might be
instructed to block everything except access to port 25 (the SMTP mail port) on a
particular mail server, such as


.. literalinclude:: code/fwrule3


We can specify this behavior with ufw:

.. literalinclude:: code/fwsnip3


Experience has shown that firewalls are very frequently configured
incorrectly, allowing unsafe access, or breaking applications that
need access. Part of the problem is that filtering rules can overlap
in complex ways, making it hard for a system administrator to
correctly express the intended filtering. A design principle that we
discussed in Chapter 2 comes into play here: fail-safe defaults. The
application of that principle to firewalls says they should, by
default, discard all packets other than those that are explicitly
allowed. Of course, this means that some valid applications might be
accidentally disabled; the typical approach is that users of those
applications eventually notice the breakage and ask the system
administrator to make the appropriate change.

Many client/server applications dynamically assign a port to the client.
If a client inside a firewall initiates access to an external server,
the server’s response would be addressed to the dynamically assigned
port. This poses a problem: how can a firewall be configured to allow an
arbitrary server’s response packet but disallow a similar packet for
which there was no client request? This is not possible with a
*stateless firewall*, which evaluates each packet in isolation. It
requires a *stateful firewall*, which keeps track of the state of each
connection. An incoming packet addressed to a dynamically assigned port
would then be allowed only if it is a valid response in the current
state of a connection on that port.

Modern firewalls also understand and filter based on many specific
application-level protocols such as HTTP or FTP. They use
information specific to that protocol, such as URLs in the case of HTTP,
to decide whether to discard a message. When firewalls are able to
inspect payloads that are inside the TCP header (for example, to parse
an HTTP request), this is referred to as *deep packet inspection*
(DPI). Of course, DPI can be a challenge if end-to-end encryption is used.

A particular type of firewall that can interpret application traffic
is the *Web Application Firewall*. Such firewalls are often placed
directly in front of servers delivering web applications (or are
implemented as a module within the server). They inspect the
application traffic and apply filtering rules to identify and block
specific attacks that target known vulnerabilities, such as SQL
injection. When TLS is in use (as it invariably is in the modern web),
web application firewalls terminate the HTTPS connection so that the
application payload can be inspected. The open source ModSecurity
project is a widely-used example of a web application firewall.

9.2 Strengths and Weaknesses of Firewalls
-----------------------------------------

At best, a firewall protects a network from undesired access from the
rest of the Internet; it cannot provide security to legitimate
communication between the inside and the outside of the firewall. In
contrast, the cryptography-based security mechanisms described in this
chapter are capable of providing secure communication between any
participants anywhere. This being the case, why are firewalls so common?
One reason is that firewalls can be deployed unilaterally (by a
network administrator, for example), using individual
commercial products, while cryptography-based security requires support
at both endpoints of the communication. A more fundamental reason for
the dominance of firewalls is that they encapsulate security in a
centralized place, in effect factoring security out of the rest of the
network. A system administrator can manage the firewall to provide
security, freeing the users and applications inside the firewall from
security concerns—at least some kinds of security concerns. And as
noted at the start of the chapter, encryption and authentication offer
limited protection against exploitation of bugs in the operating
systems of hosts.

Unfortunately, firewalls have serious limitations. Since a firewall does
not restrict communication between hosts that are on the same side of the firewall,
the adversary who does manage to gain access to one host at a site
potentially has access to
all local hosts. How might an adversary gain access inside the firewall? The
adversary could be a disgruntled employee with legitimate access, or the
adversary’s software could be hidden in some software installed from a
USB drive or downloaded from the Web. It might be possible to bypass the
firewall by using a VPN—this has proven to be a common form of attack
in recent years.

A related problem is that any parties granted access through your
firewall, such as business partners or externally located employees,
become a security vulnerability. If their security is not as good as
yours, then an adversary could penetrate your security by penetrating
their security.


While part of the motivation for firewalls is to protect machines that
may have vulnerabilities from attack, their ability to do so is
limited. If, for example, an attacker is able to gain access to a
machine inside the firewall via one of the methods mentioned above,
they may then be able to connect to another machine inside the
firewall that contains an unpatched vulnerability, even though that
machine itself is not directly accessible through the firewall.
System administrators are expected to monitor for new vulnerabilities
and patch them, but there is always a chance the vulnerabilities
appear faster than then can be remediated. While staying up to date
with patches is a best practice, it is certainly not one that is
followed uniformly.

In Chapter 1 we discussed the threat posed by viruses, worms, and the
general category of malware. While firewalls aim to stop the spread
of malware, it can be a difficult task, since many operations that the
firewall needs to permit, such as web browsing or email delivery, can
also be used for the delivery of malware.

One approach that is used to detect malware is to search for segments of
code from known malware, sometimes called a *signature*. This approach
has its own challenges, as cleverly designed malware can tweak its
representation in various ways. There is also a potential impact on
network performance to perform such detailed inspection of data entering
a network.

Some of the limitations of firewalls are related to the assumption
that all traffic has to be funneled through a single appliance (or a
small number of them). This leads to challenges both in performance,
since so much traffic passes through a single choke point, and in
effectiveness, since there can be plenty of traffic within an
enterprise or a data center that has no need to pass through such a
choke point. These limitations have led to the development of
*distributed firewalls*, which we discuss in the following section.

9.3 Distributed Firewalls
-------------------------

A conventional firewall is implemented as a *choke point:* the network
is set up in such a way that traffic must pass through the firewall to
get from one part of the network to another. It is common to talk
about devices as being "inside" or "outside" the firewall based on
which side of that choke point they sit on. There are two implications
of such an approach. One is that any devices that sit on the same side
of the firewall are free to communicate with each other uninterrupted
by the firewall. The second is that there must be an impenetrable
barrier around the devices that are "inside" the firewall, with the
firewall being the only way to get through that barrier. This seems
appealing if you are trying to secure, say, the machines inside a
single building, with only one connection to the outside world, but it
becomes a lot less attractive if we are talking about securing all the
machines in a complex enterprise spread across many sites. Even in
simple cases, people find ways around a firewall. Once upon a time you
might have to worry about an unsanctioned dial-up connection bypassing
a firewall, while wireless networks and users at the ends of VPN
tunnels are a bigger issue in contemporary settings. In any case, the
concept of an impenetrable perimeter can be very difficult to sustain
in practice.

The fact that a perimeter firewall does not filter traffic between
machines on the same side of the firewall has enabled a set of attacks
that make use of *lateral movement*. The core idea is that an attacker
obtains a foothold in one system *inside* the firewall and then uses
that as a base of operations to move around to the ultimate
target. The initial system that the attacker breaches may not be particularly
important. Perhaps he gains access via a phishing attack or by leveraging a
vulnerability in the OS. But at this point the firewall is of no use,
and the attacker can start trying to find ways to move from one system
to another inside the firewall, until eventually he has access to a
machine of interest, such as one holding sensitive data. These types
of lateral movement attacks are extremely common and have been well
documented, often lasting for months before they are detected.

The obvious solution to problems of lateral movement would seem to be
internal firewalls. However, such a solution raises a new set of
challenges. Consider the example in :numref:`Figure %s
<fig-dc-firewall>`, in which a single firewall has been deployed to
filter traffic flows among a set of virtual machines in a datacenter.

Suppose that traffic sent from VM A to VM C needs to be processed at
the firewall. To ensure it is filtered, traffic needs to be routed
over a path that traverses the firewall, not necessarily the shortest
path from A to C. In the more extreme case of traffic from VM A to VM
B, the two VMs sit on the same host, so the traffic from A to B needs to be
sent out of the host, across the network to the firewall, and then
back to B. This is clearly not efficient, and consumes resources both
within the network and at the network interface for the hairpinned
traffic. Furthermore, the firewall itself has the potential to become
a bottleneck, as all traffic requiring filtering must pass through to that
centralized device.

Finally, there is considerable management overhead in supporting such
an internal firewall. Assume that we start with some sensible
default policies that deny all traffic flows aside from those
explicitly allowed. Each new application that is deployed will require
some new firewall rule to be created to allow traffic to flow between
the component machines for that application. If a VM is moved, we may
need to update the routing and the firewall rules to ensure that
traffic continues to be filtered correctly. All of these concerns have
led to internal firewalls being used rather sparingly.




.. _fig-dc-firewall:
.. figure:: figures/single-firewall.png
   :width: 600px
   :align: center

   A single firewall in a virtualized datacenter.

The solution to the many issues with internal firewalls appeared as
one of the features of network virtualization, the distributed
firewall. :numref:`Figure %s <fig-dist-firewall>` illustrates a
distributed firewall implementation. In this case, traffic sent from
VM A to VM C can be processed by a firewall function at either (or
both) of the virtual switches that it traverses, and still be sent
over the shortest path through the network underlay between the two
hosts, without hairpinning to a centralized firewall. Furthermore,
traffic from VM A to VM B need never even leave the host on which
those two VMs reside, passing only through the virtual switch on that
host to receive the necessary firewall treatment.

A significant side effect of distributing a service in this way is
that there is no longer a central bottleneck. Every time another
server is added to host some more VMs, there is a new virtual switch
with capacity to do some amount of distributed processing. This means
it is relatively simple to scale out the amount of firewalling in this
way.

.. _fig-dist-firewall:
.. figure:: figures/distributed-firewall.png
   :width: 600px
   :align: center

   A distributed firewall is implemented as part of the virtual
   switch in every host in a datacenter.

A detail that we have glossed over up to this point is that the
distributed firewall needs to be configured somehow. It would be
intractable to configure firewall policies in every single virtual
switch throughout a data center. This is why distributed firewalls
appeared as a feature of software-defined networks. The SDN controller
provides a central point of administrative control for firewall
policies, while the implementation of filtering rules is distributed
out to the virtual switches. Thus, for example, a rule that specifies
how traffic from VM A to VM B should be filtered can be expressed to
the SDN controller, which then calculates how to create the low level
filtering rules to push out to the appropriate virtual switches. The
SDN controller can also take account of such events as the migration
of a VM from one location to another, or the addition of a new VM that
requires additional firewall rules to be installed.

The approach described above is often referred to as
*micro-segmentation*. By contrast to traditional firewall approaches
in which large network segments offer unfettered access among all
devices on the segment, micro-segmentation creates small and precisely
defined virtual networks that connect only those systems that need to
be able to communicate to deliver a particular service or function.

For further details on network virtualization and distributed services
we recommend our companion book on software-defined networks.

.. admonition:: Further Reading

   L. Peterson, C. Cascone, B. O’Connor, T. Vachuska,
   and B. Davie. `Software-Defined Networks: A Systems
   Approach <https://sdn.systemsapproach.org>`__.

9.4 Zero Trust Security
-------------------------

As we noted above, traditional firewalls suffer from a fundamental
weakness: they attempt to divide the network into "trusted" and
"untrusted" zones. If an attacker manages to find a way to get access
within the trusted zone, perhaps by compromising a legitimate piece of
software running on the trusted side of the firewall, they now have a
large set of machines on which to launch further attacks without
interference. This state of affairs runs counter to some of the
security principles we outlined in Chapter 2, notably the principle of
least privilege. A machine on the trusted side of a firewall often has
access to a lot more resources—other machines and data—than is necessary to do
its job. One of the main approaches to improve this state of affairs
is known as "zero trust security".

The term "zero trust" has been in use for decades, but started to
enter widespread use around 2009, helped by the analyst firm
Forrester. The central idea behind zero trust is that, by default,
every device and user should be untrusted. Each user and device then
needs to authenticate itself to gain access to a precise set of
services. There is no blanket "trust this device to access anything"
policy. Zero trust stands in contrast to the old "perimeter security"
model in which there is the idea of a trusted region within a
perimeter protected by firewalls and an untrusted region outside the
perimeter.

Zero trust is sufficiently well accepted that NIST has written a
specification (see Further Reading below) which provides this helpful
definition:

   *“Zero trust…became the term used to describe various cybersecurity
   solutions that moved security away from implied trust based on network
   location and instead focused on evaluating trust on a per-transaction
   basis.”*

The classic example of trust based on location would be sitting in an
office connected to some corporate network. Such a network would, in
the past, have allowed anyone connected to it to access to a wide set of
servers and data, with the trust being assumed because the user had
managed to get admitted to the building.

There is more to zero trust that just getting away from location-based
access.  As a motivating example, consider a traditional VPN server
providing remote access to a corporate network. Once a remote user is
authenticated to the server, they are given access to a broad set of
"inside the perimeter" resources (machines, data, etc.)  within the
enterprise. In effect, the user and their device are now treated as
"trusted" to access many different systems. Such an approach has been
the cause of many published breaches of sensitive data. An infamous
example involved a heating and cooling system contractor's VPN
credentials being used to access a retailer's database of customer
credit card data. By contrast, a zero trust approach would entail a
precisely defined policy that authenticates the user and their device
to only the systems that are needed for them to do their job.

Micro-segmentation, described above, was one of the early technologies
created to help in the implementation of zero trust. Rather than
allowing all machines in a large network segment to communicate
freely, as was the case previously, micro-segmentation allows
precisely defined policies to limit communication among a set of
devices to just what is needed to deliver their intended function. For
example, the systems related to heating and cooling don't need access
to the systems where customer credit card details are stored. A
default policy of no access can also be established, with explicit
configuration then required to create precisely specified allowable
communication patterns between specific devices.

The VPN example above provides motivation for a different approach to
handling users and devices outside of the confines of a traditional
office and outside the perimeter defenses. A well-known and
comprehensive approach to rethinking the perimeter defense and VPN
model is Google's BeyondCorp, described in a 2014 paper in the further reading
section below. It is both an approach used to implement zero trust at
Google for employees accessing corporate resources and a service that
enterprises can implement themselves.

The starting position for BeyondCorp is that the perimeter model no
longer makes sense in a world of global connectivity, remote workers,
and ubiquitous mobile devices. The name suggests moving "beyond corporate"—that
is, moving away from the idea that there is a trusted corporate
network inside a perimeter and than any user or device inside that
perimeter should have access to everything on the corporate
network. Instead, users and devices have to be authenticated and
authorized to receive fine-grained access to specific resources, no
matter what their location is.

A central aspect of BeyondCorp is that only "managed devices" get to
access protected resources. These managed devices are all issued with
their own certificates so that they can be authenticated. Before
connecting to any service within the enterprise, the device connects
to a proxy that can perform authentication. After a
device has been authenticated, then the user also has to authenticate
themselves using two-factor authentication, and the user's access
privileges are looked up in a database. Thus, for example, engineers
may have access rights to engineering systems such as code
repositories, but not to finance or HR systems. Only after all these
checks—device, user authentication, and user authorization—have passed
does the connection get established between the client and the
server. Device checks may include not only verifying the certificate on a
device but also checking that its operating system is up to date or
that it has relevant antivirus software installed.

There are quite a few subtleties to BeyondCorp (and there are
publications describing its deployment) but the core principal of zero
trust that it embodies is this: location inside a perimeter (or
building) is not a reason for a device or user to be trusted to access
a broad set of resources. Nor should a VPN tunnel grant access to a
all the resources within the perimeter. Only after the device and the
user have been authenticated and authorization has been checked can
the user access a specific resource. So another way to say
“zero trust” might be “narrow and specific trust after authentication
and authorization” although it's less memorable.




.. admonition:: Further Reading

   S. Rose, O. Borchert, S. Mitchell, S. Connelly. `Zero Trust
   Architecture <https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-207.pdf>`__. NIST, 2020.

   C. Cunningham. `A Look Back At Zero Trust: Never Trust, Always Verify
   <https://www.forrester.com/blogs/a-look-back-at-zero-trust-never-trust-always-verify/>`__. Forrester, 2020.

   R. Ward and B. Beyer. `BeyondCorp: A New Approach to Enterprise
   Security <https://www.usenix.org/system/files/login/articles/login_dec14_02_ward.pdf>`__. ;login:, Usenix, 2014.

9.5. Intrusion Detection and Prevention
--------------------------------------------

As introduced at the beginning of this chapter, *security appliances*
are a generalization of firewalls. Such appliances are placed
throughout the network, watching for and responding to unwanted
traffic. The main challenge they face is how to distinguish between
good and bad traffic. This section looks at two examples: an
*intrusion detection systems* (IDS), and its sibling *intrusion
prevention systems* (IPS).

These systems look for anomalous activity, such as an unusually
large amount of traffic targeting a particular port number, which
often signals a malicious attempt to probe for a vulnerability. When
identified, the appliance either generates an alarm for network
managers, or in some cases, is able to take immediate action to limit
the damage.

A good example of an IPS is Snort, an open source project first
published in 1999, having started life as an IDS, and now owned by
Cisco. In its original incarnation, Snort provided a lightweight,
rule-based packet filtering and capture tool based on Berkeley Packet
Filters. The idea is that attacks, such as worms, have a recognizable
signature—for example, a particular combination of header values or a
unique byte-string embedded in the payload—and that the IDS can be
programmed with a rule to recognize the attack traffic, and raise
alerts when this happens. As an IPS, Snort now takes the additional
step of blocking the attack.

.. admonition:: Further Reading

   `Snort: Open Source Intrusion Prevention System (IPS)
   <https://snort.org/>`__.

Like firewalls, IDS and IPS need to see all the traffic traversing a
network if they are to detect attacks, and so strategic placement is
important. That leads to the same concerns about east-west traffic
that we discussed above, and thus there are also distributed versions
of these systems.

.. sidebar::  Identifying Unwanted Traffic

  *Our overview of security appliances could lead to the conclusion
  that decisions about whether traffic is legitimate or malicious is
  clear cut.  It often is not, and there can be real consequences in
  both directions.  For example, an overly aggressive IPS rule-set or
  anomaly detection heuristic could raise false positives, restricting
  legitimate traffic and negatively impacting revenue. Too conservative,
  and malicious traffic could crowd out legitimate traffic.*

  *It is also the case that that "unwanted" is in the eye of the
  beholder.  Network probes that are sometimes used in research, with
  the ultimate goal of improving the Internet in some way, are often
  flagged as malicious. Even a single unexpected UDP packet can
  trigger a cease-and-desist letter. In other situations,
  administrators want to ensure that a human (and not an automated
  crawler) is sending requests to their websites. There are "opt-out"
  conventions (e.g., adding a robots.txt file), but they depend on
  the good will of other actors. Some website administrators are now
  using Anubis, an open source web application firewall, to ensure that
  a human, and not an AI bot
  trying to scrape their content, is at other end of every HTTP request.*

For an IDS/IPS that uses packet signatures to be effective, the set of
potential attacks need to have been spotted in the wild and analyzed
so that suitable rules can be formulated. Sharing rules among a
community of users helps to speed up this process, and commercial
IDS/IPS systems typically come with a subscription to a
frequently-updated rules database. (See the Snort website referenced
above for an example set of community rules.)

Another approach to using signatures is to look for
*anomalies*—patterns in the behavior of traffic that somehow stand out
from "normal" and can be categorized as a potential attack. Clearly
this approach is attractive in that novel attacks can be detected
before they make it into a rule database. The hard part is achieving
high detection accuracy. Anomaly detection typically relies on machine
learning algorithms to classify traffic as "normal" or
"anomalous". Because both signature-based and anomaly-based detection
have their respective strengths and weaknesses, it is common to find
both approaches used in modern IDS/IPS systems.

The proliferation of security appliances again highlights the
principle of defense in depth. For example, if we had a perfect
firewall, we might not require an IDS or IPS. However, knowing that
firewalls will never block all forms of malicious traffic leads to the
conclusion that an IDS/IPS is worth having as a second line of
defense.
