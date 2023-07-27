---
layout: post
title:  "Upgrading DJI RC-N1 firmware"
date:   2023-07-27
---

After been banging my head against the wall trying to pair my DJI remote
control (RC-N1) with a [Mini Pro 3](https://www.dji.com/mini-3-pro) I thought
it would be nice to share how it could get done.

The problem I got was that the [DJI Fly
app](https://apps.apple.com/us/app/dji-fly/id1479649251) on iOS complained
about inconsistent firmware and tried to upgrade it from `V02.00.1200`. All
attempts failed with "*Server error. Wait a moment and try again
(0×115000100002)*". The app was up-to-date and it didn't work to wait or reboot
all devices in diffrent order, the attempts failed. Even tried the terrible
[DJI Assistant 2](https://www.dji.com/downloads/softwares/assistant-dji-2) app
(which forced me to install Rosetta 2 😞) on my Mac which didn't even list any
available updates, only crashed a few times.

![DJI Assist 2](/images/2023/dji-assistant-2.png)

In the final moment (before total meltdown) I found [this
thread](https://forum.dji.com/thread-283060-1-1.html) on DJIs forum with a
solution! The magic trick was to upgrade the remote with an Android phone. The
only problem with that was that I then was located long out in Stockholm
archipelago and my family and relatives rocks iOS. I walked over to the closest
neighbour on the island and had the luck that he had an Android phone and was
willing to help me with the upgrade. Did take a few attempts to install the DJI
Fly app becase it wasn't available in Google Play, it was only available as
an APK [downloaded from DJI directly](https://www.dji.com/se/downloads/djiapp/dji-fly)
 (what's up with that Android and DJI?! 🤢).

After one or two failed attempts to upgrade on the Android phone it did get
through and the firmware was up-to-date, we at least thought. Once again
connected to my iPhone the DJI app found another version to upgrade to which it
this time was able to complete and a take off with the drone was done. Happy fly times!

My best guess (without any evidence) is that the firmware version I needed to
go through was located on a bad backend/CDN which the iOS version shipped
though the AppStore couldn't access for any reason (bad TLS that the Android
APK didn't care about or something like that) and that the newer firmware
files were located on a "better" backend. 🤷‍♂️

