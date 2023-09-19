---
title: "The HP Revolve 810 G1"
date: 2022-04-13T06:00:00-04:00
draft: false
tagline: "it's too cute to not try"
slug: revolve-810-g1
---

#### *wow that's a cute laptop, too bad it runs windows*

I was given this laptop with a dead battery and a keyboard sans Lshift.  
It looks like a macbook, so lets turn it into one before fixing it!

![HP Revolve 810 Stock Photo](/2022/04/revolve-810-g1/revolve-810-g1.jpg)

## Don't Panic

This hardware is older and more specialized, this won't be nearly as straightforward as my desktop was.
Because of this I'll be going a bit more in depth on the weirder things. You can take that as a bad thing or a good thing, your choice.

### The setup

Like before, we start with the installer.
Spoiler ahead, I already tried this with Monterey and failed, so I'll be typing this as I attempt it with Big Sur 11.6.5
I'll gloss over how to create the installer as dortania's documentation is plenty.

Our first stop is into a linux livecd to collect some info on our hardware. The debian live+nonfree is in my urlbar history, so that's what I grabbed for this.
Anything with lspci and lshw will be fine.

```none
root@debian:~# grep -i "model name" /proc/cpuinfo 
model name : Intel(R) Core(TM) i7-3687U CPU @ 2.10GHz
model name : Intel(R) Core(TM) i7-3687U CPU @ 2.10GHz
model name : Intel(R) Core(TM) i7-3687U CPU @ 2.10GHz
model name : Intel(R) Core(TM) i7-3687U CPU @ 2.10GHz

root@debian:~# lspci | grep -i vga
00:02.0 VGA compatible controller: Intel Corporation 3rd Gen Core processor Graphics Controller (rev 09)

root@debian:~# dmidecode -t baseboard
# dmidecode 3.3
Getting SMBIOS data from sysfs.
SMBIOS 2.7 present.

Handle 0x0002, DMI type 2, 16 bytes
Base Board Information
  Manufacturer: Hewlett-Packard
  Product Name: 18F8
  Version: KBC Version 51.27
  Serial Number: PDMQT1A2F4J0SU
  Asset Tag: Not Specified
  Features:
    Board is a hosting board
    Board is replaceable
  Location In Chassis:  
  Chassis Handle: 0x0003
  Type: Unknown
  Contained Object Handles: 0

Handle 0x0005, DMI type 10, 6 bytes
On Board Device Information
  Type: Video
  Status: Enabled
  Description: 64

Handle 0x0006, DMI type 41, 11 bytes
Onboard Device
  Reference Designation: 64
  Type: Video
  Status: Enabled
  Type Instance: 1
  Bus Address: 0000:00:02.0

Handle 0x0007, DMI type 41, 11 bytes
Onboard Device
  Reference Designation: WLAN
  Type: Ethernet
  Status: Enabled
  Type Instance: 1
  Bus Address: 0000:03:00.0

root@debian:~# dmesg | grep -i input
[    2.485380] input: AT Translated Set 2 keyboard as /devices/platform/i8042/serio0/input/input0
[    2.762604] input: Sleep Button as /devices/LNXSYSTM:00/LNXSYBUS:00/PNP0C0E:00/input/input5
[    2.762682] input: Lid Switch as /devices/LNXSYSTM:00/LNXSYBUS:00/PNP0C0D:00/input/input6
[    2.762729] input: Power Button as /devices/LNXSYSTM:00/LNXPWRBN:00/input/input7
[    3.003341] input: Video Bus as /devices/LNXSYSTM:00/LNXSYBUS:00/PNP0A08:00/LNXVIDEO:01/input/input12
[    4.026364] input: Atmel Atmel maXTouch Digitizer Touchscreen as /devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.1/1-1.1:1.0/0003:03EB:840B.0001/input/input13
[    4.026449] input: Atmel Atmel maXTouch Digitizer as /devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.1/1-1.1:1.0/0003:03EB:840B.0001/input/input14
[    4.026553] hid-generic 0003:03EB:840B.0001: input,hiddev0,hidraw0: USB HID v1.11 Device [Atmel Atmel maXTouch Digitizer] on usb-0000:00:1a.0-1.1/input0
[    4.026683] hid-generic 0003:03EB:840B.0002: hiddev1,hidraw1: USB HID v1.11 Device [Atmel Atmel maXTouch Digitizer] on usb-0000:00:1a.0-1.1/input1
[    4.027623] hid-generic 0003:0483:91D1.0003: hiddev2,hidraw2: USB HID v1.10 Device [Љ Љ] on usb-0000:00:1d.0-1.6/input0
[    4.287256] input: AlpsPS/2 ALPS GlidePoint as /devices/platform/i8042/serio4/input/input11
[    9.390753] input: HP Wireless hotkeys as /devices/virtual/input/input15
[    9.658174] input: PC Speaker as /devices/platform/pcspkr/input/input17
[    9.668017] input: HP WMI hotkeys as /devices/virtual/input/input16
[    9.769920] input: Atmel Atmel maXTouch Digitizer as /devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.1/1-1.1:1.0/0003:03EB:840B.0001/input/input18
[    9.772380] input: Atmel Atmel maXTouch Digitizer Stylus as /devices/pci0000:00/0000:00:1a.0/usb1/1-1/1-1.1/1-1.1:1.0/0003:03EB:840B.0001/input/input19
[    9.772731] hid-multitouch 0003:03EB:840B.0001: input,hiddev0,hidraw0: USB HID v1.11 Device [Atmel Atmel maXTouch Digitizer] on usb-0000:00:1a.0-1.1/input0

root@debian:~# lspci | grep -i network
00:19.0 Ethernet controller: Intel Corporation 82579LM Gigabit Network Connection (Lewisville) (rev 04)
03:00.0 Network controller: Intel Corporation Centrino Advanced-N 6235 (rev 24)
root@debian:~# 
```

Now that we know what hardware we have we can gather up the stuff we need to boot into the installer (we'll figure out the rest after).

First I run `installinstallmacos.py` again, this time selecting Big Sur 11.6.5.
It took quite a while to download:

![Screenshot of wget](/2022/04/revolve-810-g1/hide_the_pain.png)

And even longer to flash. Maybe my usb3 ports... aren't usb3.

*hey siri, remind me tomorrow to look into usb mapping*

I then took a quick run through the "gathering files" section of the opencore install guide.
I felt pretty confident that I kinda knew what I was doing, so I speedran the kexts and acpi sections (this will bite me later, I'm sure).

At this point my efi looked like this:

![Screenshot of EFI Folder](/2022/04/revolve-810-g1/efi_take1.png)

I plugged the drive in and attempted to boot. The boot menu came up, but after passing to the initramfs or whatever (how does macos boot work anyway?) the framebuffer was out of sync.
Looked like the horizontal signal wasn't quite right.

![Picture of screen tearing](/2022/04/revolve-810-g1/wut.jpg)

> I decided to let it cook.

After a few minutes, I was presented with the Big Sur recovery/installer screen.... In Russian.

For some reason, OpenCore defaults to Russian for the locale. Probably because most of the big nerds in the hackintosh world are from over there.
I was aware of this, and had flipped the necessary bits in my opencore config. For some reason though, this didn't work on the wee laptop.

It was now the wrong side of 5am (again!). I went to sleep.

### Why the f*!$ is the installer still in russian?

![Screenshot of MountEFI application](/2022/04/revolve-810-g1/best_friend.png)

> Here I am again my old friend. Mounting an ESP to make yet another change to a `config.plist`.

I fire up ProperTree for what must be the 200th time (they should add an hour meter), and confirmed what I knew last night: I had indeed changed the language entry.

![Screenshot of ProperTree application](/2022/04/revolve-810-g1/if_this_doesnt_work.png)

Frustrated at that not working, I threw in an entry that would delete that parameter, forcing opencore to rebuild it.
I think that's what this does at least. Time to go back to the laptop and see if it boots.

After too long a time, I'm presented with the installer again. This time, it's in english!
I (try to) fire up terminal.app to see what's what (does my disk show up?), but my trackpad doesn't work.
Sadface, but that's on me for not pulling in the right kexts.
We'll fix that later, for now I plugged in a mouse into the other usb port. Trackpads suck anyway, trackpoint4lyfe.

Thankfully, the keyboard did work. A quick `diskutil list` showed that indeed I did have a `/dev/disk0` of the right size and shape. Time to install!

But first, I needed a partition name. After a few minutes of thinking, I decided to take the lazy out and just name it `macos-root`.
APFS is neat. It smells like LVM, and probably acts somewhat like it. I should learn more. Although to be fair I don't know much about ntfs or ext4 either.
I'm a ceph (and zfs) person, smol volumes have never mattered much to me.

I click through the installer and got comfy, this could take a while. I like how macos has the installer log right there.
I also like how it's basically just `dmesg -w`, so if you turn up the verbosity you can see all the kernel output.
Wait. 1 pm already? It was noon when I woke up, and I've done basically nothing today!

After about 20 minutes the first stage of the install was complete, and the laptop rebooted itself (as intended).
It continued on, framebuffer still broken for kernel `-v` output, but whatever. Another half hour or so and it should be done!

While waiting for that, I did a little online shopping - this time I was looking for a passthrough edid faker for my blikvm (more on that in a later post).
The tl;dr there is that it refuses to capture video from everything I've tried - except for another raspi. Go figure.
And yes, I've tried dozens of different edid settings on the csi capture card - no avail. The target just acts like there's no cable connected.
Any ideas? Shoot me an email (please).

Been at *About 7 minutes remaining* for half an hour now... How much longer will this take?

Oh. There it goes. ~~Four~~ Five *long* reboots later and I have the OOBE to click through.

After that, I land at the desktop. Woohoo!

![Screenshot of Big Sur desktop](/2022/04/revolve-810-g1/great_success.png)

First things first, let's dd our esp over so we don't need the install usb anymore:  
`dd if=/dev/disk2s1 of=/dev/disk0s1`

## Making all the things work

Let's take inventory of what works and what doesn't.

#### Not working:

- Wifi (needs a kext)
- Webcam
- Audio (needs applealc + layout)
- Trackpad multi touch (surprising that it works at all , as it didn't work in the installer)
- Lshift key (this is getting very annoying very fast)
- Touchscreen
- Battery info and other sensors
- Volume keys
- s3 sleep

#### Working:

- Ethernet (took a minute to warm up but it did start working)
- External displayport connector (suprising)
- Bluetooth (without any kexts!)

I turned on screen sharing so I didn't have to stare at a 11" screen. This is just vnc so it's not very sekure.
*hey siri, remind me to turn off vncd on lady3jane*
Yeah, I came up with a hostname cooler than what i use on my desktop. Do you know what book(s) I'm taking names from?

It didn't take long to get the basics setup. Firefox (with "tree tabs" extension), xcode tools for git and friends, propertree, etc.

After some more reading and downloading, this is what my efi looked like:

![Screenshot of efi directory](/2022/04/revolve-810-g1/efi_take2.png)

> Note that I've replaced VoodooPS2Controller.kext with [this fork](https://github.com/BAndysc/VoodooPS2), which gives me proper right-click behavior.

I added stuff for wifi, battery, and other sensors. I didn't do audio yet because I don't know my alc codec (yet).
I reboot annnnd... Boot device not found? Seems my boot entry has dropped off the firmware.
No problem, HP is a bro and provides a "boot from efi file" option which gets me back into big sur.
Battery info is still not displaying - I wonder if this is because it doesn't detect the battery (linux showed no battery connected too), or if I have the wrong kext.
Guess I'll have to just buy a new one and hope it works.

But first. Benchmarks.

Let's see what this old ivy bridge can do. Remember that we're calling ourselves a MacBookAir6,1 but we actually have a i7-3687u under the hood.
I'm not expecting much, but that's not a bad thing.

While the benchmarks were running, I went looking for fixes for the little things. I had the trackpad working properly now, but that was about it.

[This repo](https://github.com/scarymen1325/opencore_hp_810_g2/tree/BigSur/OC/Kexts) helped me find what kexts I might need (it's for the gen2, but appears to be pretty close to the gen1)

Here's the benchmark results. Better than what i was expecting:

[cpu benchmark results](https://browser.geekbench.com/v5/cpu/14297688)

[opencl benchmark results](https://browser.geekbench.com/v5/compute/4659524)

Now a quick trip into a linux liveusb to determine our audio layout. I have the 92hd91bxx in this system, which gives me a list of layouts to pick from.
While I'm in here I should fix the bootloader firmware entry:

`efibootmgr -c -d /dev/sda -p 1 -L "MacOS" -l '\efi\oc\opencore.efi'`

Except... That didn't work. Neither do my new kexts, the boot won't progress past 50%.
Part of determining that was to enable csm support - this allowed me to see the progress bar.
Will it fix the text framebuffer of verbose mode as well?

Back into the liveusb we go to add `-v` back to our boot args!

And hey, turning on csm did fix the text framebuffer! Some good news.

Oh. I remember now. You're supposed to disable injection of one of the intel bluetooth kexts. Derp.
Back into the liveusb. Again. I fire off the efibootmgr command again to no avail. Maybe I need to clear nvram again? I wonder if there's any bios updates for this lil guy...

There is a bios update available, I should apply that at some point.

Meanwhile, everything but audio and battery display is working! I think I can blame the battery display on the battery itself - linux was saying "bro no battery at all" as well.
Audio though, that's probably just the AppleALC layout...

This last trip into the liveusb I found my coded, it's a IDT92HD91BXX
ALC Layout 3 no joy. Let's try 84 ([why?](https://github.com/acidanthera/AppleALC/blob/master/Resources/IDT92HD91BXX/Info.plist#L47)) (spoiler alert: it didn't work either)
Neither 12, 13, or 33 worked either. Sadface. Now we have to boot into a liveusb again to run ssdttime and get a proper acpi mapping.

I really need to roll a custom liveusb with all these tools preinstalled...

Back in the liveusb, I run ssdttime and generate all my ssdts and patches. For the 300th time, I open propertree and add them to my config.plist.
The system boots afterwards, but none of my input devices work anymore. Grr. At least I don't have to setup the liveusb with propertree and friends again.
The culprit behind input devices failing was `SSDT-EC-LAPTOP.aml`. I think. After reverting from the generated one back to the generic one inputs work again.
Still no sound though.

After some messing around with ssdts, I have working audio!
The issue was, I was including the entire DSDT.aml thinking that was da wae. After removing that, the system boots and has audio and input.
I realize now that I did this on my desktop as well, and took a few minutes to fix that. I wonder if it'll improve performance at all?

At this point the only things left to do are to sign in with my apple account and get s3 sleep working.
S3 sleep is probably going to take some manual ssdt editing, which I *really* don't feel like doing right now.
It only takes a few seconds to boot, so for now I can live without it.

![Screenshot of about this mac and neofetch](/2022/04/revolve-810-g1/greater_success.png)

Next up: Getting it to run Monterey.
