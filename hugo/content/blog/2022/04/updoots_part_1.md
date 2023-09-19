---
title: "Armitage upgrades - gpu, nvme, blikvm"
date: 2022-04-17T06:00:00-04:00
draft: false
tagline: "the strong get stronger"
slug: updoots-part-1
---

### *When packing this order to ship our inventory was off on these, we have 4 of the 8 sticks. We can refund you for 4 and send out the other 4, or we can cancel the whole order. Sorry for the mix-up.*

![packages](/2022/04/updoots-part-1/nerd_shit.jpg)
> I arrived home to four bits of nerd shit in my mailbox

I have several upgrades in mind for my lab box.

### blikvm

A while back I ordered a blikvm (I liked the board layout more than the pi-kvm, and that it uses a cm4) for this box, but I was unable to get video capture working at all.
I messed around with edid settings on the capture device for days with no joy. Tried several cables, different gpus even. Nothing.
`/sys/class/drm/whatever` showed disconnected on the output each time.
Feeling somewhat desperate, I ordered a vga -> hdmi adapter. Spoiler alert, it kinda works. I have video in bios, grub, and initramfs, but after that... nothing.
I'll have to play with edid settings again now that I have the adapter in play.
I'm probably going to order an inline hdmi edid emulator to see if that works better, as a bonus it'll be more compact.

I also found a remote power switch header on the board (thanks dell!) so I even have power button control. That'll be handy, I'm sure.

![pikvm webui](/2022/04/updoots-part-1/pikvm_webui.png)
> INTEL (R) CPU 0000  
> that's what you get for buying qa samples I guess

### gpu

With kvm access finally working, I moved on to the gpu - a sexy dell rx580 with 8gb of vwam. Why dell / reference blower layout?
There's only about a half inch of space above the top of the pcie slots. Thanks dell... When I first got this box, I had to cut some supports out of the lid to fix my 660 in there.
This gpu will be passed through to my ***GAMING VM***, which will be covered in a future post.

### nvme

A few years back I acquired 4x 1tb intel nvme without realizing it was in 22110 form factor. Whoops, that won't fit in anything.
I was too lazy to return it, so into the parts bin it went.
Until one day, I was quoting a new (to us) server for work, and I saw this card was a thing that existed:

![dell 0NTRCY](/2022/04/updoots-part-1/0NTRCY.jpg)
> dell p/n 0NTRCY

It takes 2x 22110 (or smaller) nvme bois and lets you plug them into a x8 slot - although you do require bifurcation support as there is no plx chip.
I only ordered one of the two that I'll need because I am unsure if armitage even supports bifurcation - there's nothing in the bios, but I've seen automagic bifurcation in the past.

Does it work? I'm about to find out! *hits enter on grub2*

It sure does!

```none
root@armitage:~# lsblk | grep nvme
nvme0n1   259:0    0 931.5G  0 disk 
nvme1n1   259:1    0 931.5G  0 disk
```

So now I just need to create a zfs pool for this and start setting up vms on it.

### fan controller

This is a simple one that I'd recommend for any dell workstation.
Apparently exhaust fans aren't required on a dual socket workstation.
HP has a different opinion on this, as do I.
So I added some exhaust fans. 2x 80mm delta somethings.
At full 12v, they're rather loud. They're the loudest fans in this system, by a noticeable margin.
$5 fan controller kit fixes that. Not much more to say about it.

### up next

I need to order another of those dell carrier cards, and the balance of the ram.
The ram seller messaged me saying they only had 4 of the 8 sticks I wanted.
Giant sadface, now I need to buy from 3 different vendors to get the remaining 4 sticks.
After that, I think I'll be done upgrading armitage for a while.
The only other thing I really want to do is to trade the 4x2.5 bay for a 6x2.5, and run boot/root off full size satabois.
Right now it boots from a pair of msata cards on a startech.com carrier.
It's.... acceptable. Not performant by any measure. dist-upgrades are pretty painful.

### edit - oops

Got tired of not having hardware encode on the 1030 for the hammer vm. That can live inside a dell business desktop I have anyway.
So I did this:

![rx 550 order page](/2022/04/updoots-part-1/oops.png)

Amd cards are easier to passthrough anyway.