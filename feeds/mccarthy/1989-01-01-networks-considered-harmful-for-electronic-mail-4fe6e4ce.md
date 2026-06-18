---
title: "Networks Considered Harmful - For Electronic Mail"
url: http://jmc.stanford.edu/articles/harmful.html
published: "1989-01-01T00:00:00Z"
feed: mccarthy
guid: http://jmc.stanford.edu/articles/harmful.html
---

# Networks Considered Harmful - For Electronic Mail

Professor John McCarthy

Father of AI

## Articles

### Networks Considered Harmful - For Electronic Mail

This is a 1989 editorial for the CACM. Now that Internet is universally available, its considerations are mostly obsolete.
2003 Note: This article was published in Communications of the ACM, 32(12), pp. 1389-1390, December 1989. Its contentions remained valid until the Internet was universally available. It remained valid in countries that had telephone systems but didn't have Internet yet.

Electronic mail (email), using ARPANET and other networks has been in use for almost 20 years. The widespread use of telefax is more recent. However, unless email is freed from dependence on the networks, I predict it will be supplanted by telefax for most uses in spite of its many advantages over telefax. These advantages include the fact that information is transmitted more cheaply as character streams than as images. Multiple addressees are readily accommodated. Moreover, messages transmitted as character streams can be readily filed, searched, edited and used by computer programs.

The reason why telefax will supplant email unless email is separated from special networks is that telefax works by using the existing telephone network directly. To become a telefax user, it is only necessary to buy a telefax machine for a price between $1,000 and $5,000 (depending on features) and to publicize one's fax number on stationery, on business cards and in telephone directories. Once this is done anyone in the world can communicate with you. No complicated network addresses and no politics to determine who is eligible to be on what network. Telefax is already much more widely used than email, and a Japanese industry estimate is that 5 percent of homes will have telefax by 1995 and 50 percent by 2010. This is with a $200 target price.

Email could work the same way at similar costs, but because of a mistake by DARPA about 1970, i.e. making a special-purpose, special-politics network the main vehicle for electronic mail, it was combined with other network uses that require higher bandwith and packet switching.

Another mistake was UUCP. It uses the telephone network, but three features inherited from its use within Bell Telephone Laboratories made its widespread adoption a blunder.

1. It assumes that both parties are using the UNIX operating system rather than using a general mail protocol. This is only moderately serious, because some other systems have been able to pretend to be UNIX sufficiently well to implement the protocols.

2. It requires that the message forwarding computer have login privileges on the receiver. This has resulted in a system of relaying messages that involves gateways, polling and complicated addresses. This results in politics in getting connected to the gateways and causes addresses often to fail.

3. Today forwarding is often a service provided free and therefore of limited expandibility.

There has been a proliferation of networks and message services on a variety of time-sharing utilities. Some of them are commercial and some of them serve various scientific disciplines and commercial activities. The connections between these networks require politics and often fail. When both commercial and noncommercial networks must interact there are complications with charging. A whole industry is founded on the technologically unsound ideas of competitive special purpose networks and storage of mail on mail computers. It is as though there were dozens of special purpose telephone networks and no general network.

The solution is to go to a system that resembles fax in that the ``net addresses'' are just telephone numbers. The simple form of the command is just

MAIL addressee@telephone-number,

after which the user engages in the usual dialog with the mail system.

The sending machine dials the receiving machine just as is done with fax. When the receiving machine answers, the sender announces that it has a message for . Implementing this can involve either implementation of protocols in a user machine or a special machine that pretends to be a user of the receiving machine or local area network. The former involves less hardware, but the latter involves less modification to the operating system of the receiving machine.

I have heard various arguments as to why integrating electronic mail with other network services is the right idea. I could argue the point theoretically, but it seems better to simply point out that telefax, which originated more recently than electronic mail is already far more widespread outside the computer science community. Indeed it is often used for communicating with someone who is thought to have an email address when getting the forwarding connections right seems too complicated.

The World of the Future

Eventually, there will be optical fiber to every home or office supplied by the telephone companies. The same transmission facilities will serve telephone, picturephone, telefax, electronic mail, telnet, file transfer, computer utilities, access to the Library of Congress, the ``National Jukebox'' and maybe even a national video jukebox. In the meantime, different services require different communication rates and can afford different costs to get them. However, current telephone rates transmit substantial messages coast-to-coast for less than the price of a stamp. Indeed the success of telefax, not to speak of Federal Express, shows that people are willing to pay even higher costs.

What about the next 20 years of email?

There are two kinds of problems, technical and political. Guess which is easier.

The main technical requirement is the development of a set of point-to-point telephone mail protocols. Any of several existing network mail protocols could be adapted for the purpose. Presumably the same kinds of modems and dialers that are used for fax would be appropriate but would give better transmission speeds.

Perhaps the organizationally simplest solution would be to get one or more of the various UNIX consortia to add a direct mail telephone protocol to UUCP. Such a protocol would allow mail to be addressed to a user-id at a telephone number. The computer would require a dialer and a modem with whatever characteristics were taken as standard and it would be well to use the same standards as have been adopted for telefax. It mustn't require pre-arrangement between the sending and receiving computers, and therefore cannot involve any kind of login. Non-UNIX systems would then imitate the protocol.

Fax has another advantage that needs to be matched and can be overmatched. Since fax transmits images, fully formatted documents can be transmitted. However, this loses the ability to edit the document. This can be beaten by email, provided there arises a widely used standard for representing documents that preserves editability.

The political problem is more difficult, because there are enormous vested interests in the present lack of system. There are the rival electronic mail companies. There are the organizers of the various non-profit networks. There are the engineers developing protocols for the various networks. I've talked to a few of them, and intellectual arguments have remarkably little effect. The usual reply is, ``Don't bother me, kid, I'm busy.''

It would be good if the ACM were to set up a committee to adopt a telephone electronic mail standard. However, I fear the vested interests would be too strong, and the idea would die from being loaded with requirements for features that would be too expensive to realize in the near future.

Fortunately, there is free enterprise. Therefore, the most likely way of getting direct electronic mail is for some company to offer a piece of hardware as an electronic mail terminal including the facilities for connecting to the current variety of local area networks (LANs). The most likely way for this to be accomplished is for the makers of fax machines to offer ASCII service as well. This will obviate the growing practice of some users of fax of printing out their messages in an OCR font, transmitting them by fax, whereupon the receiver scans them with an OCR scanner to get them back into computer form.

This is probably how the world will have to get rid of the substantially useless and actually harmful mail network industry.

More generally, suppose the same need can be met either by buying a product or subscribing to a service. If the costs are at all close, the people who sell the product win out over those selling the service. Why this is so I leave to psychologists, and experts in marketing, but I suppose it has to do with the fact that selling services requires continual selling to keep the customers, and this keeps the prices high.

I hope my pessimism about institutions is unwarranted, but I remember a quotation from John von Neumann to some effect like expecting institutions to behave rationally is like expecting heat to flow from a cold place to a hot place.

I must confess that I don't understand the relation between this proposal and the various electronic communication standards that have been adopted like X25 and X400. I only note that the enormous effort put into these standards has not resulted in direct telephone electronic mail or anything else as widely usable as telefax.

I am grateful for comments from many people on a version distributed by electronic mail to various BBOARDS.

Download the article in PDF.

"I don't see that human intelligence is something that humans can never understand."



Articles

Books and Reviews

Notes on AI

Slides

What is AI?

Top
|
John McCarthy's Original Website
|
We invite you to send comments and feedback
