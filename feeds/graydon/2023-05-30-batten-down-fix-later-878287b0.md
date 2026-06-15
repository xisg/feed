---
title: Batten Down Fix Later
url: https://graydon2.dreamwidth.org/307105.html
published: "2023-05-30T18:27:05Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:307105
---

# Batten Down Fix Later

Over on the socials, someone asked "Do you ever wish you had made yourself BDFL of the Rust project? Might there be less drama now if the project had been set up that way?"

This is a tricky question, not because of what it's superficially asking -- the answer there is both "no" and "no" -- but because of what I think it's indirectly asking. I think it's asking: is there some sort of original sin or fatal flaw in the Rust Project's governance -- perhaps its lack of a BDFL -- that's to blame for its frequent internal sociopolitical crises?

To that I can speak, hopefully briefly, or at least concretely. I think there are a few underlying causes, and one big pattern.

**The Pattern**

I haven't been involved in the project for a decade so I think anything I say needs to be taken with a big grain of salt, but I do keep up somewhat with the comings and goings, and it has not escaped my notice that _a lot_ of people over the years have left the project, and a lot of the leaving has been on fairly sour terms. A lot of people feel regret and resentment about ever participating. This is sad.

I think the way this works is:

2. There's an internal conflict in the project.

3. The conflict is **not well managed or resolved**, at least not in any way that one might call professional or healthy.

4. The conflict doesn't go away though, instead it's acted-out in ways we might call unprofessional or unhealthy, sometimes all at once, more often over a long period.

5. Possibly after some period of stewing in phase 3, one of the parties to the conflict just leaves the project, and everyone tries to pretend the conflict didn't happen.

I've seen about 4 variants of the unhealthy acting-out phase, though there might be a few more:

2. Use of informal power: pressure, social sway, cliques.

3. Use of external formal power: going to someone's boss.

4. Use of internal formal power: engaging the moderators.

5. Wearing-down opposition through expenditure of time.

The departure phase is often a bit abrupt-seeming because of the opacity of the causes, and takes a few different forms too, often patterned after the acting-out:

2. Quitting in disgust or even protest.

3. Being managed-out or fired by one's boss.

4. Being moderated-out or banned by the moderators.

5. Disengaging due to exhaustion or burnout.

**Understanding**

All these phases have **understandable** dynamics. I'm not saying "good", but understandable. I understand how they happen, I may even have contributed to establishing the pattern. Here are some background facts that help understanding:

2. Lots of people are just conflict-averse, from upbringing or personal trauma history or inherent nature or whatever. I'm conflict-averse myself! It's hard and tiring to engage with conflicts (especially for volunteers not being paid). It's often seemingly easier to avoid. Conflict avoidance is a sort of buy-comfort-now-pay-for-problems-later thing.

3. Rust feels very high stakes. It feels incredibly improbable. It frequently attracts a Significant Amount Of Internet Attention. It feels universally adored and universally hated at once. It feels like it could fail at any moment, but also like it's always on the cusp of victory. It creates in many a sort of [siege mentality](https://en.wikipedia.org/wiki/Siege_mentality), which I'm sure I helped incubate (I'm an anxious paranoid) and I'm sure this mentality is part of what drives so many to dedicate so much of themselves to it. It may be fun and gratifying, but for many it becomes bigger, they give their all to it, dedicate their careers or sense of purpose, identity. A lot of jobs and futures are often on the line! And many just legitimately burn out, working too much, too hard. And those who don't often retain an internal level of commitment that makes de-escalation hard, it makes compromise hard, it makes even publicly admitting problems hard.

4. One of Rust's weirdest cultural norms here -- again I might have helped make it and if so I apologize -- is to behave like conflicts are all the sort of "false tradeoffs" that can be solved like the speed-vs-safety tradeoff Rust famously claims to solve _without compromise_. That everything can be win-win, and if you don't find such a solution you're not trying hard enough. I mean, it's cool when that can happen, but sometimes things are just in legitimate tradeoff or conflict! Some things are zero-sum.

5. The project has, especially for its size, fairly minimal formal structure with which to manage or resolve conflicts. It was built out of volunteers on the internet, and incubated in an organization (Mozilla) that itself had weak formal structure. Go read [The Tyranny Of Structurelessness](https://www.jofreeman.com/joreen/tyranny.htm). Informal structures or strictly last-ditch formal structures (moderators or external options) are often the only thing anyone can see to grab onto.

6. One informal structure that governs a lot of Rust and a lot of open source in general (and is mentioned in The Tyranny Of Structurelessness) is that of time commitment. The passionate contributor often is willing and able to spend a lot of time on the project. And that time commitment is not always available to others. This is often explicitly stated as a positive virtue: "people who do the work get to decide". But those decisions often affect many other stakeholders, and "the people with the most time" might not represent those stakeholders well, or might lack skills or knowledge for the task at hand. And "putting in more time than others" is also a way -- a fairly unhealthy one -- of dealing with a conflict: rather than addressing it head-on, you just wear the other side out. This can even be employed _within_ a formal structure, if there's wiggle room for "how much input you contribute per unit time". Some people will show up to every city council meeting to push the same agenda. It works, you get your way, and it's not great.

**Fixing**

I don't really know how to fix these problems. If I knew I would most certainly make suggestions. I've said above I feel a fair amount of responsibility for setting some patterns, but of course to some extent the past is past and the future is what matters for the project most.

I guess my main suggestion is a don't-listen-to-me suggestion: "hire and listen to professionals with training in the subject", where "the subject" covers everything "a bunch of compiler nerds" are typically bad at. Project management to political science to finance to communications to mediation to personnel. The Project is now a decently large (and very diffuse) organization, and humans have studied how to run those for a long time, have categories of professionals who are expert in each topic. Listen to them. Don't try to work each out from first principles, and don't pretend that because you're a bunch of compiler nerds on the internet you get to dodge all the mechanisms of a normal organization.

I don't know to what extent the new governance system bears enough of the fingerprints of such professionals, and I don't know if it will or won't do much to address The Pattern, but I might be surprised. I'm not skilled in these areas! Casually and ignorantly speaking: I like the parts that sound like formal delineation of powers, and term limits and role rotation to avoid burnout, and transparency of decisions. I like things that sound like stakeholder representation and time-investment limitation.

I don't know that I see enough acceptance of the reality of conflict, and the need to resolve it explicitly. I don't know if it does enough to imbue positions of power in the project -- including informal power -- with accountability for their actions, or to communicate that publicly to instil confidence. Mainly: I don't know if it will do enough to make life livable for people who don't want to dedicate themselves to the project body and soul.

I personally haven't anywhere near the bandwidth, it's all just too much. To those participating, you have my best wishes. Good luck.

**Footnote: The Foundation**

I want to be clear on one point: the site of origin for problems in Rust's governance is, as far as I can tell, not the Rust Foundation, and the chorus of people jumping on the Foundation every time there's Some Drama In The Community is usually misplaced.

Again speaking only from what I've been able to tell as an outsider observing and listening on back-channels, the Foundation _usually_ appears to be the Adults In The Room, and when it does something that seems superficially weird it's _usually_ because it's trying to impossibly square some circle handed to it by the Project.

I think the Foundation actually does more or less just want to support the Project, and the Project is consistently not being a very easy thing to support.

**Footnote: Corporate sponsors**

Moreover: while the project is partly volunteer-driven and partly corporate-sponsored (often but not solely via Foundation members), and at times I believe corporate sponsorship produces [bad incentives in maintainers to not do quite enough simple maintenance](https://graydon2.dreamwidth.org/306832.html), I don't think in Rust's case this has ever gone in the direction people worry the most about: companies "buying influence" in conflicts or unpopular decisions, or otherwise hijacking the language.

That's a _possible_ problem, but also one the Foundation's structure was substantially designed to minimize the risk of, and so far I think we're not seeing it.

**Footnote: The original ("BDFL") question**

To give the "no" and "no" answers at the top of this post a little more flesh: I don't like attention or stress, I was operating near my limits while I was project tech lead back in 2009-2013, and part of my own departure had to do with hitting those limits and kinda falling apart (as well as the company not really responding to that fact well -- see also "everyone is human"). Everything I've described here in terms of people's human fallibility applies to me in spades!

Additionally, I've no reason to believe I would have set up strong or healthy formal mechanisms for decision making, conflict management or delegation and scaling. I have no training in any of these subjects and was totally winging it within my role at Mozilla. Mozilla itself seemed to have little skill at these subjects either. The one time I tried to do a "formally structured decision" on Rust's design, I tried to hold a ranked-choice vote on keywords, and it went terribly: everyone hated the results.

**Footnote: Moderation**

I don't know the entire saga of the moderators of the Rust project -- I really haven't been involved since like 2013 -- and so I don't wish to imply anything specific about their past behaviour (especially since today's mods aren't yesterday's mods anyway). IMO that's a subject for someone informed to debate elsewhere. I do want to say two things:

2. I wrote the original CoC (it was shorter and simpler then) and I stand by the notion that having written community norms and a process of enforcing them is generally a thing internet communities need. I don't think "having mods" is bad, or mods exercising their powers (with care and oversight) is bad.

3. I do want to acknowledge that mods are human and can both act-out in unhealthy ways themselves, or be engaged-with in bad faith to be instruments of someone else's unhealthy acting-out. I think this is usually rare, I don't think a lot of the people who complain about this usually have a leg to stand on, but it's possible and it's fair to demand a heightened level of scrutiny of moderator behaviour to avoid the possibility. IME most good mods welcome the opportunity to leave a paper trail for their decisions and explain themselves, subject to the caveat of "not wanting to make things worse and/or engage with internet grief mobs".

**Fotnote: Cliques and lack of transparency**

Having friends you collaborate with is great! And doing stuff in private and not having to explain every little thing you think or say to randos on the internet is great! Neither of these things is a problem _on their own_; indeed these are often prerequisites for a lot of people feeling safe and comfortable and willing to participate at all. Being a Scrutinized Public Figure is often exhausting and can exclude people who don't have it in them, either by nature or circumstance. But a certain amount of transparency is a necessary part of making accountable decisions affecting other people -- part of the exercise of power.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=307105) comments
