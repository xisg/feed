---
title: The habitat of hardware bugs
url: https://yosefk.com/blog/the-habitat-of-hardware-bugs.html
published: "2016-07-13T00:00:00Z"
feed: yosefk
---

# The habitat of hardware bugs

The Moscow apartment which little me called home was also home to many other creatures, from smallish cockroaches to biggish
rats. But of course we rarely met them face to face. Evolution has weeded out those animals imprudent enough to crash
your dinner. However, when we moved a cupboard one time, we had the pleasure to meet a few hundreds of fabulously evolved
cockroaches.

In this sense, logical bugs aren't different from actual insects. You won't find bugs under the spotlight, because they get
fixed under the spotlight, crushed like a cockroach on the dinner table. But in darker nooks and crannies, bugs thrive and
multiply.

When hardware malfunctions in a single, specific way, software running on it usually fails in several different, seemingly
random ways, so it sucks to debug it. Homing in on the cause is easier if you can guess which parts of the system are more
likely to be buggy.

When hardware fails, nobody wants a programmer treating it as a lawyer or a mathematician (the hardware broke the contract!
only working hardware lets us reason about software!) Instead, the key to success is approaching it as a pragmatic entomologist
knowing where bugs live.

Note that I'm mostly talking about design bugs, not random manufacturing defects. Manufacturing defects can occur absolutely
anywhere. If you're in an industry where you can't toss a faulty unit into the garbage can, but instead must find the specific
manufacturing defect in every reported bad unit, I probably can't tell you anything new, but I can offer you my deepest
sympathy.

## CPUs

CPUs are the perfect illustration of the "spotlight vs nooks and crannies" principle. In CPUs, the spotlight, where it's hard
to find bugs, is functionality accessible to userspace programs - data processing, memory access and control
flow instructions.

Bugs are more likely in those parts of the CPUs only accessible to operating systems and drivers - and used more by OS
kernels than drivers. Stuff like memory protection, interrupt handling, and other privileged instructions. You can sell a buggy
CPU if it doesn't break too many commercially significant, hard to patch programs - and there aren't many important OS kernels,
therefore a lot of scenarios are never triggered by them.

A new OS kernel might bump into the bug, of course, but at that point, it's the programmer's problem. A friend who wrote a
small real-time operating system had to familiarize himself with several errata items, and was the first to report some of these
items.

It should be noted that an x86 CPU should be way less buggy in the privileged areas than the average embedded CPU. That's
because it's more _compatible_ in the privileged areas than almost any other CPU. AFAIK, today's x86 CPUs will still run
unmodified OS binaries from the 80s and 90s.

Other CPUs are not like that. I recall that ARM has 2 instructions, MCR and MRC (Move Register to/from Co-processor), and the
meaning of those instructions depends on their several constant arguments. It could flush the cache or program the memory
protection unit or do other things - a bit like a hypothetical CALC instruction where CALC 0 does addition, CALC 1 subtracts,
CALC 2 multiplies, etc. My point isn't that MCR and MRC look cryptic in assembly code, but that _the meaning changes between_
_ARM generations_. MIPS is similar, except they're called MFC0 and MTC0, Move From/To Coprocessor 0.

These incompatibilities do not break userspace programs, which can't execute any of these instructions - but the OS needs to
be tweaked to support a new core. If a new core introduces a bug in a privileged instruction, _that doesn't break old OS code_
_any more than it's already broken_ by ISA incompatibilities. Updating OS code is the perfect opportunity to also work around
fresh hardware bugs.

x86 chips also run more OSes than chips based on most other architectures. For instance, a now-defunct team making a fairly
widespread ARM-based application processor had to port about 3 versions of Linux (is there a chip maker who likes Linux with its
endless versions and having to port it themselves? Or do they secretly wish they could tell Linus Torvalds what he publicly said
to NVIDIA, namely, "fuck you"?) They also supported OS vendors in the porting of Windows and QNX. Overall, the chip probably
ever ran 5 full-blown OSes. x86 chips need to run endless OS builds - often built from very similar source code, but still.

The same principle applies to all hardware. **It's bug-free if and only if they can't sell it with bugs**. If
they can sell it with bugs and make it your problem, they very well might.

## Memory

$100 says your DRAM chip works. The DRAM chip is a mindless slave implementing precise commands by the DRAM controller on the
master chip, without any feedback - there are no retries, no negotiation, no way to say you're sorry. And no software will run
properly on faulty DRAM. Faulty DRAM isn't a marketable product.

Your board is definitely buggy. They told you they checked signal integrity, but they lied. If DRAM malfunctions, it's
probably the board, or the boot code programming DRAM-related components in a way that doesn't work on this board.

In the middle, there's the DRAM controller and the PHY. You'll only see bugs there if you're a chip maker - a chip is not
marketable unless such bugs are already worked around somehow. If you are indeed a chip maker, this is when you find out why
fabless chip companies are worth so much more than the equally fabless vendors of "IPs" such as CPUs and DRAM controllers. The
short answer is that chip makers are exposed to most of the risk. And in your case, some of this risk has just
been realized.

A DRAM controller bug can be very damaging to the chip maker, whose engineering samples might not work and whose production
schedule might be delayed. For the DRAM controller vendor - no big deal, "we have 3 more customers affected by this bug, we must
say you're taking it unusually passionately!" This is an actual quote. I want to add something here, something describing what
we chip makers think of these people, but words fail me. The point is, they fix the bug and ship the fixed version to their next
customers. You get to figure out how to make your engineering samples kinda work (often lowering the DRAM frequency helps), and
perhaps how to fix the design without too many changes to the wafer masks.

Bottom line is, DRAM controllers and PHYs can have bugs, usually it's the chip maker's problem, managing this risk is not
fun.

The bus interconnect between your processors and the DRAM controller probably doesn't have bugs - not correctness bugs, at
least. That's because today it's usually produced by a code generator, and such a code generator is really hard to market if it
has bugs, because they'll manifest in so many different ways and places. I found a bug in an interconnect once, and I was very
proud of my tests, but that was a preliminary version, and they found the bug independently. Real, supported versions always
worked fine.

_Performance_ bugs around memory access are legion, of course, because you can totally sell products with performance
issues, at least up to a point. A chip can have 2 processors with 8-byte buses each, going to a DRAM giving you 16 bytes per
cycle, through a shared 8-byte-per-cycle bottleneck. This interconnect is the handiwork of some time-starved dude on the chip
maker's team, armed with an interconnect-generating tool. Even such an idiotic issue will manifest on some benchmarks but not
others, and might not get caught at design time. And if you think _that_ is stupid, I've heard of a level 2 cache which
never actually cached anything, and this fact happily went unnoticed for a few months. (Of course, _this_ not being
caught at design time is when the team should start looking for a new career.)

Similarly, DRAM schedulers, supposedly very clever about optimizing DRAM performance, can in practice be really stupid, etc.
In fact, performance issues are among the hardest to pinpoint, and so are found in the greatest abundance in hardware and
software alike. But in a way, they aren't bugs.

## Peripheral devices

Expect peripheral device controllers to be pretty shitty. There really is no reason to make them particularly good. Only
device drivers access these things, so it all concerns just a handful of programmers, and then working around a hardware bug
here is easier than almost anywhere else.

A device driver has the device all to itself, nothing can touch the hardware concurrently unless the driver lets it, and the
code can fiddle with the hardware controller all it likes, perhaps emulating some of the functionality on the CPU if necessary,
and doing arbitrarily complex things to work around bugs. Starting with simpler things like reading memory-mapped registers
twice - a workaround for a real old bug from a real vendor in the automotive space, one who huffs and puffs a lot about
reliability and safety.

And a lot of peripheral devices also allow some room for error at the protocol level - you can drop packets, retransmit
packets, checksums tell you if ultimately all the data was transferred correctly, you can negotiate on the protocol features,
etc. etc. All that helps work around hardware bugs, reducing the pressure to ship correct hardware.

Also, since few people read the spec, there's no reason to make it very clear, or detailed, or up-to-date, or fully correct,
or maintain errata properly. This is not to say that nobody does it right, just that many don't, and this shit still sells.
Nobody cares that driver programmers suffer.

(By the way, I'm not necessarily condemning people at the hardware side here. Some low-level programmers like to complain
about how bad hardware is, but it's not obvious how much should be invested to make the driver writer's job easy, even from a
purely economic point view of optimally using society's resources, regardless of anyone's bottom line. If a chip is shipped
earlier at the cost of including a couple of peripheral controllers which are annoying to write drivers for, maybe it's the
right trade-off. I'm not saying that bugs should be exterminated at all cost, I'm just telling where they live.)

## Miscommunication

As a programmer, do not expect every device to follow protocols correctly. What will work is the CPU accessing memory, in any
way the CPU can access memory - with or without caching (the two ways generate vastly different bus commands.) But if the path
between the device doing the access and the device handling the access is a less traveled one, then you might need to do the
access in a very specific way.

For instance, bus protocols might mandate that access to an unmapped address will result in an error response. But an
unmapped address might fall into a large region which the interconnect associates with a hardware module written by some bloke.
So it routes your request to the bloke's module. The bloke can and will write hardware description code that checks for every
address mapped within his range, and then returns a response - but the code does _nothing_ when the address is
unmapped. Then reading from this address will cause the CPU to hang forever, and not only a debugger running on the chip,
but even a JTAG probe will not tell you where it's stuck.

There are many issues of this sort - a commonly unsupported thing is byte access as opposed to full word access (the hardware
bloke didn't want to look at low address bits or byte masks), etc. etc. A bus protocol lawyer might be able to prove that the
hardware is buggy in the sense of not following the protocol properly. A programmer must call it a feature and live with it.

As a chip maker, there's the additional trouble when you hook two working devices together, but lie about the protocol subset
they support, and they will not work together. For instance, a DMA engine and a cache might both "support out of order bus
responses." But the cache will return the response data interleaved at the world level, while the DMA might require responses to
be interleaved at the burst level, where the burst size is defined by the DMA's read commands.

The chip maker is rather unlikely to ship hardware with this sort of a bug, so by itself it's rarely a programmer's problem.
But they might make you set a bit in the DMA controller that disables the kind of requests producing out of order bus responses
when accessing certain addresses. Again you can argue if it's a bug or a feature, but either way, if you won't set the bit,
interesting things will transpire.

## Summary

- Don't trust freshly designed boards
- Don't trust peripheral controllers
- Trust CPUs in userspace & DRAM chips (almost always), and everything between the two (unless the chip is new &
  untested)
- Expect to bump into unsupported bus protocol features if you do anything except accessing memory from a CPU
- If you write your own OS, be prepared to work around CPU bugs (except perhaps on the PC)

[I wrote a long time ago](low-level-is-easy.html), and I still believe it, that lower-level programming is made
relatively easy by the fact that you're less likely to have bugs in your dependencies. That's because low-level bugs both hurt
more users and are harder to fix, and therefore people try harder to avoid them in the first place. However, this isn't equally
true for all the different low-level things you depend on.

I've described the state of things with hardware as it is in my experience, and attempted to trace the differences to the
different costs of bugs to different vendors. The same reasoning applies to software components - for instance, compilers are
more likely to have bugs than OS kernels - because, by definition, compiler bugs cannot break existing binaries, but kernel bugs
will do that. So I think it's a generally useful angle to look at things from.
