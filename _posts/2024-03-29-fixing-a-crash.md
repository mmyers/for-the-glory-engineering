---
title: "Fixing a Crash"
date: 2024-03-29
tags: technical bug
---
*This is not my best bug story. The best one happened during patch 1.2 development in early 2010. Unfortunately, I no longer remember enough details to write a full blog post about it. All I know is that it was a seemingly random crash that took me 3-4 months to track down, and it turned out to have been my fault for confusing pass-by-reference with pass-by-value resulting in a corruption of game data thanks to a single missing `&`.*

*Instead, I'll tell a story that I __do__ know the details of.*

---

I'm not usually one to journal my work as it happens. I have not found a way to reliably take notes without pushing myself out of the flow of work. However, I did try some journaling back in October for a week, during which I happened upon an interesting bug.

I had just released a beta patch and had started running some hands-off games to test AI tech investment balance. During one of those sessions, the game crashed while Brandenburg was in the process of [annexing Prussia](https://en.wikipedia.org/wiki/Brandenburg-Prussia).

I ran it again and was able to reproduce the crash. I'd been running the hands-off games in release mode for faster play speeds, so I then switched to a debug build to see if I could isolate the crash. But it didn't crash.

I tried again in release mode. Crash.  
Again in debug mode. No crash.

WELL, THAT'S UNFORTUNATE.

That meant I was most likely dealing with [undefined behavior](https://en.cppreference.com/w/cpp/language/ub), which Visual Studio [partly prevents in debug mode by initializing memory](https://stackoverflow.com/questions/370195/when-and-why-will-a-compiler-initialise-memory-to-0xcd-0xdd-etc-on-malloc-fre). This would explain why the behavior was different. But it wouldn't help me find the cause.

There is lots of logging in debug mode, but all of that is disabled in release mode. The only decent way to log in release mode is logging to history.

I put history logging statements at the start and end of several methods, including `Annex()`. That didn't show anything. The annex method always succeeded.

I thought it might be related to the fact that Prussia was Holy Roman Emperor at the time they were annexed... maybe the election is bugged when there's no current emperor? But no, I logged the election method and it never even started. Something happened before that.

Since it didn't crash in debug mode, that indicated that something was writing out of bounds to an array (in retrospect that's exactly what happened and maybe I could have guessed which array).

I tracked the Annex event all the way through its journey and it had no problems. What else to try?

It occurred to me that I could log EVERY event that passed through the queue... but that would be both overwhelming and so slow I might not even get as far as the annexation event. So I made a small hack: set a `static bool` to `false`, until the first annexation event comes through, then set it to true. After it has been set to true, log every event - both the name of the event type (there's fortunately a handy method for that) and the ID of the receiver.

(For the Glory is [built on a custom event queue system](/2024-02-02-for-the-glory-architecture-part-1.html), remember. Anything that affects every client must pass through the event queue.)

I hoped the last event in the log would be the same on multiple runs, which would point to exactly what happened. But no such luck. Sometimes it was `_CEUEVT_AI_TICK_` (and there's no easy way to know which AI was being "ticked", i.e. polled for behavior). Sometimes it was `_CEUEVT_EXEC_CHANGE_TOLERANCE_`. But... hmm... both of those do happen around the same area: during an AI tick, the AI might change tolerance. What if I started in the AI tick method? 

I couldn't just log the start and end of every AI tick. That's too much logging again, too slow to run. But I could comment out chunks of code in the tick method, recompile, and run to see if it crashed. And fortunately for me, commenting out two thirds of the method did indeed stop the crash. That meant the hardest part of my work was done!

Now all I had to do was bisect the part I had commented out. Soon it became obvious that the culprit (duh again) was the newest addition to the method, a single line of code: `CheckDecisions();`. (This should perhaps have been obvious for longer than it was... Decisions were a brand new feature, and of course that's what would crash it. But there are enough old crashes that I hadn't assumed this one was new.)

I then added logging to the `CheckDecisions()` method to see which country and which decision was causing the problem. That took a couple of iterations of logging, because it turned out it wasn't the same country every time. It was the same decision, though: *Form the Kingdom of Germany*. And *Form the Kingdom of Germany* included a new trigger: `relation = { country = -6 value = 100 }`.

#### Sidebar: Background on triggers for non-modders
Many triggers can target a country. The relation trigger, for instance, can look like this:
```
relation = { country = ENG value = 100 } # our relation with England must be at least 100
```
Some triggers allow targeting the current country by using the special value `-1`, like this:
```
owned = { province = 382 value = -1 } # we must own Calais
```
I had recently added the special targets `-2`, meaning our overlord (if we have one), and `-6`, meaning the current Holy Roman Emperor.
```
relation = { country = -2 value = 100 } # good relations with our overlord
relation = { country = -6 value = 100 } # good relations with the emperor
```
Internally, the game treats both tags and -1/-2/-6 as integers until it comes time to execute the trigger script.

#### Back to the story
Some time ago, I had already checked every place in the code that referred to the current Holy Roman Emperor and ensured that none of them assumed that such a country existed. But because of the way triggers worked, there was a level of indirection which I hadn't spotted. There was a small function in the trigger code to resolve the parsed value of either a tag or -1/-2/-6 into a valid country ID. But if it failed to resolve (most likely because the input value was already a valid country ID), the function simply returned the value as-is.

That meant that all callers were required to check the return value before calling `CEUCountry::GetCountry(id)` with the return value as the parameter, since `GetCountry()` directly accesses the country list by index without checking bounds. And none of the callers did. **This, then, was the undefined behavior!**

Why did none of the callers check for negative values? Because, before I added -2 and -6, the only possible value to resolve was -1, which is guaranteed to resolve. So when I'd implemented -2 and -6, I had unwittingly changed the contract of the function by making it possible to return invalid country IDs.

The simple fix was to return `_CEUCOUNTRY_TERRA_` (a designated signal value equal to zero) if -2 or -6 failed to resolve. All callers DO already check the return value from `CEUCountry::GetCountry()`, so that's all that is needed. If -2 or -6 fails to resolve, then `GetCountry()` will return `NULL`, and the trigger will return false rather than crashing.

(An astute reader will observe that although some of my thought processes were questionable, my initial hunch that it was related to the annexation of the Holy Roman Emperor turned out to be correct!)

This fix went out in the [November 2023 beta patch](https://forum.paradoxplaza.com/forum/threads/ftg-1-3-beta-november-2023.1612602/).

### Postmortem: How could I prevent this bug?
The core issue here is that in the Europa engine, country IDs are simple integers indexing into the country list. That is why the helper function accepted an `int` and returned an `int`. This allowed it to accept either a tag or a -1/-2/-6 value and return a tag. But it also allowed it to return a negative number, which causes undefined behavior.

I'd like to do a refactoring and create a new type for country tags so they couldn't be treated as numbers, but they're so intertwined with every part of the codebase that it's a daunting prospect. For example, my little function to resolve -1/-2/-6 would likely have to also become a brand new type - we'd need some way to store the country from a `relation = { country = XXX value = YYY }` between parsing and resolution time. At the moment the trigger stores it as a simple `int`, but with a `CountryTag` type, we would have no way of expressing a reference value. We'd have to create a `CountryReference` type to store the parsed value as either a `CountryTag` or some kind of reference.

I don't know what other cascading effects such a major change would have, and with limited development manpower available, I'm always leery of putting the code base in a state where it might possibly take months to get it compiling again. Even source control branching wouldn't necessarily help - I'd still be sinking many hours into the project without any guarantee of succeeding, and in the best case it still would have no visible impact on players.

It might still be worth it just for the potential of saving more hours of bugfixing than I put into it.

