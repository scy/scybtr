# scybtr - some of my tools for btrfs

This repository includes some utility scripts I use when dealing with [btrfs](https://btrfs.wiki.kernel.org/). They might or might not be useful for you.

## btrsnap

Automatically create read-only snapshots of `/` and remove them again after 24 hours. Provides a bit of a Time Machine like experience to protect you from short-term fuckups. For the long-term ones, you better have a _real_ backup.

The crontab entry I use:

    */5 * * * * (date; PATH="/sbin:$PATH" /root/btrsnap /snap) >> /var/log/btrsnap.log 2>&1

## btrvm

Create lightweight throw-away VMs using libvirt and btrfs subvolumes. Please note that this tool contains next to no error checking and can have sharp edges.

### Initial Setup

Create a basic VM with its storage on a subvolume. How you're doing this is up to you. In the end you need a VM that's for example called `BASE.debian-7.4` with storage in a (read-only?) subvolume `/virt/base/debian-7.4/BASE.debian-7.4.img`.

(I could maybe write a bit more here about how to do that.)

### Creating a New VM

Run `btrvm clone debian-7.4 whatever` to create a new (writable) subvolume at `/virt/whatever` and a new libvirt VM called `whatever` that is a clone of the `BASE.debian-7.4` VM. Its storage volume will be `/virt/whatever/whatever.img`, and it'll be based on a snapshot of the base VM. That means, it takes up no disk space initially! It'll only begin using disk space when you start making changes to the disk's contents by running the VM and doing stuff in it.

### Deleting a VM

Run `btrvm delete whatever` to kill and undefine the VM and then remove the subvolume `/virt/whatever`. The space that VM used will be freed again, of course.

## Author and License Information

Written by [Tim Weber](http://scy.name/), [WTFPL](http://www.wtfpl.net/about/) version 2.
