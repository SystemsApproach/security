Preface
========

The field of network security is roughly as old as networking
itself. Conventional wisdom tells us that the original Internet was
built without security features and we have been dealing with the
effects of those design decisions ever since. It is not that the
Internet's designers, implementers and architects were unaware of
security concerns; many of them were directly involved in developing
security technologies in early operating systems. But building a
high-performance network that could scale to global proportions and
accommodate the heterogeneous set of technologies that existed (and
those still to come) presented more than enough challenges. To quote
Internet pioneer Vint Cerf: *"getting this thing to work at all was
non-trivial.”*

Adding security capabilities to the Internet and the systems connected
to it has been a multi-decade effort. While there have been some
notable successes, such as widespread adoption of Transport Layer
Security (TLS), vulnerabilities continue to be found and exploited. In
part this is because security is a "negative goal": we are always
trying to ensure that there are no more cracks in our defenses, and it
is fundamentally impossible to prove that we are "done". So security
continues to evolve as new weaknesses are exploited and new mechanisms
deployed to prevent further exploits.

We have been educating ourselves about network security since we wrote
the first edition of *Computer Networks: A Systems Approach*
in 1995. There is an adage in security circles that no-one should
write their own cryptography code because it is so hard to get right,
and something similar might be said about trying to write a security
book. It is easy to make mistakes, especially if you are not deeply
immersed in the security world and the "look for every possible
weakness" mentality. We've had to make a few corrections over the
years to our security section in the big textbook. It is our
perspective on security as viewed in the broader networking context
that we have endeavored to bring to the topic.

So why did we decide to write the current book? First, we saw an
opportunity to write about security in a way that would make sense to
a networking person. Also, we wanted to take more of a systems
approach to the topic. While we always try to take a system-level view
in everything we write, it's easy with security to get bogged down in
the details of individual components, such as cryptographic algorithms,
without really tackling the systems issues. Cryptography is cool and
interesting (in our view at least) but it isn't really the main thing
to focus on if you are building secure systems. So while we do explain
the basics of cryptography here, it's not the focus. We're aiming to
explain how a system that comprises many moving parts, both in the
network and the end-system, can be made secure.

This question of focus caused us to examine how much we wanted to say
about end-system security. There are entire books to be written on
operating system security, processor architecture bugs such as Spectre
and Meltdown, and mitigating or preventing malware such as viruses on
end-systems. We made a conscious decision to draw a line around the
network and focus there, recognizing that, just as TCP is an important
network protocol that runs in end-systems, protocols like HTTPS and
TLS need to be covered in a book on network security. In fact TLS
provides an excellent case study in the system-level issues that come
into play when you try to secure traffic that flows between
end-systems over the Internet, and we devote an entire chapter to it.

With the never-ending set of threats and vulnerabilities that need to
be dealt with, it is all too easy to start thinking of security as
just a collection of point solutions to the problems that have been
identified so far. But in fact there is a well-established set of
principles that have been identified and written down by pioneers in
the field. The principle of least privilege and defense in depth are
two noteworthy examples. We have dedicated a chapter to exploring some
of the most widely accepted principles, and then we see them applied
repeatedly throughout the book. Perimeter firewalls, for example, can
be a part of a defense in depth strategy, while distributed firewalls
have been proposed as a way to apply least privilege to datacenter
networks.

Inevitably there are security technologies and types of attack that we
have not covered in this book. What we have tried to do is to give the
reader a sense of the principles at work and the building blocks that
exist to build secure systems, and then illustrate how the blocks can
be assembled to tackle particular problems ranging from securing Web
traffic to securing the routing system that underpins the
Internet. Security is a never-ending set of challenges and there will
always be new threats and solutions. This book aims to lay the
foundation so you can understand the network security landscape as it
continues to evolve.












Acknowledgements
----------------

Thanks to all the people who provided feedback on our book at various
stages in its development, particularly the following people:

- Brad Karp
- Cecilia Testart
- Zack Williams
- John Kristoff
- Motonori Shindo
- Nick Feamster
- Jeroen (jeroenh)
