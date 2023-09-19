---
title: "The gaming server"
date: 2022-04-18T11:24:00-04:00
draft: false
tagline: "pew pew"
slug: gaming-vm
---

### *a somewhat elegant solution to a nonexistant problem*

MacOS is nice. Really nice. Really good. At most things...

Gaming, not so much. Windows still leads that pack (duh), with linux trailing closely behind these days.
So what does a man with a hackintosh on the desktop and some big iron in the closet do?
Slap a gpu in the iron and stream all the games to both the desktop and a steam link on the teevee!

### so many games, so little time

After the rx580 came in and was installed last night, I was ready to setup the vm that would consume it.
First I isolate the vm with vfio. tl;dr `vfio-pci.ids=1002:67df,1002:aaf0` is all you need on modern kernels.
Then a standard winderp install, q35, omvf, etc. Fight my way through the horrible oobe and I'm finally at the desktop.
Attach the gpu, detach the qemu display, and I'm done... Kinda.

This gpu has hardware encode (unlike my 1030), so I could use parsec for remote desktop while setting it up.
It's a bit nicer than reemo overall. I like the h265 support and 2fa.
I don't really like how it signed me out of the client between last night and today.
I get steam setup and click install on a bunch of games in my "play eventually" collection.

### a mostly uneventful process

After a few games finished downloading on the vm, I fired up steam on my desktop for a quick test.
I plug in my dualshock3, run [ds3activate](https://github.com/libsdl-org/SDL/issues/4923#issuecomment-966722634) (thanks apple for dropping support for it in Monterey...), and launch one of my favorite kill-5-minutes games, The Ramp.
Unsurprisingly, it fires right up and justwerks. Thanks valve!

### what about that 300gb of emulator games tho?

Over time I've built up quite a few emulator games. Sure, 200 of that 300gb is taken up by like four ps3 games, but there's quite a few retro, ngc, wii, etc games as well.

[Steam-rom-manager](https://github.com/SteamGridDB/steam-rom-manager) helps a ***lot*** with managing that.

It needs some manual overrides and massaging, but in the end I get a nice list of steam custom shortcuts for all my emul games.

"Ready to play" was looking pretty gross at this point:

![steam game list](/2022/04/gaming-vm/so_many_games.png)
> I think I'm going to remove a lot of these roms that I don't actually play.

I'm considering trying to make a collection manager, so I can have a collection that's `not emulated`.
Or, valve could add the NOT operator to dynamic collections. That'd be cool too.

Another annoyance - categories for non-steam applications don't carry between computers... Although they do show up on the steam link (as you'd expect).

So instead of having this nice organization structure, I have this mess:

![steam filters](/2022/04/gaming-vm/collections.png)
> vm on left, desktop on right (I rm'd some of the roms I was never going to play)  

From what I've read online, collections are stored ***in the cloud***, not a local vdf file like shortcuts.
Probably have to interact with the api to copy them, but a quick search on valvo's documentation page didn't show any publicly documented endpoints.
Oh well. At least I have search on my desktop.

A thought - what if there was a desktop steam link client?

![steam message](/2022/04/gaming-vm/ouchie.png)
> [Turns out there is... Rather was](https://store.steampowered.com/app/353380/Steam_Link/)... [But maybe there's hope?](https://steamcommunity.com/app/353380/discussions/9/3108017414028756807/)

With apple dropping i386 support back in catalina, I'm SOL. Again. Woo.

Yeah yeah I can just remote in with parsec and browse/play that way - and I might end up with that workflow.
I'll try both and see how it goes.

### a new hope

Some time passes. I cook dinner, feed the cats, take out the trash.

Then I think *"I never checked the actual apple app store, did I?"*

Sure enough:
![apple app store](/2022/04/gaming-vm/oh_look.png)
> There it is!

This is perfect for my needs. Thanks valve!

### cpu / node pinning

I should probably pin this vm to cpu0/node0.

A few `numactl` commands to see how my iron is laid out and I'm ready to ^c^v his hook script and config entries.
As I turn my dev vms back on, I'll want to pin them to cpu1, so i create both a `pin-numa0.sh` and `pin-numa1.sh`

This didn't quite work though.
The cpu pinning script was working (albeit with one failure call to taskset failing, not sure why yet), but the memory pinning wasn't.
Additionally, the `qm start` command was timing out every now and then. There is a `--timeout` flag, but it'd be nice if this was in the vm config instead so the webui would reflect this too.

Tossing this in the hookscript fixed the memory pinning. Yeah, kinda gross to flush page cache every time I start a pinned vm, but whatever:

```bash
if [[ "$2" == "pre-start" ]]; then
  echo 1 > /proc/sys/vm/drop_caches
fi
```

### final thoughts

Should you do something like this? Well, maybe.
If you have the extra hardware and the use case, go for it.
It's not that hard anymore, and the bennies are pretty nice.
I don't have to tie up a desktop for the steam link anymore, and I don't need to boot to windows for casual gaming.
