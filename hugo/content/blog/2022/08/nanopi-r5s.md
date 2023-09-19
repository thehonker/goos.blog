---
title: "NanoPi R5S"
date: 2022-08-01T00:00:00-04:00
draft: true
tagline: "packets, packets everywhere"
slug: nanopi-r5s
---

#### *adorable. just. adorable.*

A couple weeks ago I ordered both the [r4s](https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R4S) and [r5s](https://wiki.friendlyelec.com/wiki/index.php/NanoPi_R5S) from nanopi.
Today the r5s showed up:

***insert pic of r5s***

Right away I flashed the latest version that included docker.

```none
root@10.15.41.100's password:
 ___    _             _ _    __      __   _
| __| _(_)___ _ _  __| | |_  \ \    / / _| |_
| _| '_| / -_) ' \/ _` | | || \ \/\/ / '_|  _|
|_||_| |_\___|_||_\__,_|_|\_, |\_/\_/|_|  \__|
                          |__/
 -----------------------------------------------------
 FriendlyWrt 21.02.3, r16554-1d4dea6d4f
 -----------------------------------------------------
root@FriendlyWrt:~# uname -a
Linux FriendlyWrt 5.10.110 #1 SMP Fri Jul 29 16:37:47 CST 2022 aarch64 GNU/Linux
```
> I'm writing this line on Aug 1st.
> This build is **minty** fresh.

Now that the firmware is updated, let's run some benchmarks on it without any tweaking/tuning.

## List of benchmarks to run:

```none
// Typical home network traffic
lan <-> wan NAT

// As if you were using it as a 2-port switch
lan <-> lan bridged

// Two separate LANs with this as a firewall between them
lan <-> lan routed/firewalled

// LAN to the device itself - NAS, etc
lan/wan <-> r5s

// We'll run these later on (probably in a different post)
// Openvpn
wan (openvpn client) <-> lan

// Wireguard
wan (wireguard client) <-> lan
```

## Testing methodology:
- We'll run bidirectional tests with both iperf3 and trex.

For testing I have a Dell Optiplex 7040 with a 4-port i350 card.

## Cursory speedtest.net testing
speedtest-cli shows about 300/300, which is about what I get here (yay fios). \
So we know the r5s can NAT at least that much.

## Benchmark the first: LAN <-> WAN w/ NAT
