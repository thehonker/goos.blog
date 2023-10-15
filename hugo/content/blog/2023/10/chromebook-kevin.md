---
title: "The Chromebook `Kevin`"
date: 2023-10-14T00:00:00-04:00
draft: true
tagline: "10/10 on the cute scale"
slug: chromebook-kevin
---

#### *the aarch64 obsession continues*

*Disclaimer: I wrote most of this about a month after the fact, and am just finishing it up now several months after the initial draft. Details have been forgotten to prevent the innocent.*

Almost a year ago I picked up a samsung chromebook plus v1 (or something like that), hardware codename `gru-kevin`

It has a really nice non-16x9 screen, good battery life, usb-c... what more do you need?

I gave chromeos a fair try, and it was kinda nice.

Fair number of complaints though:

- Running a full ass vm just for a linux env is ugly and slow on the rk3399 inside `kevin` \
  (that's right, our old friend from rockchip is back)
- The current state of chromebrew on aarch64 is kinda gross (on top of the existing grossness of chromebrew to begin with)
- General google-ness of the device overall (probably the biggest yuck of them all)

## goodbye chromeos

So, as I'm sure you guessed, I decided to put a normal linux on it. \
A good amount of googling lead me to
[Cadmium](https://github.com/Maccraft123/Cadmium), a set of image building tools for aarch64 chromebooks. \
Actual :shock:, kevin is on the support list!

I cloned the repo and started poking at it. \
There hadn't been a release in a few years, and autobuilds were broken.
It didn't look good but it was a starting point so I dove in.
A few hours of pain later and I had an image of the right size and shape.

Would it boot? I dd'd it over to a thumbdrive, ready to give it a try.
But first I had to enable developer mode in chromeos so I could boot from usb in the first place.

A few incantations later, my laptop was making a (very) loud BOOP BOOP at startup and showed a very scary looking warning screen saying the firmware was unlocked. I guess whatever I did worked.

A few attempts at catching the `^d` usb boot trigger later and I'm at a login screen.

Some dd to copy the thumbdrive's kernel and rootfs partitions to the emmc and I have a persistent ubuntu to boot to.

I installed the ubuntu-desktop metapackage, logged in, setup my stuff. It all worked, but it was pretty slow. Still though. ARM64 laptop!

## please make it stop

There were a few annoyances though, listed here in descending order...

1) That BOOP BOOP at startup. I cannot stress just how loud this was.
2) The fact that it still ran a google built firmware
3) The microsd card slot didn't work (with chromebook firmware in dev mode it's a jtag)
4) The weird A+B chromeos partition layout that I had to conform to

So. What can we do about that. The sdcard slot not working smells like a device tree issue. The rest of it I'm not yet sure if I can even address...

First though. I've run through cadmium's build process a few times now, and it's.... annoying. No offense. We all just like to do things our own way I guess.

So, naturally, I did. A medium amount of bash scripting later lead to the birth of [Potassium-OS](https://github.com/potassium-os/potassium).
This allowed me to break the build into better segmented stages.
With this tooling in hand I was able to easily rebuild just the device tree and kernel, or just the rootfs, or both.

Some amount of messing with the device tree later and I didn't have the sdcard slot working yet. Bummer, as I had visions of storing multiple different os builds on different sdcards, and switching between them at will. At some point I read a mailing list or commit message or something that mentioned that the sdcard slot on kevin is converted into a jtag of some sort when some bit is set in the eeprom. These feature bits are programmed by the firmware, and while I can read them from the os, trying to edit them doesn't do anything.

I resigned myself to having a mostly-working aarch64 laptop.\
Cool enough for me, right?

## never good enough

Nah.

I wasn't going to give up until at least item 1 from above was gone.
Wasn't sure how to approach this, I did load up the google firmware in a hex editor to see.... nothing. I can't really operate on that low a level.

Then I remembered that coreboot/libreboot/etc is a thing, especially for chromebooks. However all their attention in the past had been focused on x86 support, and I'm over here with a "rare" aarch64 device. \
No chance libreboot is gonna support this.

Right?

It turns out that libreboot had ***just*** merged one of their forks back in. A fork that was focused on adding aarch64 support to libreboot. I can't find the news post as of this writing, but it had happened like a week before I got the laptop.

It was... shockingly easy to get libreboot built for `gru-kevin`. \
The board configs were already there. I just tweaked a few small things to suit my preferences, typed make, and was given a binary blob to flash.

## adventures in spi

Fun fact, (most) chromebooks have a firmware write protect screw. Remove the screw, and you can flash the firmware from inside the running os. Pretty cool, right?

Except.... where is it? There's no WP screw visible on the top side of the board, even though the actual flash chip is right there:

![gru-kevin firmware spi flash](/2023/10/chromebook-kevin/IMG_3759.jpeg)
>There it is, now if only I could write to it

A few dozen googles lead me to a mailing list thread where someone mentioned that the WP screw is ***UNDER THE HEATSINK*** on the gru-kevin mainboard.

Normally this would be acceptable. Except that on kevin, the heatsink is riveted in place. It's not coming off without an amount of effort I'm not going to put into it.

So I looked into manually flashing the spi bios chip guy. It wasn't actually that difficult or scary of a process, despite what the internet tells you. You're just flashing a spi chip. That's all.

First I read the firmware inband from inside the os. About 10 times. And I sha256'd each copy to make sure they were all the same. AND remembered to copy it elsewhere. Then I dug through the parts bin and found a chip clip for our soic-8 spi flash friend. \
I thought about using an arduino as a flasher (I had in the past for a parallel access flash chip) until I remembered that raspberry pi's are a thing and they have spi lanes and gpio as well. Way easier.

A quick check of the pinout and some soldering/crimping later and I had this kinda thing happening:

![a raspberry pi setup as a spi flasher hooked up to a gru-kevin mainboard](/2023/10/chromebook-kevin/IMG_3811.jpeg)

>Peep that brains:battery ratio tho

As of this writing I neither remember what software I used to flash or where that flasher assembly is, RIP. I feel like I wrote a small wrapper to set CS and WE on the spi chip properly before/after flashing.

The first flashing was exciting but uneventful. I read in the flash a few times, shasum'ed it against the blobs I had pulled from inband, they matched. I type the incantation to flash the chip, and it flashes. I read it back again, and it shasums to the same as my libreboot output.

Woohoo!

I plug the battery back in and hit the power button. \
A few seconds pass and I see the kernel startup. \
No BOOP BOOP this time! No scary big red box on the screen!

## make it modern

That wasn't enough though.

My sdcard slot still wasn't working, and I still had to stick to the chromeos 2-kernel-1-rootfs layout.

Turns out that libreboot was shipping this sorta setup for my firmware:

- coreboot as 1st stage
- uboot as a payload of coreboot
- uboot was configured to boot a chromeos partition layout

So. It's u-boot. I know how to configure that. \
A few menuconfigs and flashes and "why u no boot"'s later, and I had u-boot providing a uefi enviroment that then booted an efi kernel stub (although I could have used grub2 or similar).

Somewhere in this the sdcard slot started working. Either right when I flashed libreboot the first time (which could have disabled the dev-mode that exposed jtag pins on that socket, as that was being enabled by chromeos firmware in dev mode), or somewhere in u-boot/coreboot configuration I saw the option to disable the jtag on it. Either way my sdcard slot was working.

At this point I had what equated to any other laptop with modern firmware.
I didn't have a boot menu yet as I hadn't taken the time to set one up in u-boot. But I could boot from emmc, usb, sdcard, even usb nic pxe.

## next steps, further annoyances

Now that we can boot reliably and in a modern way, let's fix up some of the annoyances in lolbuntu. Wayland was remarkably stable, more stable than x11 on this hardware even! However, fractional scaling made the gpu hurt, bad. Anything graphically intensive ran at slideshow speeds. \
This kinda sucked, as I had really hoped to use this as a notes-taking tablet most of the time. Another use case was tuning the megasquirt in my na6. While it has enough stronk to run tunerstudio (java is java after all), the update rate on the gauges and datalogging was horrible. Maybe 5 frames per second. Not nearly enough.

Kevin was plenty for light laptopping but not quite enough for what I wanted here.

## can you close for me

I'll be honest, `kevin` has been turned back into a chromebook and given to my parents.

I ended up buying another aarch64 laptop (Samsung galaxy book go 5G), this one has a Qualcomm 8cx gen2 though. Quite a step up from the rk3399 in kevin. It's still running samsung firmware (which does provide a uefi) and windows 11 (yuck), but that might change soon.

What will I do with Potassium now? I think I'll clean up the repos, get my libreboot changes into a repo, fix up uboot to be more friendly, and then archive it all.

I think in the future I'll use that org for other aarch64 things, so keep an eye on it for fun things to come.
