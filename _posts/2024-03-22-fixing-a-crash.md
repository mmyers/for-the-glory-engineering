---
title: "For the Glory architecture part 1: the client/server relationship"
date: 2024-03-22
tags: technical bug architecture
---
I'm not usually one to log my work as it happens. I have not found a way to reliably note what I'm doing without pushing myself out of the flow. However, I did try some logging back in October for a week, during which I happened upon an interesting bug.

I had just released a beta patch and had started running some hands-off games to test AI tech investment balance. During one of those sessions, the game crashed while Brandenburg was in the process of annexing Prussia.

I ran it again and was able to reproduce the crash. I'd been running the hands-off games in release mode for faster play speeds, so I then switched to a debug build to see if I could isolate the crash. But it didn't crash. I tried again in release mode. Crash. Again in debug mode. No crash.

WELL, THAT'S UNFORTUNATE.

That meant I was most likely dealing with [undefined behavior](https://en.cppreference.com/w/cpp/language/ub), which Visual Studio [partly prevents in debug mode by initializing memory](https://stackoverflow.com/questions/370195/when-and-why-will-a-compiler-initialise-memory-to-0xcd-0xdd-etc-on-malloc-fre). This would explain why the behavior was different. But it wouldn't help me find the cause.

There is lots of logging in debug mode, but all of that is disabled in release mode. The only decent way to log in release mode is logging to history.

I put history logging statements at the start and end of several methods, including `Annex()`. That didn't show anything. The annex method always succeeded.

I thought it might be related to the fact that Prussia is Holy Roman Emperor... maybe the election is bugged when there's no current emperor? But no, I logged the election method and it never even started. Something happened before that.

Since it didn't crash in debug mode, that indicated that something was writing out of bounds to an array (in retrospect that's exactly what happened and maybe I could have guessed which array).

I tracked the Annex event all the way through its journey and it had no problems. What else to try?

It occurred to me that I could log EVERY event that passed through the queue... but that would be both overwhelming and so slow I might not even get as far as the annexation event. So I made a small hack: set a `static bool` to `false`, until the first annexation event comes through, then set it to true. When it is true, log every event - both the name of the event type (there's fortunately a handy method for that) and the ID of the receiver.

(For the Glory is [built on a custom event queue system](/2024-02-02-for-the-glory-architecture-part-1.html), remember. Anything that affects every client must pass through the event queue.)

I hoped the last event in the log would be the same on multiple runs, which would point to exactly what happened. But no such luck. Sometimes it was `_CEUEVT_AI_TICK_` (and there's no easy way to know which AI). Sometimes it was `_CEUEVT_EXEC_CHANGE_TOLERANCE_`. But... hmm... both of those do happen around the same area: during an AI tick, the AI might change tolerance. What if I started in the AI tick method? 

I couldn't just log the start and end of every AI tick. That's too much logging again, too slow to run. But I could comment out chunks of code in the tick method, recompile, and run to see if it crashed. And fortunately for me, commenting out two thirds of the method did indeed stop the crash. That meant the hardest part of my work was done!

Now all I had to do was bisect the part I had commented out. Soon it became obvious that the culprit (duh again) was the newest addition to the method, a single line of code: `CheckDecisions();`. (This should perhaps have been obvious for longer than it was... Decisions were a brand new feature, and of course that's what would crash it. But there are enough old crashes that I foolishly hadn't assumed this one was my fault.)

I then added logging to the `CheckDecisions()` method to see which country and which decision was causing the problem. That took a couple of iterations of logging, because it turned out it wasn't the same country every time. It was the same decision, though: *Form the Kingdom of Germany*. And *Form the Kingdom of Germany* included a new trigger: `relation = { country = -6 value = 100 }`. (The relation trigger has been around for a very long time, but -6, referring to the current emperor, is one of my newer additions.)

Now, I had already checked every place in the code that referred to the current Holy Roman Emperor and ensured that none of them assumed that such a country existed. But there was a level of indirection in the trigger code. I had written a small function to resolve -1 (the current country), -2 (the current country's overlord if present), and -6 into valid country IDs. But if they failed to resolve, the function simply returned the value as-is. That meant that all callers were required to check the return value before calling CEUCountry::GetCountry(id) with the return value as the parameter. And none of the callers did... because before I added support for -2 and -6, the only value to resolve was -1, which is guaranteed to resolve. So I had changed the behavior of the function by adding the possibility of returning invalid country IDs. 

The simple fix was to return `_CEUCOUNTRY_TERRA_` (a designated signal value equal to zero) if -2 or -6 failed to resolve. All callers DO already check the return value from `CEUCountry::GetCountry()`, so that's all that is needed. If -2 or -6 fails to resolve, then `GetCountry()` will return `NULL`, and the trigger will return false.
