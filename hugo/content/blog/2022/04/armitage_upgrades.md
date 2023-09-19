---
title: "Armitage upgrades"
date: 2022-04-13T05:38:14-04:00
draft: false
tagline: "MOAR BOOSTERS!"
slug: armitage-upgrades
---

#### *...the "Armitage" personality was constructed as part of experimental "computer-mediated psychotherapy"...*

![Armitage front view](/2022/04/armitage_upgrades/monolith.jpg)
This is armitage, my Dell t7910.  
It runs all sorts of long-lived and short-lived vms.

### Current specs

- Two E5-2667v4's
- 64gb of ram per socket
- VM Storage: LSI SAS3008
  - 4x Intel SSDSC2BB016T4 (1.6TB)
  - 4x HGST HE10 (10TB Helium)
- Boot Storage: Startech.com 2x mSata carrier card with Marvell chipset
  - 2x 64gb Sandisk whatevers
- Boot graphics: Old radeon 5450 or something
- Workload graphics: Quadro K2000

### Planned upgrades

![eBay purchase history](/2022/04/armitage_upgrades/goodbye_money.png)

I'm going to double the amount of ram, add a modest gaming gpu, and spin up a new nvme storage pool - assuming the motherboard supports bifurcation like the internet says it does.

If reddit is right, I'll order another of those dell cards for a total of 4x 1TB intel datacenter something nvme.

If not, I'll have to get a card with a PLX switch. [One from supermicro runs around 200 bones.](https://www.amazon.com/Supermicro-AOC-SHG3-4M2P-Full-Height-Quad-Port/dp/B07PNR1YSG/)

On another note, I'm not the biggest fan of the msata carrier card for boot/root.
I might get a [6-bay adapter](https://www.amazon.com/ICY-DOCK-Mobile-Comparable-Tray-less/dp/B01M0BIPYC/) for the ssds and move boot/root to 2.5" disks.
Kinda spense tho.
It would allow me to have two 2-slot gpus in the future though... If I'm ever able to upgrade my desktop graphics I could have ***two*** rx580s in there.
Which would play great into my gaming server plans.

Why have one when you can have two, amirite?
