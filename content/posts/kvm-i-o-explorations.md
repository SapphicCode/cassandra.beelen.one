+++
author = "Cassandra"
categories = ["Tech", "Virtualization"]
date = 2019-05-18T08:46:36Z
description = ""
draft = false
slug = "kvm-i-o-explorations"
tags = ["Tech", "Virtualization"]
title = "KVM I/O explorations"

+++

This post is primarily going to be a resource about what I learned about KVM/QEMU I/O performance.

<!--more-->

## TL;DR:

In a nutshell; the defaults are terrible options to use. `writethrough` is the default QEMU caching option and it makes everything painfully slow. Additionally, using a non-metadata-preallocated `qcow2` will also slow writes down immensely.

## Caching modes and read performance

The default, mentioned nowhere in libvirt's UI, configs, or QEMU, is `writethrough`. This has a huge impact on read I/O, as mentioned in the fine print of [Proxmox's PVE Wiki](https://pve.proxmox.com/wiki/Performance_Tweaks) and some other obscure places on the internet.

This also seems to be impacted mostly if you're using `qcow2`, though I have no hard data on this at this time (only experience).

I'm going to recommend switching your caching to `none` or `writeback`, especially if you use `qcow2`-backed disk images.

## Write performance and image formats

As summarized in my first paragraph, you're going to want to be using either raw sparse files, LVs, or metadata-preallocated `qcow2` images.

You may be asking yourself what a metadata-preallocated `qcow2` image is. In a nutshell, it's the `-o preallocation=metadata` flag when passed to `qemu-img`'s `create` or `convert` subcommands.

I still **would not** recommend the use of `qcow2` files, unless you absolutely need snapshotting features and don't have a LVM volume group spare. My reasoning goes that, when I converted a 66 GiB Windows Server to a sparse raw disk, the raw ended up taking up only 50 GiB. 16 GiB in metadata seems a bit much for snapshotting in my book.

I **would** recommend using either sparse raw disk images, or LVM logical volumes. With LVM you even get snapshots back.

## Thin provisioning and dynamic sizing

This chapter isn't much about performance and mainly about thin raw disk's sizes and how to keep them down.

The `virtio` block driver does not let you set (or pass through to the VM) `discard=unmap`. Thus, I would recommend using the `virtio-scsi` SCSI controller instead, which does let you pass disk images through as `discard=unmap`. At that point, any VM discard commands will free up space on the host.

## Final words

I hope this post can be of some use to someone, and save them of the despair of using `qcow2`-backed storage in `writethrough` mode, and wondering if their hard drives / fancy new RAID / hypervisor is dying.

Additionally I would like to ask the developers of QEMU and Libvirt to recognize these immense performance hurdles and indicate them as such, before someone makes the experiences I've made.
