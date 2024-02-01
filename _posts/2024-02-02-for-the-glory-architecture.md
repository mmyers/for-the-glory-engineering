---
title: "For the Glory architecture: the client/server relationship"
date: 2024-02-02
tags: technical bug architecture
---

It took me months, if not years, to really understand the structure of the Europa engine on which For the Glory is based. I hope to help you understand it in the next ten minutes.

If you're a programmer, you've likely coded your own game at some point. I know I had tried my hand at a few before working on FTG. If you've built any kind of map-based game, you'd expect to find C++ objects like Province and Country in FTG, and you'd be correct. However, it's not that simple.

For the Glory allows multiplayer games. There is no separate server, so the engine has to act as both client and server, and it must always keep track of whether it is acting as a server or as a client. Any modification of state (e.g. adding monthly income to a country or changing the population of a province) has to either be tracked entirely on the server, be sent in an event to all clients, or be completely deterministic. Of course, for most games, there is only one client, but this does not change the program flow.

**This architecture means we must be VERY careful to have all randomness occur ONLY on the server.** Clients must not derive their own random numbers unless the random number generators are all seeded deterministically with something that all clients can agree on. This is a class of bug that you won't discover immediately, since testing is usually done on a single machine that (by definition) agrees with itself. And it's not always obvious to a new programmer whether a given piece of code is intended to be run by the server or by the client.

#### An Unfortunate Example
One of my early modifications to the engine was to missionaries. In EU2, missionary success is determined at sending time. A missionary might be working for a decade or more, but if you save the game and examine the file immediately after sending, you'll see either "success = yes" or "success = no" in the script block. You can easily cheat it by simply changing a no to a yes.  

I didn't like the Calvinist interpretation, so I found the place in the code where the missionary is sent and the place where actual conversion happens. The code looked something like this:

    public void CCountry::SendMissionary(CProvince &prov)
    {
        bool success = this->GetMissionaryChance(prov);
        CMissionaryEvent *event = new CMissionaryEvent;
        event->SetSuccessful(success);
        event->SetTargetProvince(prov);
        event->SetDate(GetDate() + this->GetMissionaryTime(prov));
        PostGlobalEvent(event);
    }
    
    public void CCountry::HandleMissionaryFinished(CMissionaryEvent *event)
    {
        if (event->IsSuccessful())
        {
            event->GetTargetProvince()->SetReligion(this->GetReligion());
        }
        else
        {
            CreateRevolt(event->GetTargetProvince());
        }
    }

*(not the actual code - I wrote this from memory and haven't looked at the real code in months - but you can see the idea. There is also internal logic that routes events to the correct country at the correct time, logic that cancels missionaries if the province changes hands or if the country changes religions, a check for possibly changing cultures, popup messages for the player, yada yada yada)*

I removed the `success` field from the event and changed the handler logic to:

    public void CCountry::HandleMissionaryFinished(CMissionaryEvent *event)
    {
        if (CMath::GetSuccess(100, this->GetMissionaryChance(event->GetTargetProvince())))
        {
            event->GetTargetProvince()->SetReligion(event->GetTargetReligion());
        }
    }

Have you spotted the problem? (I hope so, because I telegraphed it two minutes ago.)

This code works! This 100% works when you fire up the game and test it! If you save a game a month before a missionary finishes, you can't predict whether it will succeed or fail!

It also causes an out-of-sync error in multiplayer games SOME of the time.

If one client has a good roll and another client has a bad roll (more likely if there are more than two clients or if success rate is close to 50%), then they will disagree on whether the missionary succeeded or not. One client will believe there is a revolt in the province (all failed missionaries result in an automatic revolt) and the other will not. Out-of-sync occurs and the game stops.

I've never been interested in multiplayer, so I did not prioritize this bug. It languished in the bug report forum for THIRTEEN YEARS before I finally realized multiplayer players are real people too.

I even added this comment at one point:

    // MichaelM (8/1/2016): TODO: THIS IS VERY WRONG. All clients are checking the success independently.
    //                      Need the host to roll the dice and post a secondary event if successful.

But I still didn't fix it for another six years after writing that comment!

The fix isn't that difficult, but it take a level of willingness to mess with game internals that I didn't really have in 2008 or 2009. I had to create two new event types, add them to the routing logic, and write new handlers. The fixed code looks something like this:

    public void CCountry::HandleMissionaryFinished(CMissionaryEvent *event)
    {
        if (IsHost()) // only do this on one machine!
        {
            if (CMath::GetSuccess(100, this->GetMissionaryChance(event->GetTargetProvince())))
            {
                CMissionarySuccessEvent *successEvent = new CMissionarySuccessEvent;
                event->SetTargetProvince(event->GetTargetProvince());
                ProcessGlobalEvent(successEvent); // ProcessGlobalEvent handles it immediately
                // unlike PostGlobalEvent, which waits until the event's target date
            }
            else
            {
                CMissionaryFailureEvent *successEvent = new CMissionaryFailureEvent;
                event->SetTargetProvince(event->GetTargetProvince());
                ProcessGlobalEvent(failureEvent);
            }
        }
    }

    public void CCountry::HandleMissionarySuccess(CMissionarySuccessEvent *event)
    {
        event->GetTargetProvince()->SetReligion(this->GetReligion());
    }

    public void CCountry::HandleMissionaryFailure(CMissionaryFailureEvent *event)
    {
        if (IsHost())
            CreateRevolt(event->GetTargetProvince());
    }

I built a hotfix .exe with this change in early 2022 for the FTG Discord, and the public fix went out in the [November 2022 beta patch](https://forum.paradoxplaza.com/forum/threads/ftg-1-3-beta-november-2022.1560292/).

And I really wish that were the only time I made that mistake, but of course it wasn't. I think I've rooted out almost all of that class of bug, but years of not properly thinking about the client/server relationship are hard to undo.
