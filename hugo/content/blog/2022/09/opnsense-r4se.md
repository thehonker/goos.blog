---
title: "OPNsense on the NanoPi R4SE"
date: 2022-09-25T00:00:00-04:00
draft: false
tagline: "dude where's my pcie?"
slug: opnsense-nanopi-r4se
---

#### *tl;dr s/0x82000000/0x83000000... but not really*

As I write this, I'm /still/ (48hr and counting) waiting for a freebsd/opnsense build to finish. \
I'm not sure what it's doing, looks like it's building every damn package in the ports tree. \
I'm impatient, so while we're waiting for that to cook (I hope it caches them for later use), let's get someone elses opnsense image working!

## first attempts
A good amount of google brought me to the [rockchip partition table info](https://opensource.rock-chips.com/wiki_Partitions) which gave me the typical offsets that prebuilt images would look for. \
[This page](https://opensource.rock-chips.com/wiki_Boot_option) helped as well.

I knew that I'd need a different DTB for this board to get emmc working. I figured the boards were similar enough that I could just copy the one for the R4S, change the model strings to match the new board, and rm the "disable emmc" section.

Spoiler alert... This almost worked.

## a shot in the dark

I don't remember how many combinations and permutations of partition layout, vendor uboot vs freebsd uboot, trying to load in my own dtb via the freebsd bootloader shell, etc screwball things I tried. \
At one point I was combining the uboot provided by friendlyelec alongside the openwrt image with an opnsense 22.1.10 image. \
This got me into useland but there were other issues - if I let it go all the way to freebsd's multiuser runlevel it would kernel panic when the "device manager" was started. \
I could boot into singleuser mode without a panic but eth1 was missing. \
I dove into friendlyelec's horribly disorganized sources looking for the dts for the r4se. Didn't find it anywhere. \
~~If you're able to find the one they use, send me an email.~~ nvm I found it \
I eventually found and started using [this dts](https://raw.githubusercontent.com/DHDAXCW/lede-rockchip/22da83e80131a9883714f747e13b6f4d4cc343c9/package/boot/uboot-rockchip/src/arch/arm/dts/rk3399-nanopi-r4se.dts), but it hasn't gotten me any further.

![rkdevtool screenshot](/2022/09/opnsense-nanopi-r4se/rkdevtool.PNG)

Gah.

## do the easy things last

I spent way too much time trying lots of stupid things before i finally just did `git clone uboot`. \
I got further and learned more in the hour after that than the 6 before.

After poking around the uboot sources I had a shaky grasp on how things worked. \
I copied in my dtb and hit the build button. \
After gcc did its thing and copying the output to my rkdevtool directory, I hit the maskrom button for what feels like the 10000000th time today. \
A few minutes later and I'm back into singleuser mode, this time without having to shuffle baudrates 4 times and manually load the dtb at the loader prompt.

```none
root@:/ # dmesg | grep pci
rk_pcie_phy0: <Rockchip RK3399 PCIe PHY> mem 0-0xff76ffff,0-0xffff on rk_grf1
pcib0: <Rockchip PCIe controller> mem 0xf8000000-0xf9ffffff,0xfd000000-0xfdffffff irq 6,7,8 on ofwbus0
pcib0:  At least memory range should be defined in DT.
device_attach: pcib0 attach returned 6
```
> sussy af. someone go hit the emergency meeting button

2am. Time to make this a tomorrow thing.

After figuring out the freebsd equiv of `lsmod` (it's `kldstat`), I noticed that the re module wasn't even loaded. Device won't show up if the kmod isn't there, derp. \
Another trip through the workflow to get a force load added to loader.conf and....

It still panic'ed. But. BUT. Different error!

```none
Launching the init system...done.
Initializing...........done.
Starting device manager...Fatal data abort:
  x0:                0
  x1: ffffa00006358580
  x2:                3
  x3: ffff0000d994f650 (re_netmap_txsync.__cnt + d866d4dc)
  x4: ffff0000d994f5d8 (re_netmap_txsync.__cnt + d866d464)
  x5:                0
  x6:                0
  x7: ffff0000d994faa0 (re_netmap_txsync.__cnt + d866d92c)
  <snip>
```

Before it was a generic vmem abort. Now I have some confirmation that it is indeed the pcie nic causing issues. \
I went digging into the friendlyelec uboot repo. It /had/ to be a device tree fuckup. \
Finally, after checking 4 different branches, I found the vendor dtb sources.

## well-defined spaghetti

Immediately I found my smoking gun (I thought...):

```none
&pcie0 {
    max-link-speed = <1>;
    num-lanes = <1>;
    vpcie3v3-supply = <&vcc3v3_sys>;

    pcie@0 {
        reg = <0x00000000 0 0 0 0>;
        #address-cells = <3>;
        #size-cells = <2>;

        r8169: pcie@0,0 {
            reg = <0x000000 0 0 0 0>;
            local-mac-address = [ 00 00 00 00 00 00 ];
        };
    };
};
```

Up until now I had this in that section:

```none
&pcie0 {
    max-link-speed = <1>;
    num-lanes = <1>;
    vpcie3v3-supply = <&vcc3v3_sys>;
};
```

The rest of that must be defining that there's actually a device there. \
Yet another trip through the workflow and.... Nothing. \
I forgot to set the model string on the latest ^c^v'ing of the dts. \
Fixed that, reflashed yet again, reset the board, waited for it to boot, and....

Kernel panic.

Kernel panic.

Kernel panic.

Finally googled the right thing.

```none
handle_el0_sync()
```

## closing: unable to tie cause to effect

That brought me to an old forum thread saying that `if_bridge` causes panics if `VNET` is enabled in the kernel. \
So like, I'll blocklist the bridge module, and it'll justwerk, right?

Turns out there is no freebsd equiv of the `blacklist some_module` directive that exists in linux.

At this point I wanted to see what was going on with the R4S (non-emmc) images I had. \
I burned the 22.1.9 image from personalbsd and booted it up. \
Immediately I noticed this:

```none
pcib0: <Rockchip PCIe controller> mem 0xf8000000-0xf9ffffff,0xfd000000-0xfdffffff irq 6,7,8 on ofwbus0
pci0: <PCI bus> on pcib0
pcib1: <PCI-PCI bridge> at device 0.0 on pci0
pcib0: failed to reserve resource for pcib1
pcib1: failed to allocate initial memory window: 0-0xfffff
pci1: <PCI bus> on pcib1
re0: <RealTek 8168/8111 B/C/CP/D/DP/E/F/G PCIe Gigabit Ethernet> at device 0.0 on pci1
re0: Using 1 MSI-X message
re0: turning off MSI enable bit.
re0: Chip rev. 0x54000000
re0: MAC rev. 0x00100000
```

There's a bridge. There's a pcie bridge at node 0. Back to the dts for the R4SE.

But first, let's do a quick sanity check. I flashed the vendor openwrt image and.... no eth1 \
?!?!?!?! \
Did I burn out the nic on this poor little guy? \
I'm using the other emmc one as my firewall at this moment.... \
Time to setup the non-emmc with opnsense so I can use that as the firewall for now.

It works. But having to shuffle my console baudrate is annoying. \
Mainline u-boot has it set to 1500000bps for some reason. \
I dd the new uboot onto the sdcard with a working opnsense install, popped it into the R4S, and.....

Kernel panic.

This is with mainline u-boot dtb sources. I'm starting to question my own sanity. \
What dts are other people using??? I fired off a fourm post asking for help, then I found [the repo](https://github.com/S199pWa1k9r/ports/tree/master/sysutils/u-boot-nanopi-r4s-2020.07/files) with the dtb sources for the opnsense images I've been working with.

They were. The same. As. The. Vendor. Sources. (they weren't)

4am this time. I'm just now thinking "what if I could pull the live device tree from a working openwrt install on a r4se, decompile that shit, and cherrypick out what we need to make this work?"

Turns out that's a 1-liner:

```none
dtc -I dtb -O dts /sys/firmware/fdt -o r4se.dts
```

On a hunch I decompiled the compiled dtb that I was trying to use and compared the two. \
Immediately I saw two things:

- Address ranges on vendor start at 0x82000000, whereas mine is starting at 0x83000000
  - sus af

- "compatible" field set to ` nanopi-r4s `. I have been using ` nanopi-r4se `...
  - super sus, especially as the dtb would not load when I had forgotten to add the E

Anyway. Back to the R4SE. I rebuild uboot again without any changes to have a baseline, and...

## oh no

~~Kernel Panic~~

Just kidding, although I wish I wasn't:

```none
GUID Partition Table Header signature is wrong: 0x0 != 0x5452415020494645
find_valid_gpt: *** ERROR: Invalid Backup GPT ***
GUID Partition Table Entry Array CRC is wrong: 0x307a16bd != 0xbae994fc
find_valid_gpt: *** ERROR: Invalid GPT ***
GUID Partition Table Header signature is wrong: 0x0 != 0x5452415020494645
find_valid_gpt: *** ERROR: Invalid Backup GPT ***
GUID Partition Table Entry Array CRC is wrong: 0x307a16bd != 0xbae994fc
find_valid_gpt: *** ERROR: Invalid GPT ***
```

I tried a few things, did a few googles, didn't find much info about anything. \
Time for a sanity check.

I flashed vendor openwrt and it booted right up. EMMC is fine. Yay! \
But why won't it boot my image anymore??? \
On a hunch I flashed first the image, then my u-boot over it.

Kernel panic.

Okay, at least I can get back to that state again.

I don't know why but I tried removing the 'e' from 'r4se' in the ` compatible ` field in the dts.

The board booted.

No kernel panic.

I noted that I was using vendor dts. Switched to mainline. Kernel panic.

## vindication

Then I saw it:

![theres_yar_problem](/2022/09/opnsense-nanopi-r4se/diff.png)

See it there? Open up the image in a new tab if you need to. \
~~Someone fatfingered the address range when they were rewriting the dts (why did they do that? vendor dts is fine?)~~

I had just spent more time than I'd care to admit trying all manner of different things. \
And it came down to this. PCIe address ranges. \
Wasn't the model string (derp), wasn't the finer grained pcie stanzas in the toplevel dts, wasn't busted emmc or chipsets. \
Address. Ranges.

[This commit](https://github.com/friendlyarm/kernel-rockchip/commit/8efe01b4386ab38a36b99cfdc1dc02c38a8898c3) validated my experiences of the past 2 days. Apparently this was an issue in linux with some nvme disks? \
It's clearly a Big Issue in freebsd.

Because of the way freebsd set things up, someone needs to update binaries in the ports tree (you heard me) with this fixed.

## a ~~new~~ lost hope

After a lot of fucking around and a little bit of finding out, I went back to the non-emmc R4S. \
I wanted opnsense at home. Really bad. \
Thankfully it boots right up. I do the thing and configure it. The way I have my network laid out physically lends itself well to warm-swapping firewalls.

![the board of networking](/2022/09/opnsense-nanopi-r4se/IMG_3416.jpg)
> A dumbswitch + mac spoofing on wan means my v4 doesn't change that often. \
> Although whenever I want a new one verizon will just shit one out. \
> ...on a whole new subnet most of the time. Kinda annoying actually.

30 seconds or so of downtime as arp/fdb tables clear/refresh and I'm back online.

Within 5 minutes I was experiencing the infamous re driver watchdog timeouts. \
I tried a few combinations of offload on/off, etc. \
Tried the realtek provided driver too - that instapaniced when the config manager added vlan interfaces. \
Woo. \
No matter what I did I couldn't get it to be stable for more than a few minutes at a time. \
That's... Unacceptable. And a big sad. I really wanted HA at home, and there's nothing like pfsync (yet).

## back to linux, where I belong

At this point I accepted that freebsd/openwrt on these guys in my configuration was basically a lost cause. \
Maybe if I wasn't running vlans it'd be okay. It certainly seemed that way in my initial testing.

My next step from here is to get a decent build of openwrt going for my own use. \
Then, I need to redirect this effort into plonkfw. The world deserves better.
