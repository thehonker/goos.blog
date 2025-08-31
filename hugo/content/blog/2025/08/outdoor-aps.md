---
title: "Cisco 1572 series wifi APs"
date: 2025-08-31T00:00:00-00:00
draft: false
tagline: "such power"
slug: outdoor-aps
---

#### *protip to oems: make your ca last for 40 years*

I recently acquired four cisco 1572EAC wifi APs for uh.... like 40$ a radio.

I run a cisco 9800-CL vWLC at home for my wifi. The existing APs in my infra are a mix of 2802i and 3802i's, and on WLC version 17.16.01 they work just fine.
Adding new APs has literally been as simple as adding the s/n to the allow list and plugging the AP into a switchport.

Until I got these 1572's.

They refused to join with the message "AP Auth Failed".

Given that these were "new open box" i figured they had ancient firmware on them, and I was right.

tl;dr they were failing to join because of this issue:

[Field Notice: FN63942 Cisco Wireless Lightweight Access Points and WLAN Controllers Fail to Create CAPWAP Connections Due to Certificate Expiration](https://www.cisco.com/c/en/us/support/docs/field-notices/639/fn63942.html)

The link title really says it all. Certs were baked into the firmware at manufacturing time, and they've since expired.
If a radio was kept up to date over the years, it wouldn't be an issue. BUT. Mine weren't.

The official solution to this is to just ratchet back the clock on the WLC to sometime before December 2022.

Naturally, this doesn't work.

My next option is to tftp in [newer firmware](https://software.cisco.com/download/home/286283419/type/280775090/release/15.3.3-JPT3?i=!pp) that has newer certs. Having done many a switch in my day, I feel confident.

Out comes the serial cable and I reach for my knockoff Knipex to crack open the console port cover plug.

I plug 'er in and uh...... how do you update a cisco lwap from tftp again?? There's no copy command, no archive command, nothing???

A google turned up this: `debug capwap console cli` which enabled a lot of the cli commands I'm used to.

HOWEVER. Even though I could ping my tftp server (which was on the same L2 domain as the radio), I couldn't actually tftp files from it. Even after setting `ip tftp source-interface` and stuff. Weirdness.

I remembered that modern Cisco bootloaders have tftp support too. I decided to try that.

A few googles and some frustration later and I had this working set of commands:

```bash
# Interrupt boot at bootloader with BREAK signal (delete on macos using screen)

set IP_ADDR 192.168.20.250
set NETMASK 255.255.255.0
set DEFAULT_ROUTER 192.168.20.1

tftp_init
ether_init
flash_init

format flash:

tar -xtract tftp://192.168.20.254/c1570-k9w8-tar.153-3.JPT3.tar flash:

set BOOT flash:/c1570-k9w8-tar.153-3.JPT3/c1570-k9w8-tar.153-3.JPT3

unset IP_ADDR
unset NETMASK
unset DEFAULT_ROUTER

boot
```

This worked, and worked well. The APs came up and joined the controller.

I'd hate to think about what I'd do if I had hundreds of these things though.......
