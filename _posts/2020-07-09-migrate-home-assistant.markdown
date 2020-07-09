---
layout: post
title:  "Migrate Home Assistant between Raspberry Pis"
date:   2020-07-09
---

Received a new fancy [Raspberry Pi 4](https://www.raspberrypi.org) from a
friend. This made my Home Assistant instance very jealous and requested to be
upgraded/migrated to the new Pi (from an older Pi 3).

Internet made me clear that I could not just move the SD card between the
devices since diffrent images between diffrent hardware.

I didn't find any good migration guide that I thought was easy enough so I
decided to write my own guide from this experience.

1. Create and download a Full snapshot from the old device's webgui (Supervisor > Snapshots)
2. Power of the old device
3. Download the Home Assistant image for [your device](https://www.home-assistant.io/hassio/installation/)
4. Write the image to your SD card (`dd if=/hassos.img of=/dev/SDCARD`)
5. Mount the SD card (require a system which can read and write EXT4)
6. Copy the snapshot from your Downloads folder to the folder `/supervisor/backup` located in the partition called `hassos-data`
7. Boot up the new device and follow the setup wizard (you can give it foobar since it will be overwritten in the next step)
8. Use your snapshot to Wipe and restore Home Assisant through the webgui (Supervisor > Snapshots)
9. Enjoy your old Home Assistant on your new hardware!

For a more seamless experience you might consider updating your DHCP records between step 2 and 7 so
that the new device receives the same IP adress as the old device.
