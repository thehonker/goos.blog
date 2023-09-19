---
title: "Adventures with MacOS"
date: 2022-04-13T00:00:01-04:00
draft: false
tagline: ourhardworkbythesewordsguarded
slug: macos-adventures
---

#### *why not just install linux dude?*

I've always liked ~~OSX~~ MacOS, dating back to when we had iMac G1s (and later G4s) in our school computer lab.

![iMac G1 in Aqua](/2022/04/macos_adventures/imac-g4-imac-g3-comparison.jpg)

Remember those? They were really nice. We should go back.

Back when I was in uni I used some suuuuper sketchy patched installers to run Mavericks (iirc) on my FX-4350 based system.
Back then documentation was nonexistent, you had to read through scammy-looking forums for answers, etc etc etc bullshit.
But you know what? It worked. It worked surprisingly well. I even played quite a bit of Team Fortress 2 on it.
For whatever reason I eventually I switched to Arch+KDE4 (remember tabbing together arbitrary windows?), but I've always fondly remembered those osx days.

I'm not sure what inspired me to try running ~~OSX~~ MacOS this time around, but I'm very happy to report that the situation has improved quite a bit.
It's like adults have taken over the whole ecosystem. Most things are documented, everything is on Github instead of totally-not-a-virus.com, etc.

My current build is a 5600x with an rx580 (wanna sell me something better at msrp?). I certainly didn't plan this build for macos, but I'll be damned if I can't get it working.
After all, if I could do it when I was dumb enough to go to uni, I should be able to do it now, right?

### The Starting Lineup

First things first, take a careful inventory of your system, note down the exact hardware ids you have, blah blah blah.  
Don't forget to read all 50000 pages of the [OpenCore Documentation](https://dortania.github.io/OpenCore-Install-Guide/).  
After that you need to do this that and the other thing.

Screw that, lets dive right in.

### The Plan (initially)

My plan at first was to install Monterey as a vm on [Proxmox VE](https://www.proxmox.com/en/proxmox-ve).
I've been using PVE for years both at work and on my personal iron, and I'm no stranger to PCI passthrough, so I figured this would be a cinch.
[This guide](https://www.nicksherlock.com/2021/10/installing-macos-12-monterey-on-proxmox-7/) made it look pretty easy, and it was.

GPU passthrough was straightforward. Enable iommu, setup some vfio stuff, and you're done.

Emboldened by the almost effortless gpu passthough, I passed through the usb3 controllers. *"There's no way this works, right?"*

It did. It literally justwerked™, no kexts or device-id changes required.

Apparently Apple likes AMD these days? Or maybe these usb3 controllers are uh, class compliant? Is that even a thing for things that live in the chipset?

In the past I've had good luck switching between different Windows and Linux vms, moving the gpu around in a digital game of hot potato.
Knowing that I wouldn't really be able to game on MacOS no matter how stronk my hardware, I had this workflow pictured:

- Monterey, Windows, and Linux (possibly multiple, more below) vms on the same host
- Suspend-to-disk the current vm
- Switch to a different vm
- The gpu gets released by the old vm and picked up by the new one
- It all justwerks, I don't have to do a full reboot, and because I'm suspending to disk, I don't even have to re-open all my programs!

Of course, it's never quite as easy as you think it will be.
In my case, it *almost* completely worked.
Going from MacOS -> (Linux, Windows) worked perfectly (after I installed the [Vendor Reset kernel module](https://github.com/gnif/vendor-reset)).

Going back to Monterey though... Didn't work. MacOS just wouldn't pick up and reactivate the gpu. Sadface.

I was also faced with the vm not grabbing the gpu on first startup. I had to do this each time the host booted, and it was annoying:

`qm start 3001 && sleep 5 && qm stop 3001 && sleep 5 && qm start 3001`

I'm sure someone could figure it all out, but I'm not a hardware hacker by any means (the last time I wrote C was pen on paper in CS101).

I had gotten a fresh taste of the MacOS life, and I wasn't going to settle for Windows or Open Sores on my desktop.
It was time for a different plan.

### The Plan (reality check)

Knowing that I was getting pretty deep into the black magik voodoo tier of host/guest interaction, I decided to take a step back.

*"It isn't that bad restarting programs, it's really just a few terminal windows and discord, I guess we can do a true multi-boot...
Yeah performance inside the vm is okay, but I want native power management and stuff like that.
Might as well install on iron.*

Reality checked, I did a quick `zfs send | ssh root@maelcum zfs recv` to save my Monterey vm on my nas just in case I had to go back, and started actually reading OpenCore's documentation.

First things first, we need some install media for MacOS. The guide recommends starting with Big Sur or something old like that.
Here at goos.blog, we don't use old software, ever. Hardware, sure, that's just cheaper. But all software is free (if you try hard enough), so why use old things?

Remembering how long it took the macos netinstall to run from a minimal boot usb, I created a full one.
It took well over three hours (a good hour longer than the netinstall took on the vm) to download and flash to a reasonably good usb3 thumbdrive from inside the vm. Second sadface.
That was the easy part.

Not that any of this was particularly hard, but that was the easy part.

I'll spare you the boredom of going through the opencore configuration, if you're trying to do this you should be reading the documentation, not a blog post.
I ended up booting a livecd on my laptop to run ProperTree because I still cba to figure out how to mount an esp on Windows.
It didn't take much for me to get to a working installer, some kexts here, some patches there, about 10 roundtrips back into the vm to make edits, and here we go!
Oh... Wait... Why aren't any of my disks showing up?

Like always, I had forgotten something.
This time around the "something" was an entire section of the documentation detailing how some sata controllers aren't supported and need an additional kernel extension.
Easy fix, just annoying that I had glazed over yet another thing.

At this point it was 5am (from the wrong direction). I went to bed.

### The Aftermath

I awoke at noon, somewhat refreshed, still buzzing on the "did a cool thing" high.

Going into my office, I see that the desktop is in s3 sleep. *"Huh, didn't think that'd work out of the box"*

I tap ctrl a few times and to my delight the system wakes up and I'm presented with a login screen.

It didn't take long to get things setup. Firefox, plex, vscode, my various email accounts (apple mail > outlook / thunderbird), and I'm good to go.
I do most of my development and testing work in vms on armitage, my stronk dual socket system with lots of wam and ssd, so my actual dev envs are pretty portable.

### The Current State Of Things

As I type this on the Monterey side of my desktop *...tabs over to ebay tracking and hits refresh...*
I'm waiting on another RX580 to come in the mail. This one is going to go into armitage (dell t7910) and is going to be used for a ***GAMING SERVER***
(I've been jealous of [tmick0's setup](https://lo.calho.st/posts/gaming-vm/) ever since I pointed him in the right directions on it).
Like him, I use a steam link on my teevee, and I'll also be able to stream games to my desktop.
Once that's setup the only time I'll have to leave macos is for playing CS:GO - it *runs* on macos, but uh... shittily.
I have grand ideas for a boot-to-csgo linux install, we'll see if I find a roundtuit for that project.

Was it all worth it?

Yes (even though I've spent $700 in the past two days on more wam and a gpu for armitage).

Could I have just stuck with winderp or installed a linux?

Sure. But where's the fun in that?

![Screenshot of "About this Mac" and "Neofetch"](/2022/04/macos_adventures/great_success.png)
