---
title: "Cisco T-Rex - Benchmark Selection"
date: 2022-08-11T00:00:00-04:00
draft: true
tagline: "python goes brrr"
slug: trex-benchmark-selection
---

#### *time to make the donuts*

Now that I kinda know our way around trex and have an automated way of running repeatable tests... \
We need to find out what tests we want to run. \
We also need to properly benchmark our test equipment. Can't test things if you don't have a "calibrated" test setup.

![r5s openwrt emix2 profile graph](/2022/08/nanopi-r5s/img/r5s-owrt-nat-full-yes_steering-emix2-mult_2-dur_1200-2022-08-11T15-43-25.907Z-bits.png)
> Preview of our tests of the r5s

## graph setup

First let's finalize our graph format, colors, etc
Right now I'm graphing these things: (all of these are total/global)
```
- tx_bps
- rx_bps
- rx_drop_bps
- tx_pps
- rx_pps
- (we don't get a rx_drop_pps datapoint)
- tx_cps
- active_flows
```

## test selection

We need a good mix of tests. \
Here's a list, maybe.

### test setups

```
- typical lan -> wan nat (home router setup)
- lan -> lan briding (using each lan port for the same l2 domain)
- lan -> lan routing (using each lan port for a different l2 domain)
```

### test profiles
