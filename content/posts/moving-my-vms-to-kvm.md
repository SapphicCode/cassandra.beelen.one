+++
author = "Cassandra"
categories = ["Tech", "Virtualization"]
date = 2019-01-24T20:51:48Z
description = ""
draft = false
slug = "moving-my-vms-to-kvm"
tags = ["Tech", "Virtualization"]
title = "Moving my VMs to KVM"

+++

I've painstakingly started offloading my VMs from my ESXi host to my new libvirt+KVM host.

<!--more-->

The new host is essentially my desktop, but is mostly compatible (as the motherboards are identical, the only differences are the CPU model (FX-6300 compared to an FX-8350) and a RAM downsize from 32GB to 8GB though the swap space I've allocated should make up for that). Once I'm done migrating everything I plan to just swap disks.

## Migration plan

I prepared the new host with a 50GB LVM SSD cache backed by a 1TB HDD, both of which are encrypted by LUKS2. The rest of the SSD PV was put to use housing the root filesystem, as well as swap.

I plan to shut down every VM, migrate the disks over with `qemu-img`, start the aforementioned VM on the new host and fix strange behavior.

(More to the point, the command I used was `qemu-img convert $file -o preallocation=metadata -O qcow2 $name.qcow2`, as it converted the `*-Flat.vmdk` files to preallocated qcow2 files.)

### Other goals

- Less reliance on (Windows) DHCP and respective (Active Directory) DNS, through ZeroTier static IP management and internet-facing DNS

## Observations

- libvirt/qcow2 on GlusterFS: Don't do that.(It corrupted one of my early imports, thankfully I hadn't deleted it from the old host yet, and it was only a personal-use Windows guest.)
- Most OSes will only boot with SATA until the necessary VirtIO drivers are in place. (`virtio-win` SCSI for Windows, usually just an initram rebuild on Linux, though Debian was a notable exception and worked perfectly out of the box)
- Linux guests will almost certainly need to be reconfigured unless they have manually-configured IP addresses and static interface names.
- Windows guests will also recognize the network adapter change and reset all static IP settings, though this shouldn't come as a surprise it's good to keep in mind.

## Closing remarks

This post is still very much a work-in-progress as things develop, however I can say this: I'm running VMs that have seen Hyper-V, then ESXi, and now QEMU/KVM, and they're not fun to migrate, pick carefully. Thankfully I only have a dozen or so.

Best I can say is, pick the hypervisor that is right for you. I expected Hyper-V to have good performance monitoring tools, it had nothing of the sort and I only noticed it after I had migrated. I expected ESXi to run stable on commodity hardware and here I am, two or three purple screens later.

libvirt won't be for everyone, but I can say that it's worked out very smoothly for me thus far and provides ample expansion opportunity should I feel the need, and that much I appreciate.
