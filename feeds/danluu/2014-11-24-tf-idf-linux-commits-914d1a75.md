---
title: TF-IDF linux commits
url: https://danluu.com/linux-devs-say/
published: "2014-11-24T00:00:00Z"
feed: danluu
guid: https://danluu.com/linux-devs-say/
---

# TF-IDF linux commits

I was curious what different people worked on in Linux, so I tried grabbing data from the current git repository to see if I could pull that out of commit message data. This doesn't include history from before they switched to git, so it only goes back to 2005, but that's still a decent chunk of history.

Here's a list of the most commonly used words (in commit messages), by the top four most frequent committers, with users ordered by number of commits.

User12345virotoinofandthetiwaialsathe-fortobrooniethetoasocforadavemthetoinandsparc64

Alright, so their most frequently used words are `to`, `alsa`, `the`, and `the`. Turns out, Takashi Iwai (tiwai) often works on audio (alsa), and by going down the list we can see that David Miller's (davem) fifth most frequently used term is `sparc64`, which is a pretty good indicator that he does a lot of sparc work. But the table is mostly noise. Of course people use `to`, `in`, and other common words all the time! Putting that into a table provides zero information.

There are a number of standard techniques for dealing with this. One is to explicitly filter out "stop words", common words that we don't care about. Unfortunately, that doesn't work well with this dataset without manual intervention. Standard stop-word lists are going to miss things like `Signed-off-by` and `cc`, which are pretty uninteresting. We can generate a custom list of stop words using some threshold for common words in commit messages, but any threshold high enough to catch all of the noise is also going to catch commonly used but interesting terms like `null` and `driver`.

Luckily, it only takes about a minute to do by hand. After doing that, the result is that many of the top words are the same for different committers. I won't reproduce the table of top words by committer because it's just many of the same words repeated many times. Instead, here's the table of the top words (ranked by number of commit messages that use the word, not raw count), with stop words removed, which has the same data without the extra noise of being broken up by committer.

WordCountdriver49442support43540function43116device32915arm28548error28297kernel23132struct18667warning17053memory16753update16088bit15793usb14906bug14873register14547avoid14302pointer13440problem13201x8612717address12095null11555cpu11545core11038user11038media10857build10830missing10508path10334hardware10316

Ok, so there's been a lot of work on `arm`, lots of stuff related to `memory`, `null`, `pointer`, etc. But if want to see what individuals work on, we'll need something else.

That something else could be penalizing more common words without eliminating them entirely. A standard metric to normalize by is the inverse document frequency (IDF), `log(# of messages / # of messages with word)`. So instead of ordering by term count or term frequency, let's try ordering by `(term frequency) * log(# of messages / # of messages with word)`, which is commonly called TF-IDF[1](#fn:H). This gives us words that one person used that aren't commonly used by other people.

Here's a list of the top 40 linux committers and their most commonly used words, according to TF-IDF.

User12345viroswitchannotationspatchofendiannesstiwaialsahdacodeccodecshda-codecbroonieasocregmapmfdregulatorwm8994davemsparc64sparcwekillfixgregkhccstagingusbremovehankmchehabv4l/dvbmediaatwereem28xxtglxx86genirqirqpreparesharedhsweetencomedistagingtidyremovesubdevicemingox86schedzijlstramelopeterjoeunnecessarycheckpatchconvertpr\_usetjcgroupdoesntwhichitworkqueuelethalshupoffsh64killaxel.linregulatorasocconvertthususehchxfssgi-pvsgi-modidremovewesachin.kamatredundantremovesimplernullof\_match\_ptrbzolnierideshtylyovsergeiacked-bycausedalanttygma500weupet131xralfmipsfixbuildip27ofjohannes.bergmac80211iwlwifiitcfg80211iwlagntrond.myklebustnfsnfsv4sunrpcnfsv41ensureshemmingersky2net\_device\_opsskgeconvertbridgebunkstaticneedlesslyglobalpatchmakehartleyscomedistagingremovesubdevicedriverjg1.hansimplerdevice\_releaseunnecessaryclearsthusakpmccwarningfixfunctionpatchrmk+kernelarmacked-byrathertested-bywedaniel.vetterdrm/i915reviewed-byv2wilsonvetterbskeggsdrm/nouveaudrm/nv50drm/nvd0/disponchipsetsacmegalbraithperfweisbeckereranianstephanekhalihwmoni2cdriverdriverssotorvaldslinuxcommitjustrevertccchrisdrm/i915wegpubugzillawhilstneilbmdarraysothatwelarsasocdriveriiodapmofkabernetfilterconntracknet\_schednf\_conntrackfixdhowellskeysratherkeythatuapiheiko.carstenss390sincecalloffixebiedermnamespaceusernshallynsergesysctlhverkuilv4l/dvbivtvmediav4l2convert

That's more like it. Some common words still appear -- this would really be improved with manual stop words to remove things like `cc` and `of`. But for the most part, we can see who works on what. Takashi Iwai (tiwai) spends a lot of time in hda land and workig on codecs, David S. Miller (davem) has spent a lot of time on sparc64, Ralf Baechle (ralf) does a lot of work with mips, etc. And then again, maybe it's interesting that some, but not all, people `cc` other folks so much that it shows up in their top 5 list even after getting penalized by `IDF`.

We can also use this to see the distribution of what people talk about in their commit messages vs. how often they commit.

![Who cares about null? These people!](https://danluu.com/images/linux-devs-say/null-percentile.png)

This graph has people on the x-axis and relative word usage (ranked by TF-IDF) y-axis. On the x-axis, the most frequent committers on the left and least frequent on the right. On the y-axis, points are higher up if that committer used the word `null` more frequently, and lower if the person used the word `null` less frequently.

![Who cares about POSIX? Almost nobody!](https://danluu.com/images/linux-devs-say/posix-percentile.png)

Relatively, almost no one works on `POSIX` compliance. You can actually count the individual people who mentioned `POSIX` in commit messages.

This is the point of the blog post where you might expect some kind of summary, or at least a vague point. Sorry. No such luck. I just did this because TF-IDF is one of a zillion concepts presented in the [Mining Massive Data Sets](https://www.coursera.org/course/mmds) course running now, and I knew it wouldn't really stick unless I wrote some code.

If you really must have a conclusion, TF-IDF is sometimes useful and incredibly easy to apply. You should use it when you should use it (when you want to see what words distinguish different documents/people from each other) and you shouldn't use it when you shouldn't use it (when you want to see what's common to documents/people). The end.

I'm experimenting with blogging more by spending less time per post and just spewing stuff out in 30-90 minute sitting. Please [let me know](https://twitter.com/danluu) if something is unclear or just plain wrong. Seriously. I went way over time on this one, but that's mostly because argh data and tables and bugs in Julia, not because of proofreading. I'm sure there are bugs!

Thanks to Leah Hanson for finding a bunch of writing bugs in this post and to Zack Maril for a conversation on how to maybe display change over time in the future.

* * *

1. I actually don't understand why it's standard to take the log here. Sometimes you want to take the log so you can work with smaller numbers, or so that you can convert a bunch of multiplies into a bunch of adds, but neither of those is true here. Please [let me know](https://twitter.com/danluu) if this is obvious to you.
    [\[return\]](#fnref:H)
