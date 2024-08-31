---
title: "A window into the soul"
date: 2024-08-31
tags: technical bug
---

The [Europa engine](https://en.wikipedia.org/wiki/Paradox_Development_Studio#Game_engines) that For the Glory is built on is fairly old at this point. This means that some things we take as a given in newer engines are difficult, if not outright impossible, to do in FTG.

As an example, take this simple event window:

![St. Albans](/assets/images/st-albans-old.png)

That's a lot of event text - this event is from the AGCEEP mod, which sometimes resembles an interactive history book. But nothing looks particularly unusual about this event. You can read to the bottom, then you just scroll down to read the rest of the text. Right?

...right?

In EU2, and in FTG up to very recently, the scroll bars appear on long event texts but don't do anything - they can't be moved at all. It took *fifteen years* to get the scroll bars to work in For the Glory.

Certainly someone at Paradox who was more familiar with the engine could have done it in far less time, and in fact once I put my mind to it a few days ago it didn't take all that long. But it did take a fair amount of accumulated knowledge.

### The problem

When YodaMaster and I put our heads together to look at the problem, all we really knew was that the scrollbar was visible but not functional. We didn't really understand how the windowing/drawing system worked.

It seemed obvious to us that something was preventing the scrollbar from receiving the correct mouse events. So we tried all sorts of things to manually force it to update. I can still find comments like this in the source code:

```
//case _CEUEVT_MOUSEWHEEL_: //YodaMaster: test => no effect
```
And
```
//YodaMaster
//	if (pe->GetType() == _CEUEVT_MOUSEWHEEL_)
//	{
//		...
//	}
//doesn't work but event is catched.
```
And my personal favorite:
```
//MichaelM: if this works, I don't care how ugly it is.
```
(Spoiler: it didn't work.)

We could see two places where we could catch the event, but nothing we did after that seemed to matter.
We tried firing the Update() manually on the event window - nothing.
We tried manually checking what kind of scroll wheel event was being caught, and calling the ScrollUp() or ScrollDown() on the window directly - nothing.
We tried getting a reference to the individual buttons (up, down, and "bullet") that the scroll bar was composed of, and firing the Update() methods on them manually - still nothing!

That's where it sat from 2009 until August 2024 - just last week.

### Getting closer

I did some more thorough tweaking-and-debugging sessions last week in an attempt to at least see if it was possible to fix. (Back in 2009 I hadn't been very good at that - compiling and launching took too long, so I tried to figure things out by reading the code. Making changes just to see if something on the screen changed was not in my bag of tricks.)

I discovered in one of these sessions that the events had the wrong relative origin. Or rather, the events had the upper left corner of the screen as their origin, as they always did, but the components inside the window had the upper left corner *of the window* as their origin. This meant that even if an Update() method was fired, none of the components would recognize that the mouse cursor was inside their bounds, so they never thought they needed to update. Aha, I thought, and added code to normalize the origin so the components would recognize the events correctly. And sure enough they started to update!

But still nothing happened.

### The real problem

Without realizing it, we had both based all our efforts on a single crucial assumption.

Almost all drawing code in the game follows the same lifecycle:
1. Game clears screen
2. Game fires a Draw event (there are several kinds of draw events, but this is a simplification)
3. Components catch the event
4. Components draw all their images, text, etc. to the screen

But for performance reasons, in-game message windows follow a different pattern. Message windows construct themselves *once*, drawing their components onto their own internal buffer. Then when they are moved around the screen, the buffer itself is moved. This way they don't have to cycle through drawing all their sub-components on every motion of the mouse as they're being dragged.
(This is probably not necessary nowadays, but in 1999 it was a solid design.)

Anyway, you may have already figured out why nothing we did could affect the window. It's because after the initial drawing, the window is nothing but a static image. It has *a picture of a scroll bar*, but the actual scroll bar isn't there.

After that key insight, fixing the window wasn't a very difficult process. I simply moved most of the drawing of the window from the Init() method into the Draw() method, accompanied by a Redraw() to clear the window background (otherwise the text leaves streaks when scrolled - ask me how I know).

The final result isn't very impressive to a modern audience, but I worked hard for it, so you'd better appreciate it.

![St. Albans](/assets/images/st-albans-new.gif)  
*(note that I also fixed a very old problem causing the event window to be sized as if the text were not in a scroll box, leaving a blank space between the text and the buttons.)  
(also ignore the Chimú shield - it's larger than the English shield, so I was using it to test that I didn't accidentally cause an overlap somewhere.)*
