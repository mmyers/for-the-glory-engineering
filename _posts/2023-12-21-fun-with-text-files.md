---
title: "Fun with text files"
date: 2023-12-21
tags: technical
---

### Fun With Text Files  
#### (or, Blogs About Logs)
Since October, I've been doing a lot of hands-off games to tweak and test AI changes. For the first several games, I would simply run it through to the end and visually check the state of the world and the AI's tech levels. Then I realized I was ignoring a lot of potentially useful information in the log files.

When compiled and run as a debug build, For the Glory generates three log files (not countring netlog.txt, which is only used in multiplayer games):
1. history.txt - the same history that you see at the bottom of the game screen 
2. debug.txt - general debugging messages
3. log.txt - AI debugging messages

There's not much useful in history.txt, unless the game has just crashed and I want to quickly check the last thing that happened with a `tail` command. Otherwise, all the info in history.txt is already contained in the saved game.  
Debug.txt is best used for tracking down crashes.  
But log.txt was exactly what I needed.

Paradox had already set the AI up to log almost everything it did, and I added a few new messages to call out AI features I'd added myself. For example, in the FTG 1.3 beta patches, the AI is capable of determining whether a large investment is worth minting cash from monthly income.  This allows it to more reliably build the kinds of long-term investments that players have always been able to build. But I do need to know what investments the AI is usually considering to be worth the inflation cost.

Log.txt is cleared out on game launch, then grows at a rate of around 7-8 megabytes per game year. For a long game session of a couple of centuries, it will usually be well over a gigabyte. That's too much for Notepad to handle. Even Notepad++ chokes on that kind of file. What to do?
Trying desperately to remember my college days of Linux hacking, I opened a [Ubuntu WSL terminal](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-11-with-gui-support), navigated to For the Glory/Logs, and started digging in.

### Grep it and rep it

The first and most obvious thing to do is `grep` for the phrase I'd put in the log message:

    grep -i 'worth inflation' log.txt

There are two problems with this.
1. It takes more than a minute to run on a 1Gb file
2. It returns thousands of lines of results

I quickly cancelled (good old `Ctrl+C`) and added a crucial bit:

    grep -i 'worth inflation' log.txt | tail

This solves the second problem, and (surprisingly) solves most of the first problem. I timed it at around 8 seconds for a 1Gb file. Perhaps most of the time of the first command was formatting the results for terminal output, or perhaps there's something inside `grep` that makes large outputs a particularly bad case. I don't know and it doesn't matter at this point.

The default 10 lines of output from `tail` isn't enough to be very useful, so I usually set it to 50 lines:

    grep -i 'worth inflation' log.txt | tail -n 50

I also notice that the last line of the output is usually
> Binary file log.txt matches

I looked up the message and found [this Stack Overflow question and answer](https://stackoverflow.com/questions/23512852/grep-binary-file-matches-how-to-get-normal-grep-output), which indicates that I need to add `--text` to the `grep` command to ensure that all lines are treated as text.
I believe this is necessary in For the Glory's log files because some country names, such as Küstrin, include non-ASCII characters which `grep` doesn't understand.

So:

    grep -i 'worth inflation' --text log.txt | tail -n 50

Now we're getting somewhere! Let's look at some output:

    1592-11-11 : Muslim Nations decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13766058 is worth inflation (priority 810.0)
    1593-00-01 : Netherlands decides that _CAI_PLAN_SEND_COLONIST_ plan #13861643 is worth inflation (priority 81.0)
    1593-00-20 : England decides that _CAI_PLAN_SEND_COLONIST_ plan #13770845 is worth inflation (priority 97.0)
    1593-01-17 : England decides that _CAI_PLAN_SEND_COLONIST_ plan #13890232 is worth inflation (priority 97.0)
    1593-01-22 : Muslim Nations decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13766058 is worth inflation (priority 810.0)
    1593-02-02 : Bone decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13898657 is worth inflation (priority 3010.0)
    1593-02-12 : Muslim Nations decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13766058 is worth inflation (priority 810.0)
    1593-03-05 : Muslim Nations decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13766058 is worth inflation (priority 810.0)
    1593-03-27 : Bone decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13898657 is worth inflation (priority 3010.0)
    1593-05-03 : Muslim Nations decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13766058 is worth inflation (priority 810.0)
    1593-05-04 : Bone decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13898657 is worth inflation (priority 3010.0)
    1593-07-00 : Bone decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13898657 is worth inflation (priority 3010.0)
    1593-07-19 : Muslim Nations decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13766058 is worth inflation (priority 810.0)

Uh. Hmm. "Muslim Nations" isn't even a real country. It's a made-up country that exists only to provide a random number generator for [Alun's Realistic Reformation Mod](https://github.com/mmyers/FTG_Empire) (the name is a leftover from EU2 and normally isn't seen by a player). It has one province that is off-map and has no income. I don't need to see an activity of "Muslim Nations". How do I get rid of it?

[This Stack Overflow answer](https://stackoverflow.com/questions/10411616/grep-regex-not-containing-a-string) explains how to match when a string is not present. I need to pipe the `grep` output through a ` grep -v 'Muslim Nations'` command before `tail`ing it. We also still need a `--text` in the second `grep` command since it will still see non-ASCII characters.

    grep -i 'worth inflation' --text log.txt | grep -v 'Muslim Nations' --text | tail -n 50

There we go. Now the output looks like:

```1591-07-11 : Austria decides that _CAI_PLAN_BUILD_CITYRIGHTS_ plan #13723920 is worth inflation (priority 99.0)
1591-07-28 : Austria decides that _CAI_PLAN_BUILD_CITYRIGHTS_ plan #13723920 is worth inflation (priority 99.0)
1591-08-13 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1591-08-29 : Austria decides that _CAI_PLAN_BUILD_CITYRIGHTS_ plan #13745103 is worth inflation (priority 99.3)
1591-09-10 : Austria decides that _CAI_PLAN_BUILD_CITYRIGHTS_ plan #13745103 is worth inflation (priority 99.3)
1591-09-29 : Austria decides that _CAI_PLAN_BUILD_CITYRIGHTS_ plan #13745103 is worth inflation (priority 99.3)
1591-10-22 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1591-11-17 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-01-25 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-02-03 : Netherlands decides that _CAI_PLAN_SEND_COLONIST_ plan #13732589 is worth inflation (priority 81.0)
1592-02-19 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-02-27 : Netherlands decides that _CAI_PLAN_SEND_COLONIST_ plan #13732589 is worth inflation (priority 81.0)
1592-03-13 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-04-20 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-05-24 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-06-16 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-07-13 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-09-27 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1592-11-06 : Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)
1593-00-01 : Netherlands decides that _CAI_PLAN_SEND_COLONIST_ plan #13861643 is worth inflation (priority 81.0)
1593-00-20 : England decides that _CAI_PLAN_SEND_COLONIST_ plan #13770845 is worth inflation (priority 97.0)
1593-01-17 : England decides that _CAI_PLAN_SEND_COLONIST_ plan #13890232 is worth inflation (priority 97.0)
```
That's the kind of thing I want to see. Austria is minting to pay for a governor, which reduces inflation besides increasing income. That's a good use of money. Kazan is a poor country just trying to promote tax collectors. Probably a good use of money, although it's taking them a long time to build up enough cash to do it. How much inflation is worth it for a single tax collector? What would *I* do if I were playing Kazan?
*(It occurs to me that I built a new system to help both players and AIs evaluate buildings back in the [June 2023 beta patch](https://forum.paradoxplaza.com/forum/threads/ftg-1-3-beta-june-2023.1589118/). The AI does use that system for deciding WHICH provinces to build in, but it does not currently use it to decide if a building is worth taking some inflation. I should put that on the to-do list.)*

I also see that England is minting to send colonists. Where are they sending them? I can `grep` for the plan ID numbers 13770845 and 13890232 to see:

    $ grep '13770845' log.txt --text
    1593-00-20 : England decides that _CAI_PLAN_SEND_COLONIST_ plan #13770845 is worth inflation (priority 97.0)

Oh. It doesn't record the plan ID when it completes the plan. But I know it does log a message like "England sends a colonist...", so I'll use grep's `-A` option to take a number of lines after a match, then search that group of lines for England sending a colonist.

    $ grep '13770845' log.txt --text -A 100 | grep 'England sends'
    $ grep '13770845' log.txt --text -A 1000 | grep 'England sends'

No results. Need more lines.

    $ grep '13770845' log.txt --text -A 10000 | grep 'England sends'
    1593-01-01 : England sends a colonist to Nueltin
    1593-01-02 : England sends a trader to Susquehanna

It looks like Nueltin in northern Canada was the target. It's not a rich province (base tax value is 1), but it does produce valuable furs. That seems reasonable.
I used the same strategy to find that the other colonist was sent to Chesapeake, which is both a rich province and has valuable goods.

### Going deeper
What if I want to find out what kinds of messages get sent the most?

[This Stack Exchange answer](https://unix.stackexchange.com/questions/97341/how-to-grep-top-most-frequent-error-messages-in-a-unix-logfile) explains one way to do it. Basically, I need to pipe the file (or `grep`'s output) through `sort | uniq -c | sort -r` to first sort the lines, then group unique lines (prepending the number of times each was found), then sort that output descending (by line count).

But that won't work as is. All the log lines have date stamps. I need to first use `cut` to chop the first part of each line off. A little trial and error shows that `cut -b 14-` is what I need.

I'll also switch to using `head` instead of `tail`, since the lines are sorted in descending order. Let's try it!

    $ grep -i 'worth inflation' log.txt --text | grep -v '.*Muslim Nations.*' --text | cut -b 14- | sort | uniq -c | sort -rn | head
         95 Kutei decides that _CAI_PLAN_BUILDARMY_ plan #112156 is worth inflation (priority 4043.4)
         92 Socotra decides that _CAI_PLAN_BUILD_BAILIFF_ plan #3587439 is worth inflation (priority 1410.0)
         83 Rajputana decides that _CAI_PLAN_BUILDARMY_ plan #18433 is worth inflation (priority 91.5)
         70 Cyrenaica decides that _CAI_PLAN_BUILD_BAILIFF_ plan #10776152 is worth inflation (priority 1610.0)
         69 Cochin decides that _CAI_PLAN_BUILD_BAILIFF_ plan #92179 is worth inflation (priority 1810.0)
         68 Georgia decides that _CAI_PLAN_BUILDARMY_ plan #11195324 is worth inflation (priority 1622.5)
         66 Chernigov decides that _CAI_PLAN_BUILD_BAILIFF_ plan #1700240 is worth inflation (priority 1810.0)
         65 Benin decides that _CAI_PLAN_BUILDARMY_ plan #852334 is worth inflation (priority 2317.8)
         54 Ashanti decides that _CAI_PLAN_BUILDARMY_ plan #4988967 is worth inflation (priority 723.1)
         49 Kazan decides that _CAI_PLAN_BUILD_BAILIFF_ plan #13264264 is worth inflation (priority 2210.0)

I see that poor countries sometimes take years of minting to achieve a goal (on average the AI will log its budget decisions 1-2 times per month, so these are AT MINIMUM 2+ years of minting, and likely more than that). That might need some balancing.

### Further exploration
I probably haven't even scratched the surface of what I can learn from the log files. I'm sure some of you have interesting suggestions, which you can comment below just as soon as I figure out how to have comments here. Or find me [on Discord.](https://discord.gg/m3eKSCJYG)

Thanks for reading!
