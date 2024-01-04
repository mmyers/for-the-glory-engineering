---
title: "10 years of 1.3"
date: 2024-1-04
tags: history
---
#### Actually closer to 11 years, but who's counting?
I took some time away from For the Glory in 2011 and 2012 while working on Magna Mundi. By summer of 2012, the Magna Mundi project had publicly fallen apart and I was ready to get back to For the Glory.

Mandead's emails had kept FTG from ever completely leaving my mind, so it wasn't hard to switch back into FTG mode. Having a ready and willing collaborator was also a large boost in motivation. Otherwise I might have just fiddled around, tweaked it here and there, and never released anything.

I had given mandead repository access, so he had been able to work on scenario and scripting improvements during my absence. By January 2013, we had whipped 1.3 more or less into shape, but still didn't have a go-ahead from Paradox to do a full release. But Johan approved my request to post public beta patches on the forum, so after a few weeks of forum hype (and a very late flurry of last-minute commits), we released the official For the Glory 1.3 beta on February 23, 2013.

The changelog contained over a thousand lines! My favorite highlights were (summarized):

- Fixed the bug that was responsible for 90% of crashes
- Seven new mapmodes
- A bunch of new tooltips to make opaque game calculations more transparent
- A bunch of AI improvements, including removal of many AI cheats that now made the game too unbalanced
- Exported a bunch of new defines to defines.txt
- Added personal unions (scripted only)
- Added automatic rebel and pirate hunting
- Overhauled all scenarios to rebalance, correct historical setups, and use new features 

I thought we might need to do a couple more betas before full release, and indeed the community found some bugs that we fixed in the March 23 beta patch. Then a few more in the May 13 beta.

In the June beta I added a few new triggers besides fixing bugs. What harm could there be in that?

In the November beta I added some more triggers and exported more defines besides fixing bugs.

In the December 1 beta I added two more new triggers, a new scripting option to hide an event from the history log, and some new commands, besides fixing bugs.

In the December 29 beta I improved the AI a bit, added a new climate mapmode (only accessible through the console since I didn't have the ability to squeeze another icon into the map mode list), added some new defines, and fixed some bugs.

Surely we were near release!

...

...

...

...

By late 2015, it had become quite clear that we were NOT near release. I'd made a few changes but not compiled them into a beta patch. I hadn't heard from mandead in two years except a very brief email exchange in early 2015. Paradox had shut down the beta forum. No one had committed to our Subversion repository since January 2014. I had asked our private beta testers (on our external beta forum) for input in early 2015, and no one had ever responded. I had switched from developing FTG to playing EU4 and working on my EU3/EU4 scenario editor.

I still kept in touch via instant message with one of the Magna Mundi devs, so one day I asked him for his opinion on what I should do with FTG. I didn't feel right about completely abandoning it, but I saw no end to it and no one to help me. He suggested polling the community to see what they would like to see in a completed 1.3 patch, then implementing the top few suggestions and calling it quits. I liked the sound of that.

I posted [a suggestion thread](https://forum.paradoxplaza.com/forum/threads/the-1-3-wrap-up-or-how-do-i-stop-this-thing.889383/) in November 2015. I didn't want to stop it too early, so I forgot about it for a few months and didn't post the voting thread until [July 2016](https://forum.paradoxplaza.com/forum/threads/cast-your-votes-for-patch-1-3-voting-closes-31-july.956638/). Voting closed at the end of July, although there were few enough ballots cast that I believe I went ahead and counted several late ones.

By the time I had tallied up the votes, I'd started graduate school and didn't have time to work on FTG. It wasn't until 2017 that I was even able to get FTG to compile on my shiny new Windows 10 machine.

At the end of my last semester of graduate school in December 2017, I was finally able to put out my first installment towards fixing the community's top issues. I fixed issues \#3, \#6, and \#10, mostly because they were the easiest ones in the top 10. I figured now that I was done with grad school, I could get right back to it and do a beta every month or two like I had in 2013.

...

...

...

Six months after posting the first beta patch in four years, which I thought would generate some buzz, the forum thread had less than 30 posts. I wanted to quit again. (This time I picked up Starcraft instead of EU4.)

Once in a while, I'd get the urge to do some coding, but then I found that our free Subversion repositories had been deleted. XP-Dev.com no longer offered a free plan, and I hadn't migrated away in time to save the repos. For once my workflow of keeping two separate folders each of game code and game files, one synced with the repository and one for my ongoing changes, paid off. If I hadn't kept a clean repo copy around, I don't know if I would have ever continued work.

Still, I did little or nothing on FTG for several more years.

In 2021, a forum member named vjw set up a FTG Modding Discord server. I thought it was worth pursuing the community wherever it was, so I joined in November 2021, 12 years to the day after FTG was released.

On Discord, I discovered that there were still tens (*tens!*) of active FTG players and modders. The instant feedback cycle of posting in a chat room rather than on a web forum appealed to my dopamine receptors. I set up new repositories on Bitbucket and began working on FTG again.

In mid-2022, mandead discovered the Discord server. Then he discovered he could DM me at all hours of the day and night. And somehow, my motivation started to return. He began working on scenarios and scripting again, and by November we were able to produce the first FTG beta patch in nearly five years.

Since then, we have produced six more beta patches, some with quite major changes. I implemented some of the community's top issues from 2016, but also took suggestions from the active community on Discord. We've overhauled the rebels system, added new mapmodes, rewritten the Holy Roman Empire, added three new historical scenarios and three fantasy scenarios... and we're just getting started...

or are we?
