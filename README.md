# scybtr - some of my tools for btrfs

This repository includes some utility scripts I use when dealing with [btrfs](https://btrfs.wiki.kernel.org/). They might or might not be useful for you.

## btrsnap

Automatically create read-only snapshots of `/` and remove them again after 24 hours. Provides a bit of a Time Machine like experience to protect you from short-term fuckups. For the long-term ones, you better have a _real_ backup.

The crontab entry I use:

    */5 * * * * (date; PATH="/sbin:$PATH" /root/btrsnap /snap) >> /var/log/btrsnap.log 2>&1

## Author and License Information

Written by [Tim Weber](http://scy.name/), [WTFPL](http://www.wtfpl.net/about/) version 2.
