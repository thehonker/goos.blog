---
title: "Hammer VM (Or: How I Learned To Hate the Quadro k2000)"
date: 2022-04-14T00:00:00-04:00
draft: false
tagline: "it's hammer time"
slug: hammer-time
---

### *so i just passthrough the quadro to another windows vm for hammer and stuff*

Sometimes I like making CS:GO Surf maps. I've only released [one](https://steamcommunity.com/sharedfiles/filedetails/?id=2404967658), but I have grand ideas for several more.

## The need

In order to support this now that I'm running Monterey on my desktop, I needed a winderp env.
I could combine this with the **GAMING VM** I have planned, but no, I don't want to.
I have a halfway-decent quadro k2000, so passing it through to a vm on Armitage should be easy enough, right?

I've passed this card through on this host before (albeit on cpu1 instead of cpu0), so I start out pretty confident. I click the buttons, add the pci device to the host config, make sure it's bound by vfio-pci, etc.

## Oh No

Except... The moment I start the vm, the host console is FLOODED with mcelog events, leading to an eventual panic and cpu reset.

Not Good. Along with the kernel panic there was some amount of human panic. Was Armitage dying (right after I spent lossa money on new goodies for it)?

The errors are logged to bios (this is a workstation with ecc after all), so I swap the stick of wam in question over to another slot.
After doing that though, the system just crashed even faster, and didn't log the problem stick to bios. I pulled the plug and went to bed.

## Trying again

I wake up the next day with a clear head and some fresh ideas.
First thing, I booted the vm without the quadro card attached. It came up just fine, and the host was happy.
:thonking:

I then created a vm that would allocate 126 of the 128gb of wam on the host. It came up just fine, and the host was still happy.

Was I imagining the mce events? No, they're right there in the bios log still... Odd.
I attached the quadro to the winderp vm and started it again. Instant mcepanic and reset.

```none
[  510.749751] DMAR: DRHD: handling fault status reg 2
[  510.749757] DMAR: [DMA Read] Request device [03:00.0] PASID ffffffff fault addr fffffff000 [fault reason 06] PTE Read access is not set
[  510.837379] DMAR: DRHD: handling fault status reg 102
[  510.837384] DMAR: [DMA Read] Request device [03:00.0] PASID ffffffff fault addr fffffff000 [fault reason 06] PTE Read access is not set
[  510.847689] DMAR: DRHD: handling fault status reg 202
[  510.847693] DMAR: [DMA Read] Request device [03:00.0] PASID ffffffff fault addr fffffff000 [fault reason 06] PTE Read access is not set
[  510.847720] DMAR: DRHD: handling fault status reg 302
```

This fun stuff was logged at vm start and right before the mce explosion. It also logged whenever I did a reset of the gpu from inside the guest.
It was starting to smell like a rom-bar type issue. I don't know what address that is referenced in those dma errors, but it sure smells like the guest is trying (and failing) to read the vbios.

On a hunch, I swapped the quadro with the boot graphics. The quadro had been in the blue slot, which is probably the "primary" pcie slot.
Now it was in a x4 slot, which in theory would restrict performance, but who cares, I just need it to drive some light opengl shit.

I was no longer getting mcedeath each time I started the vm, but I was still getting error 43'd in the guest. I tried with `rombar=0`, however that lead to a full host lockup.

I *really* wish my pi-kvm was working. Hopefully that vga -> hdmi adapter I ordered works. If not I guess I can try an inline edid emulator. More on that later though.

I decided to try passing the vbios to the guest as a `romfile=` param. This didn't log any errors, but did rather quickly lead to a whole host lockup, probably right around the time the guest activated the gpu.

## Giving up (for now)

It was at this point I gave up and removed the quadro from Armitage.
I'm not sure if the card itself is bad, or if it just hates the pcie layout on the t7910.
I can use spice/qxl and it should be enough.
If not, I'll order a firepro or something. I'm done with nvidia. At least for today.

## virt-viewer on macos

For the qxl driver, you really need a proper spice client.
There's an official .app these days, but I wasn't able to get it to ingest the spice file that pve generated.
Additionally, it wasn't providing the .vv extension with the download, so I couldn't associate it that way.
I ended up installing virt-viewer via homebrew.
I found [this post](https://gist.github.com/tomdaley92/789688fc68e77477d468f7b9e59af51c), and decided to setup an automator.app to launch it directly, instead of feeding it the downloaded .vv file from pve.
I ended up hardcoding the vmid, as this is (for now) the only vm on armitage using qxl.
If there are any more in the future I'll probably do the same, I like having direct shortcuts.

## Hammer Time. Stop.

Now that I had a working vm, I needed hammer and friends. I download steam and do the thing, then I copy over hammer++ (which is now available for csgo!).
Small problem though: The mouse in the 3d preview... is kinda screwy.
Even after dropping the sensitivity to the lowest, it's still basically unusable.
I feel like this would be resolved with a gpu and using vnc or rdp. It would also probably allow me to run csgo to quickly test maps.

Digging through the junk bins, I found two more gpus to try. A radeon 7570, and a nvidia gt1030.
Given that the 1030 is several years newer than both the radeon and the quadro, I decided to try that first.
The vm booted right up with nothing scary printed to host dmesg. That's a good start.
After removing the quadro drivers and installing consumer ones, the gpu started right up.

Hammer was much smoother and better behaved now.
I could even play csgo at a good enough framerate for testing maps, if I could find a remote desktop setup that wasn't so latent.

## Remote Desktop that Doesn't Suck

Up until this point I've been accessing the guest using vnc and rdp, both of which kinda suck for high-framerate things.
Sadly, the GT1030 does *not* have nvenc or hardware encode of any sort. That nixes Parsec and Moonlight.
Rainway worked, but it captured the mouse HARD and didn't let it go. That's no bueno for a remote desktop.
Time to try the radeon card. (spoiler alert: it didn't work for parsec either).

Given that the 1030 had given me usable framerates in csgo, I decided to swap that back in.
After editing /etc/modprobe.d/vfio.conf for the 10,000th time today, the vm was back up and running with the 1030.

I also tried out (and really liked) reemo for remote desktop. I think this is the winner.
It has the option to switch between relative and absolute mouse modes, which I haven't seen before.
This completely fixes my 3d preview issues in hammer, which is really nice.

Now that I have hammer running in a dedicated environment, I feel motivated to work on maps again.
I also have one less reason to ever reboot into my winderp install.
