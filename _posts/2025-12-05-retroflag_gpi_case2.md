---
layout: post
title:  "Recalbox + Retroflag GPi Case 2 + Compute Module 4"
date:   2025-12-05 17:18:00 +0000
categories: jekyll update
---

Last year, I got gifted the legendary Game Boy (shoutout to my German bois), in a let's say, much more modern variant! Let me walk you through the pain of setting up this little bugger, luckily I've had documented my pain in short bullet points.

![GPi Case 2 for CM4](https://thepihut.com/cdn/shop/products/gpi-case-2-for-cm4-retroflag-104829-31588508074179_700x.jpg?v=1646518340)
![Raspberry Pi Compute Module 4](https://assets.raspberrypi.com/static/bdb6ad43efc8fc2590705f10fce9d069/248f1/temperature.webp)

# The Goal
Starting point is a beautiful [Retroflag GPi Case 2](https://thepihut.com/products/gpi-case-2) + [Compute Module 4](https://www.raspberrypi.com/products/compute-module-4/?variant=raspberry-pi-cm4001000) + 64GB SD card. The goal is to assemble everything, install Recalbox, install some games and play!

# Let's start
* Found a tutorial how to set it up, looks easy, following the tutorial
* Got the Raspberry Pi Imager, installed Recalbox on the SD card, easy
* Downloaded the GPi Case 2 patch, installed it (that patch allows you to use the built-in screen instead of HDMI)
* The Youtube comments are a bit confusing, some say you don't need the patch, what is it? (Spoiler: not needed)
* Put in the SD card, not that easy, it can move slightly
* Put in the CM4, difficult?? It took a bit until I had it in place, without it falling out... because no screws are needed, you just plug it in, but there is not much space for your fingers :S
* Started it, it's on, but nothing is happening, black screen
* Tried again with a HDMI cable instead, bootloader is trying to boot, looks like the logs are repeating, it's stuck bro
* Apparently there is nothing to start, because __"no sd card detected"__
* WHYYY ??? Is the SD card properly in place? It was a bit wobbly? I checked - all good. Is it broken? Can't be, Windows can read from it and write to it, no issues. Maybe the slot is broken?

# Of course it's not working
* Gooooooogle time, again, yay
* Turns out there are many different variations of the Compute Module 4
* Long story short, the light version version can only use the SD card, which I don't have. I have the CM4 8GB RAM with 32GB storage (eMMC)
* Why is the YT video not mentioning which variant it uses?? Am I the only fool who has run into this??
* Let's continue, the __new goal is to flash Recalbox onto the chip__
* Found instructions of how to flash the eMMC
* Installed rpiboot, took a while until I had figured out the right parameters and how to mount the chip as USB
    * Windows: [Flashing with rpiboot](https://docs.kubesail.com/guides/pibox/rpiboot/), download [rpiboot_setup.exe](https://github.com/raspberrypi/usbboot/raw/master/win32/rpiboot_setup.exe)
    * Linux: [https://github.com/raspberrypi/usbboot#building](https://github.com/raspberrypi/usbboot#building)
* As admin run `./rpiboot.exe -v [-d path to mass-gadget-64 etc.]`
* Plug in the micro USB-B cable and connect with laptop, start the thing and rpiboot will do the magic
* Now File Explorer shows an unknown/empty volume `D`, which means I can use the Raspi Imager to install Recalbox again, noice
* 5 hours later *\*in French accent\**

![5 hours later](https://media1.tenor.com/m/0dX6BLx4HZkAAAAC/yeet-yee.gif)
* At the end when it verifies the image, it fails with "SD Card broken bla bla blaa"
* Nooooooooooo

![noooooooooo](https://media1.tenor.com/m/UbgP_VN5GX0AAAAd/noooo-star-wars.gif)
* So basically it's saying, the onboard storage (eMMC) is broken, have I broken the chip while I was trying to place it inside the case?? Somehow I don't believe it
* Played around with PARTDISK, cleaning, formatting, creating new partitions bla bla
* Managed to create a working volume that I can access with the File Explorer
* Tried to install Recalbox again with the Raspi Imager -> same error at the end
* After a lot of googling and frustration I got an idea, I just force the image on the chip ( ͡° ͜ʖ ͡°)
* Downloaded the latest Recalbox image manually, and called in [Rufus](https://rufus.ie/en/), let's go!!
* Installed the image with Rufus
* That seemed to work quite well, after 2 minutes it was already installed, almost... *\*sigh\** it's stuck at 99.9%
* I cancelled after a while, as nothing was happening - mission failed - again
* Regardless, tried to boot up the module -> black screen
* Again with HDMI, I cannot believe it... Recalbox is starting up
* I just cannot use any buttons to control it
* Plugged in my Microsoft controller -> doesn't work
* Great, I have bruteforced the Recalbox image onto the chip, but I cannot use it... :S

# Did you really think it would be that easy?
* Let's mount the eMMC chip again and see what else I can do
* Windows (or is it rpiboot??) has problems to mount the chip as volume, so I can't really access it, I also can't install anything else on it, since I need an accessible volume
* Shite.. I might haved bricked the whole thing :cry:
* Let's try the same bullshit again, just from a Linux machine
* I just needed to mount it manually and repeat the whole process
* Recalbox is installed yet again :cool:
* But I need to install the patch, which is a batch file, only used by Windows :S (or do I?)
* Tried starting without patch, as I use the HDMI anyway, well it's starting up but I STILL DONT HAVE ANY CONTROLS (I'm going crazy)
* Let's try on Windows again, after several attempts mounting the chip with rpiboot (which sometimes randomly doesn't work?), I gained access
* I finally have access to the chip again, and could install some roms, *\*wohooo\**, but before I can do anything with it I need to make the system usable
* Installed patch
* Started without HDMI, black screen
* Started with HDMI, black screen
* Uninstalled patch
* Started with HDMI, it WORKS!!
* Alright, I need to make the controls somehow work, let's get a terminal and see what we can do
* Plugged in a keyboard, tried to open a terminal with some shortcuts, that should work according to Recalbox docs, but only a Recalbox-screen pops up, no terminal in sight

What a journey, like a rollercoaster lol, but I'm making slowly progress! Up until here (16.12.2024) I have spend many hours troubleshooting and jumping from one issue to the next one. So far, we have installed Recalbox, it's booting, but controls are missing. Next mission: get access to the terminal, I guess?


# Back into the last round
It's January 2025, after a litte christmas break I'm back from Germany and ready to continue with the fun. I didn't manage to get acccess to the terminal, nor did anything else work, so what do we do if nothing works? Yeah let's just try again :)
* Run Git Bash as admin (yes I need my linux commands even on Windows)
* `cd C:\Apps\ Raspberry Pi` and `./rpiboot.exe -v`
    * without `-d /path/to/massgadget/` as it would only mount the volume `SHARE`!
* Now we have mounted `REACALBOX (D:\)` and `SHARE (E:\)`
* Run Raspi Imager (v1.8.5)
    * Select "Raspi 4 / CM4 / bla bla"
    * Select Recalbox as OS
    * Select the mounted eMMC (RPi-MSD-0001)
* The image version is `Recalbox 9.2.3-Pulstar 64bits for Raspi 4 / 400 / CM4 / GPiCase2 / PiBoyDMG` (released 2024-08-01)
* Started the device with HDMI -> it WORKS
* Started the device without HDMI -> it WORKS
* Niccce

![niceee](https://media1.tenor.com/m/d8GH3gU0abUAAAAC/nice-south-park.gif)


# It's working
At this point I don't even know what exactly has worked and what not? I might find out whenever I'm brave enough to reinstall everything, but for now, I will only put some more games on it and play! If this finds you, may you have an easier journey than me lol.



# What have I learned?
One word, __patience__.

<br/>

---

<br/>

### Resources

Additional resources that I have used to troubleshoot
* <https://thepihut.com/products/raspberry-pi-compute-module-4?variant=39486530519235>
* <https://datasheets.raspberrypi.com/cm4io/cm4io-datasheet.pdf>

| Wireless | RAM | Storage | SKU | Part # | EAN |
| --- | ----------- |
| YES | 8GB | 32GB | [SC0678](https://thepihut.com/products/raspberry-pi-compute-module-4?variant=39486530519235) | CM4108032 | 0617588405464


* <https://www.reddit.com/r/retroflag_gpi/comments/rr9p5k/gpi_case_2_black_screen_at_boot/>
* <https://www.cytron.io/tutorial/installing-retropie-on-cm4-emmc-with-retroflag-gpi-case-2-swift-performance-actionclear-all-cache_wpnoncef3e213e2f7>
* <https://www.jeffgeerling.com/blog/2020/how-flash-raspberry-pi-os-compute-module-4-emmc-usbboot>
* [Retroflag GPi Case 2 w/CM4 - Part 1: Initial Setup Guide](https://www.youtube.com/watch?v=a3WvGds3ihM) (@TeamRetrogue)
* Note to myself, I installed rpiboot here `C:\Apps\Raspberry Pi\`
