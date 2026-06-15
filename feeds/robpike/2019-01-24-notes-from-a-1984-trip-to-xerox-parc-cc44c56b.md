---
title: Notes from a 1984 trip to Xerox PARC
url: https://commandcenter.blogspot.com/2019/01/notes-from-1984-trip-to-xerox-parc.html
published: "2019-01-24T05:27:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-6272964685230353465
---

# Notes from a 1984 trip to Xerox PARC

Someone at work asked about this trip report I wrote long ago. I had not realized it was ever seen outside Bell Labs Research, but I now know it was. It's a bit of a time capsule now, and I was able resurrect it. Importing into Blogger was a bit of an ordeal that informs some of the points raised here, but other than that I make no comment on the contents beyond apologizing for the tone.

This Dorado is MINE, ALL MINE!

Rob Pike

April 29, 1984

A collection of impressions after doing a week’s work (rather than demo reception) at PARC.
Enough is known about the good stuff at PARC that I will concentrate on the bad stuff. The following may
therefore leave too negative an impression, and I apologize. Nonetheless...

Dorados work, and although most people at PARC now have their personal machine, it is hard to find
a spare one for a visitor (e.g. me), and the allocation method is interesting. There are rooms full of Dorados
in racks, and each office has a terminal in it. To connect a machine to a terminal, you make a connection at
the patch panel. There are just enough Dorados to go around, which makes it hard (but not impossible) to
find a spare when one is needed. What seemed to be in shorter supply was the terminals. There is a terminal in every office, but nothing like a terminal room. The presence of a visitor made it necessary to go to
the patch panel a couple of times a day (with muffled annoyance) to connect the temporarily vacant office
to the vacant machine. It really seems to me that it would make more sense and involve less mechanism to
get a couple more machines and terminals, put them in a public place, and have the plug board there for
emergencies like machine failure. Personal or not, it’s too big a deal to get a machine.

The idea of personal computers is an abreaction to time-sharing, with the unfortunate consequence
that personal computers (to be specific, D machines) look like personal time-sharing computers, albeit
without time-sharing software, an issue I’ll return to. The major issue is disks. Beside the plug board is a
white board divided into a grid of time-of-day cross machine-name. The machines have names because
they have disks; the reason you need the name is to get to a file you left there last time. Unfortunately,
Dorados need disks to run, because they must behave like Altos to run the crufty Alto software that performs vital services such as file transfer. The machine bootstraps by reading disk, not Ethernet. If this
were not the case, machine allocation could be done much more simply: you sit down at the (slightly
smarter) terminal you want to use, it finds a free Dorado, you log in to that and (if you care) connect, with
the Ethernet, to the disk you want.

The machines run one program at a time, and that seems fundamental. When I had finished preparing a figure with Draw, I had to exit Draw (‘‘Do you really want to quit? \[Yes or No\]’’ No. (I should write
the file.) ‘‘Name of file to write?’’ foo.draw ‘‘Overwrite the foo.draw that’s already there? \[Yes or No\]’’
Yes (dammit!). ‘‘Do you really want to quit? \[Yes or No\]’’ YES!!!!!!) to run the printing program. The
printing program then grabs the Dorado (running as an Alto) and holds it until the file is printed, which can
take quite a while if someone’s using the printer already. Once that’s accomplished, Draw can be started
again, but it won’t even accept a file name argument to start with. All the old state is lost.

As another example, if there is a file on your local disk that I want to copy to my local disk (the standard way to look at a remote file, it seems), I must find you in your office and ask you to stop whatever you
are doing, save the state of your subsystem (Smalltalk, Cedar, etc.) and return to the Alto operating system.
Then, you run ftp on your machine, then go get a cup of coffee while I go to my machine and pull the file
across. On a related note, I spent an hour or so one morning looking for a working version of a program,
which eventually turned up on someone’s Alto. Copies of the file didn’t work on my machine, though, so
whenever we needed to run it we had to find the person and so on... Given the level of interaction of the
subsystems, the way PARC deals with files and disks is striking.

Files are almost outside the domain of understanding at PARC. In fact, the whole world outside your
machine, Ethernet notwithstanding, is a forbidding world. I really got the feeling that their systems are so
big because they must provide everything, and they must provide everything because nobody out there provides anything, or at least it’s so hard to get to anyone helpful it’s not worth it. Smalltalk files are awkward,
but you can just (essentially) map the information into your address space and hold it there, whereupon  it becomes remarkably easy to use. These subsystems are all like isolated controlled environments, little heated glass domes connected by long, cold crawlways, waiting out the long antarctic winter. And each dome is a model Earth.

Using Smalltalk for a few days was interesting. The system is interactive to a fault; for example,
windows pop up demanding confirmation of action and flash incessantly if you try to avoid them. But overall, the feel of the system is remarkable. It is hard to describe how programs (if that is the right word) form.
The method-at-a-time discipline feels more like sculpting than authoring. The window-based browsers are
central and indispensable, since all that happens is massaging and creating data structures. A conventional
text editor wouldn’t fit in. I was surprised, though, at how often the window methodology seemed to break
down. When an incorrect method is ‘‘accepted,’’ messages from the compiler are folded into the text as
selected text, often at the wrong place. I found this behavior irritating, and would prefer a separate window
to hold the messages. The message in the code seemed to make it harder to find the problem by cluttering
the text.

A more interesting example occurred when I was half-way through writing a method and found I
needed information about another method. The browser I was in was primed with the information I needed,
but I couldn’t get at it, because the browser itself does not have windows. I could not leave the method I
was working on because it was syntactically incorrect, which meant I had to ‘‘spawn’’ a new browser. The
new browser remembered much of the state of the original, but it felt awkward to step away from a window
that had beneath it all the information I needed. A browser can only look at one method at a time, which is
annoying when writing programs, but if it had windows itself, this restriction could disappear.

A similar problem is that, when writing a method, if a new method is introduced (that is, invented
while writing the caller), you must keep the missing method name in your head; the window that tells you
about it when you accept the method disappears when you say it’s O.K. The complexity of managing the
browser puts an undue strain on the programmer; I had an urge to make notes to myself on a scrap of paper.

I missed grep. It’s not strictly necessary, because the browser does much of the same functionality,
but in a different way. To look for something, you look up its method; if you can’t remember the name, you
may have trouble. Experienced users clearly had no difficulty finding things, but I felt lost without automatic tools for searching.

Believe it or not, the response sometimes felt slow. I had to slow down frequently to let the eventfielding software catch up with me. For example, when making a rectangle for a new window, I could lose
as much as two inches off the upper left corner because of the delay between pushing a button and having it
detected. I therefore compensated, watching the cursor subconsciously to see that it was with me. The
obvious analogy is locking the keyboard.

The language is strange but powerful; I won’t attempt a full critique here, but just comment on some
peculiarities. The way defaults work in methods (isomorphic to procedures) is interesting, but necessary
because the generality of the design leads to argument-intensive methods. There are too many big words,
which stems partly from the lack of a grep (making it necessary to have overly descriptive names) and
partly from a desire to have the language look a little like English. For example,

    2 raisedToInteger: 3

is perfectly readable, but chatty at best. Because all messages must be sent to a left operand, unary minus
didn’t exist. I complained and it appeared on Wednesday. The type structure is somewhat polymorphic,
but breaks down. For example, rectangles cannot have both scalars and points added to them, because the
message is resolved by the left operand of the message, not the left and right.

One philosophical point is that each window is of a type: a browser, a debugger, a workspace, a bit
editor. Each window is created by a special action that determines its type. This is in contrast to our style
of windows, which creates a general space in which anything may be run. With our windows, it is easy to
get to the outside world; the notion of Unix appears only when a Unix thing happens in the window. By
contrast, Smalltalk windows are always related to Smalltalk. This is hard to explain (it is, as I said, philosophical), but a trite explanation might be that you look outwards through our windows but inwards through
theirs.

A few years ago, PARC probably had most of the good ideas. But I don’t think they ran far enough
with them, and they didn’t take in enough new ones. The Smalltalk group is astonishingly insular, almost
childlike, but is just now opening up, looking at other systems with open-eyed curiosity and fascination.
The people there are absorbing much, and I think the next generation of Smalltalk will be much more modern, something I would find always comfortable. However, it will probably be another model Earth.
