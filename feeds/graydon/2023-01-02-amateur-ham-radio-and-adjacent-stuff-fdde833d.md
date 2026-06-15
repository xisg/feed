---
title: amateur (ham) radio and adjacent stuff
url: https://graydon2.dreamwidth.org/305034.html
published: "2023-01-02T21:47:48Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:305034
---

# amateur (ham) radio and adjacent stuff

On a particularly stressful day this holiday I decided to distract myself by looking into the state of amateur radio (and adjacent issues). I don't remember what specifically set it off, but I fell into the rabbit hole for the next 48h or so and opened a million tabs and read a bunch and learned enought that I figured I'd make some notes here for future reference to myself / picking up if I'm ever interested again.

Reader beware: amateur radio is the natural home of some of the deepest turbo-nerds on earth, and it gets .. a bit heavy.

**what even**

[Amateur radio](https://en.wikipedia.org/wiki/Amateur_radio) is a set of frequency bands of the radio spectrum allocated by governments to use by basically anyone, so long as you have a license and behave yourself. It is also the term for the culture of people who tinker in this range, who also are called "hams". Unlike things like wifi (short range / low power) amateur radios can broadcast at higher power levels and cross longer distances, potentially globally. But again, you [need a license](https://www.rac.ca/requirements/).

**jargon**

There is a _ton_ of jargon in the space. The websites read like gibberish at first. You have jargon about electronics and radio frequencies, jargon about computer software and networking, jargon about legal licensing regimes and governmental organizations, plus call signs and residual code terms emerging from the long history of people trying to pack a lot of meaning into a very lossy channel. As just one example: there is a set of "Q codes" that are 3-letter codes starting with the letter Q that represent [all sorts of subjects](https://en.wikipedia.org/wiki/Q_code#Q-codes_as_adapted_for_use_in_amateur_radio) and people _literally use these as parts of speech_ like saying "QTR" rather than "what time is it" or "QSO" for "nice to meet you". Perhaps no more challenging than internet-ese ROTFL AIUI IIRC MFBT, but it's .. dense.

**population**

As far as I can tell amateur radio is _very scarcely populated_. Official numbers suggest something like 1/1000 of the general population. An informal poll of my mastodon followers showed 4% of 174 respondents as licensed users. It's a small enough community that the "clubs and member lists" approach can easily track everone who's into it.

**licenses**

Licenses used to require morse code. They don't anymore, though you get some bonus privileges if you can do that. They do require some electronics and regulation knowledge, so you have to study. When you get a license you get an alphanumeric callsign like "WB9NYI" assigned to you personally, which is a global thing. You can look up anyone by their callsign on [QRZ](https://www.qrz.com/).

Amateur radio is hyper-organized and overlaps significantly with emergency response organizations and military auxiliaries (and boat people, hiking people, weather people, astronomy people). There will be a local club with local nerds who will put you on their mailing list and train you for the exam and give you an exam, or you can take an exam online.

**bands**

There are [separate amateur bands](https://en.wikipedia.org/wiki/Amateur_radio_frequency_allocations) carved out of different parts of the radio spectrum: HF ("high frequency"), VHF ("very high frequency") and UHF ("ultra high frequency") and even some in SHF ("super high frequency") and EHF ("extremely high frequency"). Very roughly the higher the frequency the more data you can cram into the signal per second but the less distance you can propagate the signal, though the SHF bands become long-distance capable again -- the propagation characteristics are all complex physics issues.

There's some status to using HF because you can bounce signals off the ionosphere and moon or whatever, and a lot of old timers have a subculture of trying to contact one another over long distances competitively (which they call [DXing](https://en.wikipedia.org/wiki/DXing)), but this is of little interest to me and the kit is more expensive.

Many hams (especially those more cost conscious) stick to the [2 meter band of VHF, 144-148Mhz](https://en.wikipedia.org/wiki/2-meter_band) or the [70cm band of UHF, 430-450MHz](https://en.wikipedia.org/wiki/70-centimeter_band), and a lot of cheaper equipment is wired to just transmit and receive on these bands. Even in these bands, transmission rates are puny. Expect 1200bps, which for those of you who did not have the pleasure of growing up in the 80s is [slow enough that you can read faster than it](https://youtu.be/qdayzRIPEMk?t=438), and you're going to be waiting for the next lines of text.

The [5cm, 5Ghz](https://en.wikipedia.org/wiki/5-centimeter_band) SHF band is used for high speed projects, all digital stuff. Usually repurposed / slightly tweaked wifi equipment. Projects like [HSMM](https://en.wikipedia.org/wiki/High-speed_multimedia_radio) (high speed multimedia radio). I think these bands tend to overlap with other important industrial and science users and it's often a little hazardous to use them without knowing what you're doing.

**equipment**

As an example of "cheap" equipment: the Kenwood [TH-D72A](https://www.kenwood.com/usa/com/amateur/th-d72a/) which will run something like $500. There are 2 major legal suppliers of radios ( [Kenwood](https://en.wikipedia.org/wiki/Kenwood_Corporation) and [Icom](https://en.wikipedia.org/wiki/Icom_Incorporated), both Japanese) and one major illegal one ( [Baofeng](https://en.wikipedia.org/wiki/List_of_amateur_radio_transceivers#Illegal_marketing_and_distribution_in_the_United_States), Chinese). What makes a transciever illegal is mainly "whether you can casually program it to transmit outside of the licensed bands"; if so it won't get a license. The Baofeng ones can be programmed to do whatever you want with some software you download from presumably Highly Reputable Internet Places, though I suspect if you only run them in the licensed bands nobody will be the wiser. The internet appears to be full of people doing just that, but I'm the kind of rule-following stooge who would just buy an Icom to avoid worrying. There are a few canadian suppliers of all these things, probably [FreewayCom](https://www.freewaycom.ca/collections/vhf-radios) or [Burnaby Radio](https://burnabyradio.com/) or [Victoria Mobile Radio](https://www.vicmobile.com/products/) or something would be fine.

There is also expensive equipment, running into the thousands of dollars. Radios come in 3 form factors: handheld, things you wire into your car (dashboard radio, car-mounted antenna), and things you wire into part of your house (stereo-sized radio, roof-mounted antenna). The larger the radio waves you want to use (eg. HF) the larger the antenna you need. Again: handhelds tend to be 2m or 700cm wave so the antennas can be small. The HF nerds need gigantic antennas, and usually do it from home.

There are multiple parts, of course, and you can build every component yourself, or buy high end multi-component systems, or buy (typically lower-end) tightly integrated systems that have all the components wired together in a fixed configuration.

**amateur-adjacent commercial bands (resource roads)**

In Canada there are _zillions_ of roads that are not "official" roads but are managed by and for resource-extraction companies. The problem with these is that they are typically only one lane wide and they have giant industrial vehicles barreling along them working. So you don't drive on them unless you have some means of traffic control, and the [system settled on is specific radio frequencies](https://ised-isde.canada.ca/site/spectrum-management-telecommunications/en/licences-and-certificates/conditions-licence-appendices/rr-british-columbia-resource-road-channels). At the entrance to each road there's a sign saying which frequency to tune to. You tune to this frequency and call your position (kilometer marks) as you drive the road, so you don't get hit.

You have to be a licensed owner of a radio with these frequencies programmed in by RR number. They are not amateur-radio spectrum frequencies, but just next door in the 150Mhz ballpark. Again, you buy the radio from the distributors and you take a government test and pay an annual license fee ($45). Again, you can and [many people do use Beofengs they just program themselves](https://www.westcoastplacer.com/program-your-radio-for-bcs-backroads/); but I would prefer to get a licensed legit [Icom F1000](https://www.freewaycom.ca/collections/vhf-radios/products/icom-f1000-vhf-handheld-radio) or something, with the frequencies programmed in by the vendor.

**packet radio**

Once you have a radio, of course, you want to use it to carry computer stuff not just voice. Radio becomes modem and [packet radio](https://en.wikipedia.org/wiki/Packet_radio) tech defines various encodings of data and transmission control. There are a plethora of techs here and it would take forever to list them all, but here are some links:

- [AX.25, a data link layer protocol](https://en.wikipedia.org/wiki/AX.25).

- [AMPRnet](https://en.wikipedia.org/wiki/AMPRNet), a part of the global IP space routed into amateur packet networks.

- [APRS](https://en.wikipedia.org/wiki/Automatic_Packet_Reporting_System), automatic packet reporting system, a connectionless "broadcast datagram service" for short messages, telemetry, GIS and weather info that's very widely supported. Lots of handhelds can send and receive APRS messages and lots of people run "digipeaters" that just rebroadcast APRS traffic. Many stations that are "also doing something else" with radio will just transmit APRS packets to identify themselves.

- [APRS.fi](https://aprs.fi/) is a website tracking and displaying all APRS signals. There's a nice app that will also turn your phone (with a suitable radio) into an APRS broadcaster.

- [Winlink](https://en.wikipedia.org/wiki/Winlink) is a combination software package, set of protocols and network that runs a global email and emergency-response system that seems comparatively widely used.

- [TARPN](http://tarpn.net/t/packet_radio_networking.html), terrestrial amateur radio packet network, appears to be trying to make a low-budget / low-complexity network of long distance radio-only digital links.

- [HamWAN](https://hamwan.org/) is running an experiment in long distance 5Ghz / SHF links in the pacific northwest.

- [JNOS](https://packet-radio.net/jnos/), [BPQ32](https://www.cantab.net/users/john.wiseman/Documents/BPQ32.html) and [FBB](https://www.f6fbb.org/) are radio-based BBS software packages (many software packages are named after the call sign of the person who wrote them).

**software defined radios**

There is also [software-defined radio](https://en.wikipedia.org/wiki/Software-defined_radio), which does some of the steps that used to be done with fixed electronics digitally on your computer after just sampling input through a sound card and/or a USB dongle designed for receiving digital television (eg. the ubiquitous [RTL2832U](https://www.realtek.com/en/products/communications-network-ics/item/rtl2832u)). This is not something usually done in a handheld format, though people can and do run SDRs on phones and ipads.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=305034) comments
