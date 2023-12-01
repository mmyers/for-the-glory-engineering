---
title: "Introduction and background"
date: 2023-12-01
---

## How For the Glory came to be
I found EU2 in a bargain bin in 2003 and was intrigued by the screenshots. A game that had world maps like Risk (which was one of my favorites from years before) but with more complex mechanics than just "move armies, roll dice, conquer world"? It sounded like just my cup of tea.

And it was.

I was hooked immediately. My older brother and I played it every chance we got, then discovered that the game's files (other than the map) were almost entirely plain text and easy to modify. WHAT! I'd never successfully modded a game before, and this was AMAZING. We could (and did) set up completely new scenarios, scrap all the game events and write our own, even rename all the countries. We made a small mod of our own and published it on the Paradox Interactive forums. We found someone else working on a mod that looked interesting, and colloborated with them to improve their mod's setup of the USA since we had more local knowledge.

When I learned Java in college in 2005, it occurred to me that I could theoretically write a program that could automate some of our modifications. Of course, this was far more ambitious than anything I'd written in Java Programming 101, but I spent all summer trying it. I learned a lot about Java (making the next several programming classes very easy for me) but didn't succeed in my task. It wasn't till the next summer that I found some existing tokenize-and-parse code another modder had written and was able to repurpose it for my needs. Between 2006 and 2008 (when I graduated), I created tools including:
- [a tool to generate many similar events at once for EU2](https://github.com/mmyers/FTG_Event_Generator)
- [a tool to edit the positions of map sprites (like cities and army placements) in EU2](https://github.com/mmyers/FTG_Positions_Editor)
- [a tool to generate ID maps in EU2](https://github.com/mmyers/FTG_IDMapMaker)
- [a Javadoc-inspired tool to generate interlinked HTML documentation from (sometimes quite complex) EU2 event files](https://github.com/mmyers/FTG_Event_Doc)
- and my proudest achievement: [a full editor for EU3, the sequel to EU2](https://github.com/mmyers/eug)

The event documentation generator brought me into frequent contact with the AGCEEP team. AGCEEP (quite a mouthful, but perhaps better than the full name "Alternative Grand Campaign - Event Exchange Project") was the best known mod for EU2. It had formed from a merger of the AGC and EEP mods, which were the two top mods that focused on improving the game's historical accuracy. Where the original game included around 1000 events, AGCEEP had over 10 times as many. And the chains of "event X can fire event Y, which sleeps event Z unless event A has happened" were getting difficult to manage for the modders, let alone the players. A simple question of "How do I form the Kingdom of Italy?" on the forums would be met with either too little or too much information to be useful. The [AGCEEP website](http://agceep.org) added my generated documentation, and the modders began pointing people there for their questions.

Around this time - spring of 2008 - Paradox announced a new program: since their current flagship (EU3) and all future products would be running on a new game engine, they offered the old engine for licensing to any teams that believed they could make standalone products. The old Europa engine had first been developed in 1998-2000 for the original Europa Universalis, then used for EU2 (2001), Hearts of Iron (2002), Victoria (2003), Crusader Kings (2004), and Hearts of Iron 2 (2005). They offered to dump the final code of EU2, Victoria, and HOI2, along with licensing the existing assets of whichever of the three would serve as the base of the new game.

I knew the engine was in C++, which was not my strongest language, but I had taken a couple of classes that used C++, so I told Paradox that I was competent and showed them my Java projects as evidence.

Amazingly, Paradox accepted my application!

I signed all the agreements and mailed them off to Sweden. By June 2008, I had downloaded the EU2 source code and began looking through it. And boy, was it overwhelming. I had worked on what I thought were large projects before, but my idea of a large project was maybe a hundred source files. The engine had *around 800* .cpp and .h files - which I now know is very much on the low side for a game engine. (I later got an opportunity to work on a game using the EU3 Clausewitz engine and it had easily twice that.)

Paradox had set up a special private forum for all developers who had been accepted into the forum, and I soon noticed the names of some of the leading AGCEEP modders appearing there. One of them reached out to me to ask if I was interested in joining forces - and without a coherent plan of my own, staring at that mountain of code, I accepted the offer immediately. We recruited a couple other independent devs who had also been approved, and our team began work around the start of July 2008.
