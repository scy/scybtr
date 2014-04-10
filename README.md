# scybtr - some of my tools for btrfs

This repository includes some utility scripts I use when dealing with [btrfs](https://btrfs.wiki.kernel.org/). They might or might not be useful for you.

## btrsnap

Automatically create read-only snapshots of `/` and remove them again after 24 hours. Provides a bit of a Time Machine like experience to protect you from short-term fuckups. For the long-term ones, you better have a _real_ backup.

The crontab entry I use:

    */5 * * * * (date; PATH="/sbin:$PATH" /root/btrsnap /snap) >> /var/log/btrsnap.log 2>&1

## btrvm

Create lightweight throw-away VMs using libvirt and btrfs subvolumes. Please note that this tool contains next to no error checking and can have sharp edges.

**Also note that using btrfs for storing VM images can be [a huge performance problem](https://bugzilla.redhat.com/show_bug.cgi?id=689127). I’ve stopped using btrfs on VM host machines for that reason.**

### Initial Setup

Create a basic VM with its storage on a subvolume. How you're doing this is up to you. In the end you need a VM that's for example called `BASE.debian-7.4` with storage in a (read-only?) subvolume `/virt/base/debian-7.4/BASE.debian-7.4.img`.

You could for example do it like in the following snippet. Note that this should be considered pseudocode, I did not test these commands!

    mkdir -p /virt/base
    btrfs subvolume create /virt/newtemplate
    # The following will set up a new Debian based on my preseed file.
    # You'll most likely want to create something different.
    virt-install --name=newtemplate --ram=1024 --vcpus=2 --disk=path=/virt/newtemplate/BASE.debian-7.4.img,format=raw,size=10,perms=rw,cache=none,bus=virtio --os-type=linux --os-variant=debianwheezy --network=network=default --nographics --location=http://ftp.de.debian.org/debian/dists/wheezy/main/installer-amd64/ --extra-args='auto=true priority=critical url=http://dl.dropboxusercontent.com/u/18203695/preseed.vm.cfg console=ttyS0,115200n8 TERM=linux'
    # After the installation is complete:
    virsh shutdown newtemplate
    btrfs subvolume snapshot -r /virt/newtemplate /virt/base/debian-7.4
    virt-clone -o newtemplate -n BASE.debian-7.4 -f /virt/base/debian-7.4/BASE.debian-7.4.img --preserve-data
    # Get rid of the writable template VM.
    virsh undefine newtemplate
    btrfs subvolume delete /virt/newtemplate

### Creating a New VM

Run `btrvm clone debian-7.4 whatever` to create a new (writable) subvolume at `/virt/whatever` and a new libvirt VM called `whatever` that is a clone of the `BASE.debian-7.4` VM. Its storage volume will be `/virt/whatever/whatever.img`, and it'll be based on a snapshot of the base VM. That means, it takes up no disk space initially! It'll only begin using disk space when you start making changes to the disk's contents by running the VM and doing stuff in it.

### Deleting a VM

Run `btrvm delete whatever` to kill and undefine the VM and then remove the subvolume `/virt/whatever`. The space that VM used will be freed again, of course.

## Author and License Information

Written by [Tim Weber](http://scy.name/), [WTFPL](http://www.wtfpl.net/about/) version 2.
