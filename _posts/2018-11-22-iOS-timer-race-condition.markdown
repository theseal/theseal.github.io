---
layout: post
title:  "iOS timer race condition"
date:   2018-11-22
---

My wife has complained for some time that the alarm of the built-in timer in
iOS 12 doesn't go off when the time is up. I have never believed her until
today when I saw it with my own eyes. It was obvious that the auto-lock feature
in iOS 12 makes the timer just end quietly when the time is up if both values
are set to the same amount of time. I have confirmed the same issue on my phone
but I can't find any other person reporting the same thing. Is it just us?

<iframe width="560" height="315" src="https://www.youtube.com/embed/SInG77mbqCI" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

If my Apple Watch is connected to my phone the watch still makes a sound when the
timer is completed so it feels like a classic race condition.

Apple, please fix!
