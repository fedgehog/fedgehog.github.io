---
layout: post
title: "Phoenix Build Tool"
date: 2014-08-29 05:09:26 +0200
comments: true
categories: sbt
---

Quite some time ago I ported one small project from Maven to SBT. Surprisingly this time I had troubles causing OutOfMemoryError after 5-8 cycles of tests in continuous execution mode. After hour or two of OOMEs I even thought of Maven being a good build tool. But that feeling didn’t last long :).

After some playing with tests and code discovery happened that what I actually faced was so called referred pain. Referred pain is one that perceived at a location other than the site of actual cause.

And actual cause of pain was that dependencies on classes from OSS libraries used in project were declared in static way, what lead to heap polluted and finally caused OOMEs. Using Maven you typically won’t see this problem because each mvn invocation starts new process that ends right after execution of requested commands. SBT in contrary offers you interactive session, which is long lived, thus causing you to suffer eventually in case of careless resource management. Additionally SBT behaves oddly in case of OOME, so oddly that I couldn’t just stop process by hitting in terminal Ctrl+C and had to use process manager for these purposes. Which involved too much of annoying handwaving as for me.

So before treating real problem I decided to put a band-aid and automate a bit this process of killing-then-restarting-sbt-session.

Thus Phoenix Build Tool script born. Why Phoenix? Cause it reminds me a story about Phoenix bird that reborns arising from ashes left after it’s predecessor. But nowadays magic is not so popular as it used to be. So our phoenix is reborn with a little help of Functional Programming and Unix tools:

``` bash pbt.sh
alias pbt="(ps T | grep sbt | grep -v grep | awk '{print \$1}' | xargs kill -KILL) && sbt"
```

What it does is following:

Reports all processes associated with current terminal
Filters out only sbt processes
Plucks process identifiers
Kills processes by plucked identifiers
Starts fresh SBT session if all previous steps succeeded

However there is one peculiarity, you need to suspend process (using Ctrl+Z) to initiate this script.

But for me it was a huge time-saver already which allowed to adapt application design and see immediate feedback on changes.
