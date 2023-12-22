---
title: "The Road to For the Glory 1.0"
date: 2023-12-08
tags: history
---
*This post is adapted from a previous university talk, which was not recorded.*

*Please note, I am writing this from memories of 15 years ago. It is likely inaccurate in some specifics.*

---

I've only worked on two *real* games in my career (tiny personal projects that no one else ever saw don't count), so I don't know how a game dev project is *supposed* to work. All I know is how one project barely made it to the finish line and another imploded in what looked like the home stretch.

This blog is about For the Glory, so I'll save the Magna Mundi stories for a different time.

### 1. The distributed team
There's nothing wrong with teams that are widely distributed by geography, by areas of expertise, by talent level, or by any of a dozen other possibilities.

But I do not recommend a team whose leadership is distributed.

What do I mean by "distributed leadership"? I mean a team with multiple distinct leaders in charge and no clear idea of which is actually in chargey-charge *(hat tip to Jordan Michael Tuesday)*.

Since YodaMaster and I had both independently signed the development contract with Paradox, I saw us as equals. YodaMaster was used to leading the AGCEEP group and saw us as 1A (him) and 1B (me). We never explicitly settled this, but that wasn't the only thing we didn't explicitly settle.

### 2. The process
With no clear leader, we took a couple of months to even begin using source control. Our methods at first were:
- copy source code into personal folder
- make change, test change (optional)
- diff change back into clean source code folder
- send diffpatch to team

This worked for a bit when only YodaMaster and I were working on source code, but it quickly grew out of control and we had to switch to source control. But once in the habit, I kept parts of the "process": to this day I still have a FTG_M and a FTG_SVN project folder. This to me is simpler than branching and merging: I make changes to FTG_M and periodically use WinMerge to copy them over to FTG_SVN, which is source controlled.

Anyway, we didn't have money for an enterprise source control server. This was 2008, so we didn't know anything about distributed version control systems like Git. The only real option seemed to be finding a host that offered free private repositories. XP-dev.com had such a plan, so we went with them for both our source code and our game file Subversion repositories. More later on the wisdom of trusting important files to free-tier plans...

### 3. Features
With no clear leader, we also had no clear direction for the project. We eventually decided (by email - we **never once** used either voice or video calls) to start by fixing the numerous bugs and flaws we saw in EU2. We referred to this phase as "EU2 1.10". Afterwards, we would decide on and implement new features as selling points of our game, then we would release.

This might have worked if we had agreed on what flaws to fix. Or if we had been able to stop ourselves from adding a thing here and a thing there while we were ostensibly fixing bugs. Or, maybe, we could have gotten away with adding a few new things if our inexperience wasn't causing us to add bugs while we were still trying to fix bugs.

(One of my first commits contained a bug that I didn't realize was a bug for years afterward. But breaking multiplayer is no big deal - who plays MP anyway? Just some of the most dedicated players, that's who.)

### 4. Disincorporation
Incorporating costs money. Besides, would we have to incorporate in France or the USA or both? Who has time or money to do that? Certainly not us. Anyway, we're all friends (or at least online acquaintances). What could go wrong?

### 5. Money
Did I mention money? We had literally none. Any actual money invested into the project would come out of our own personal pockets... so we put none in. Time, yes, but not money. We coded in Visual Studio Community. We talked with our beta testers on forums-free.com. We kept our source code in a free plan on xp-dev.com.

All our work was unpaid, with the understanding that we would divide up whatever money we got at the time of release. Paradox had guaranteed us an advance of $25,000, then 50% of future adjusted gross proceeds - further payments, if any, would begin after our 50% stake (less marketing expenses) reached the $25k threshold. On the strength of this promise, YodaMaster quit his day job to work full time on FTG. I worked nights and weekends... when I was motivated. The other devs worked for a few months each.

---

### So how did we actually do it?
The source code we were given did not compile on Visual Studio 2008 (only VC6), so we spent the first month or so converting it. Then we took around a year fixing bugs and adding scripting features. I don't know how much time the others were putting in (we had no tracking of hours), but I estimate that I would put in 10-15 hours a week for a few weeks or a month, then slack off for a few weeks, then go at it again. With no deadline or concrete goals, I worked on the game when I felt like it and could work it in around my full-time job. I believe that YodaMaster put in much more time than this, but also not consistently. And we were both slowed by the fact that neither of us had ever professionally written C++ before. Two members of the team were much more proficient in C++, but neither was able to put in as much time. 

By summer of 2009, we had made a lot of progress on stabilizing the game and making it easier and more flexible to mod. We exported tons of \#defines and other hardcoded things out of the game engine and into plain text files, with the intention of leveraging that new capability in the AGCEEP mod. Only... the AGCEEP had more or less come to a halt. The "High Council" was divided into two groups: those who were too busy working on the new game to do any modding, and those on the outside who were aware that anything they did was likely to be superseded by the new game.

We still saw this as "EU2 1.10" and planned to start working on new features in the fall. But we forgot that we were not the ones making that decision.

Paradox informed us that summer that we had until fall to complete a working game. With no alternative, we signed the publishing agreement in August and Paradox officially announced "For the Glory, a Europa Universalis game" at Gamescom 2009.

*(Footnote: During creation of the promotional materials, Paradox asked us what the name of our company was. Of course we had none. YodaMaster suggested Crystal Empire Games. The rest of the team agreed that 1) we didn't like it, and 2) we couldn't think of anything better. So [Crystal Empire Games](http://crystalempiregames.com) it was, and is.)*

After the initial excitement on the Paradox forums had faded a little, people began to ask: isn't this just a reskinned EU2 with a few new mapmodes? All our hard work had only made it POSSIBLE for mods to do things - we hadn't actually DONE those things. In a bit of a panic (which I'm not sure the rest of team was feeling), I grabbed a feature from the Victoria source code and ported it into FTG. The result - "notifier" flags to help the player with some common or complex tasks - was the only visible improvement we could point to at game release.

We delivered the gold version of For the Glory 1.0 on October 19, 2009, and immediately began working on version 1.1.
