---
title: Thank you Apple
url: https://commandcenter.blogspot.com/2012/09/thank-you-apple.html
published: "2012-09-22T22:55:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-8462550624728409907
---

# Thank you Apple

Some days, things just don't work out. Or don't work.

#### Earlier

I wanted to upgrade (their term, not mine) my iMac from Snow Leopard (10.6) to Lion (10.7). I even had the little USB stick version of the installer, to make it easy. But after spending some time attempting the installation, the Lion installer "app" failed, complaining about SMART errors on the disk.

Disk Utility indeed reported there were SMART errors, and that the disk hardware needed to be replaced. An ugly start.

The good news is that in some places, including where I live, Apple will do a house call for service, so I didn't have to haul the computer to an Apple store on public transit.

Thank you Apple.

I called them, scheduled the service for a few days later, and as instructed by Apple (I hardly needed prompting) prepped a backup using Time Machine.

The day before the repairman was to come to give me a new disk, I made sure the system was fully backed up, for security reasons started a complete erasure of the bad disk (using Disk Utility in target mode from another machine, about which more later), and went to bed.

#### The day

When I got up, I checked that the disk had been erased and headed off to work. As I left the apartment, the ceiling lights in the entryway flickered and then went out: a new bulb was needed. On the way out of the building, I asked the doorman for a replacement bulb. He offered just to replace it for us. We have a good doorman.

Once at work, things were normal until my cell phone rang about 2pm. It was the Apple repairman, Twinkletoes (some names and details have been changed), calling to tell me he'd be at my place within the hour. Actually, he wasn't an Apple employee, but a contractor working for Unisys, a name I hadn't heard in a long time. (Twinkletoes was a name I hadn't heard for a while either, but that's another story.) At least here, Apple uses Unisys contractors to do their house calls.

So I headed home, arriving before Twinkletoes. At the front door, the doorman stopped me. He reported that the problem with the lights was not the bulb, but the wiring. He'd called in an electrician, who had found a problem in the breaker box and fixed it. Everything was good now.

When I got up to the apartment, I found chaos: the cleaners were mid-job, with carpets rolled up, vacuum cleaners running, and general craziness. Not conducive to work. So I went back down to the lobby with my laptop and sat on the couch, surfing on the free WiFi from the café next door, and waited for Twinkletoes.

Half an hour later, he arrived and we returned to the apartment. The cleaners were still there but the chaos level had dropped and it wasn't too hard to work around them. I saw what the inside of an iMac looks like as Twinkletoes swapped out the drive. By the time he was done, the cleaners had left and things had settled down.

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiW0xkH2xSO4pqHckjWDFktk4rp2jRnXpMedlJU555VfhFIyHWO4EkkRDwaJv8WpFp2kj0IbXMKRnw0rX3Qj3g5kGs1q2eVd4ltRw-QW4hTwNM1Tx4pYjunjMvXU1mvVi8H6lcqwg/s400/L1000753.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiW0xkH2xSO4pqHckjWDFktk4rp2jRnXpMedlJU555VfhFIyHWO4EkkRDwaJv8WpFp2kj0IbXMKRnw0rX3Qj3g5kGs1q2eVd4ltRw-QW4hTwNM1Tx4pYjunjMvXU1mvVi8H6lcqwg/s1600/L1000753.jpg)

The innards of my 27" iMac

I had assumed that the replacement drive would come with an installed operating system, but I assumed wrong. (When you assume, you put plum paste on your ass.) I had a Snow Leopard installation DVD, but I was worried: it had failed to work for me a few days earlier when I wanted to boot from it to run fsck on the broken drive. Twinkletoes noticed it had a scratch. I needed another way to boot the machine.

It had surprised me when Lion came out that the installation was done by an "app", not as a bootable image. This is an unnecessary complication for those of us that need to maintain machines. Earlier, when updating a different machine, I had learned how painful this could be when the installation app destroyed the boot sector and I needed to reinstall Snow Leopard from DVD, and then upgrade _that_ to a version of the system recent enough to run the Lion installer app. As will become apparent, had Lion come as a bootable image things might have gone more smoothly.

Thank you Apple.

\[Note added in post: Several people have told me there's a bootable image inside the installer. I forgot to mention that I knew that, and there wasn't. For some reason, the version on the USB stick I have looks different from the downloaded one I checked out a day or two later, and even Twinkletoes couldn't figure out how to unpack it. Weird.\]

Twinkletoes had an OS image he was willing to let me copy, but I needed to make a bootable drive from it. I had no sufficiently large USB stick—you need a 4GB one you can wipe. However I did have a free, big enough CompactFlash card and a USB reader, so that should do, right? Twinkletoes was unsure but believed it would.

Using my laptop, I used Disk Utility to create a bootable image on the CF card from Twinkletoes's disk image. We were ready.

Plug in the machine, push down the Option key, power on.

Nothing.

Turn on the light.

Nothing.

No power.

The cleaners must have tripped a breaker.

I went to the breaker box and found that all the breakers looked OK. We now had a mystery, because the cleaners had had lights on and were using electric appliances—I saw a vacuum cleaner running—but now there was no power. Was the power off to the building? No: the lights still worked in the kitchen and the oven clock was lit. I called the doorman and asked him to get the electrician back as soon as possible and then, with a little portable lamp, went looking around the apartment for a working socket. I found one, again in the kitchen. The iMac was going to travel after all, if not as far as downtown.

The machine was moved, plugged in, option-key-downed, and powered on. I selected the CF card to boot from, waited 15 minutes for the installation to come up, only to have the boot fail. CF cards don't work after all, although the diagnosis of failure is a bit tardy and uninformative.

Thank you Apple.

Next idea. My old laptop has FireWire so we could bring the disk up using target mode and then run the installer on the laptop to install Lion on the iMac.

We did the target mode dance and connected to the newly installed drive, then ran Disk Utility on the laptop to format the drive. Things were starting to look better.

Next, we put the Lion installer stick into the laptop, which was running a recent version of Snow Leopard.

Failure again. This time the problem is that the laptop, all of about four years old, is too old to run Lion. It's got a Core Duo, not a Core 2 Duo, and Lion won't run on that hardware. Even though Lion doesn't need to run, only the Lion installer needs to run, the system refuses to help. My other laptop is new enough to run the installer, but it doesn't have FireWire so it can't do target mode.

Thank you Apple. Your aggressive push to retire old technology hurts sometimes, you know? Actually, more than sometimes, but let's stay on topic.

Twinkletoes has to leave—he's been on the job for several hours now—but graciously lends me a USB boot drive he has, asking me to return it by post when I'm done. I thank him profusely and send him away before he is drawn in any deeper.

Using his boot drive, I was able to bring up the iMac and use the Lion installer stick to get the system to a clean install state. Finally, a computer, although of course all my personal data is over on the backup.

When a new OS X installation comes up, it presents the option of "migrating" data from an existing system, including from a Time Machine backup. So I went for that option and connected the external drive with the Time Machine backup on it.

The Migration Assistant presented a list of disks to migrate from. A list of one: the main drive in the machine. It didn't give me the option of using the Time Machine backup.

Thank you Apple. You told me to save my machine this way but then I can't use this backup to recover.

I called Apple on my cell phone (there's still no power in the room with the land line's wireless base station) and explained the situation. The sympathetic but ultimately unhelpful person on the phone said it should work (of course!) and that I should run Software Update and get everything up to the latest version. He reported that there were problems with the Migration Assistant in early versions of the Lion OS, and my copy of the installer was pretty early.

I started the upgrade process, which would take a couple of hours, and took my laptop back down to the lobby for some free WiFi to kill time. But it's now evening, the café is closed, and there is no WiFi. Naturally.

Back to the apartment, grab a book, return to the lobby to wait for the electrician.

An hour or so later, the electrician arrived and we returned to the apartment to see what was wrong. It was easy to diagnose. He had made a mistake in the fix, in fact a mistake related to what was causing the original problem. The breaker box has a silly design that makes it too easy to break a connection when working in the box, and that's what had happened. So it was easy to fix and easy to verify that it was fixed, but also easy to understand why it had happened. No excuses, but problem solved and power was now restored.

The computer was still upgrading but nearly done, so a few minutes later I got to try migrating again. Same result, naturally, and another call to Apple and this time little more than an apology. The unsatisfactory solution: do a clean installation and manually restore what's important from the Time Machine backup.

Thank you Apple.

It was fairly straightforward, if slow, to restore my personal files from the home directory on the backup, but the situation for installed software was dire. Restoring an installed program, either using the ludicrous Time Machine UI or copying the files by hand, is insufficient in most cases to bring back the program because you also need manifests and keys and receipts and whatnot. As a result, things such as iWork (Keynote etc.) and Aperture wouldn't run. I could copy every piece of data I could find but the apps refused to let me run them. Despite many attempts digging far too deep into the system, I could not get the right pieces back from the Time Machine backup. Worse, the failure modes were appalling: crashes, strange display states, inexplicable non-workiness. A frustating mess, but structured perfectly to belong on this day.

For peculiar reasons I didn't have the installation disks for everything handy, so these (expensive!) programs were just gone, even though I had backed up everything as instructed.

Thank you Apple.

I did have some installation disks, so for instance I was able to restore Lightroom and Photoshop, but then of course I needed to wait for huge updates to download even though the data needed was already sitting on the backup drive.

Back on the phone for the other stuff. Because I could prove that I had paid for the software, Apple agreed to send me fresh installation disks for everything of theirs but Aperture, but that would take time. In fact, it took almost a month for the iWork DVD to arrive, which is unacceptably long. I even needed to call twice to remind them before the disks were shipped.

The Aperture story was more complicated. After a marathon debugging session I managed to get it to start but then it needed the install key to let me do anything. I didn't have the disk, so I didn't know the key. Now, Aperture is from part of the company called Pro Tools or something like that, and they have a different way of working. I needed to contact them separately to get Aperture back. It's important to understand I hadn't lost my digital images. They were backed up multiple times, including in the network, on the Time Machine backup, and also on an external drive using the separate "vault" mechanism that is one of the best features of Aperture.

I reached the Aperture people on the phone and after a condensed version of the story convinced them I needed an install key (serial number) to run the version of Aperture I'd copied from the Time Machine backup. I was berated by the person on the phone: Time Machine is not suitable for backing up Aperture databases. (What? Your own company's backup solution doesn't know how to back up? Thank you Apple.) After a couple more rounds of abuse, I convinced the person on the phone that a) I was backing up my database as I should, using an Aperture vault and b) it wasn't the database that was the problem, but the program. I was again told that wasn't a suitable way to back up (again, What?), at which point I surrendered and just begged for an installation key, which was provided, and I could again run Aperture. This was the only time in the story where the people I was interacting with were not at least sympathetic to my situation. I guess Pro is a synonym for unfriendly.

Thank you Apple.

There's much more to the story. It took weeks to get everything working again properly. The complete failure of Time Machine to back up my computer's state properly was shocking to me. After this fiasco, I learned about the Lion Recovery App, which everyone who uses Macs should know about, but was not introduced until well after Lion rolled out with its preposterous not-bootable installation setup. The amount of data I already had on my backup disk but that needed to be copied from the net again was laughable. And there were total mysteries, like GMail hanging forever for the first day or so, a problem that may be unrelated or may just be the way life was this day.

But, well after midnight, worn out, beat up, tired, but with electricity restored and a machine that had a little life in it again, I powered down, took the machine back to my office and started to get ready for bed. Rest was needed and I had had enough of technology for one day.

#### One more thing

Oh yes, one more thing. There's always one more thing in our technological world.

I walked into the bathroom for my evening ablutions only to have the toilet seat come off completely in my hand.

Just because you started it all, even for this,

Thank you Apple.
