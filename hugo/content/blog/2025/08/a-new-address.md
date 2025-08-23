---
title: "Changing the ipv6 pools on a rke2 k8s cluster"
date: 2025-08-23T00:00:00-04:00
draft: false
tagline: "Easier (more painful) than it looks on the surface"
slug: a-new-address
---

#### *there's no place like ::1*

A short one today to get me back into it:

As part of my rework of my home network and various services, I've been moving more and more things into various k8s clusters. \
I started this process by re-using `fd16:feed:f00d::/64` as the pod network in a few different clusters. \
Now that I've decided to use `fd00::/8` for LAN as well, I decided to give these clusters different subnets - I might want to do weird mesh networking in the future

I started with the cluster that backs `auth.thegoos.cloud`, my Authentik instance. A quick few googles showed that the general process for this was to:

- Update `/etc/rancher/rke2/config.yaml` with the new subnets
- Update the CNI config with the new subnets (Cilium in this case)
- Purge and re-add nodes to the cluster one at a time

The first point was ezpz. Update a vars file and re-run some ansible. Working for a cloud provider is teaching me good habits and I like it.

The second point was easy enough too. Cilium in multi-pool mode has a CR for each pool - I simply edited the `default` one to suit.

This last point surfaced itself as kube-controller-manager failing to start, complaining that the node configured cidr didn't fall within the cluster cidr. \
I tried to just `kubectl edit node thenode` and change the cidr, but the k8s api rejected any changes to this field that I tried.

So. I just purged and re-added the nodes, juggling dns entires so `rke2-server.chi.auth.thegoos.cloud` always pointed at a live member.

All that remained to do was kick a few pods that had gone into crashloopbackoff state, and I was done.
