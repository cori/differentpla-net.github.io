---
title: "Formatting an empeg's Hard Disk Manually"
date: 2004-03-25T09:37:00.000Z
x-drupal-nid: 4
x-needs-review: 2004-03-25T09:37:00.000Z
---
The sequel to [Partitioning an empeg's Hard Disk Manually](http://www.differentpla.net/content/2004/03/partition-empeg).

So, if you've followed the instructions on the previous page, you've now got a freshly-partitioned empeg hard disk. It's time to format it.

## Formatting the music partition

The music partition is stored on /dev/hdc4\. You should format it with the following command:

<pre>empeg:/empeg/bin# **mke2fs -m 0 -s 1 -i 131072 /dev/hdc4**</pre>

This sets aside no reserved space for the superuser (that's a zero after the -m) and uses 128KB per inode. We use the default block size and tell it to use sparse superblocks, which will speed up `fsck`.
Aside: the builder formats it with `-m 0 -b 1024 -i 131072`. This is outdated. Do it the other way.

You should see output that looks a little like this:

<pre>mke2fs 1.19, 13-Jul-2000 for EXT2 FS 0.5b, 95/08/09
ext2fs_check_if_mount: No such file or directory while determining whether /dev/
hdc4 is mounted.
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
92160 inodes, 2926602 blocks
0 blocks (0.00%) reserved for the super user
First data block=0
90 block groups
32768 blocks per group, 32768 fragments per group
1024 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Writing inode tables: done
Writing superblocks and filesystem accounting information: done</pre>

## Initialising the swap partition

<pre>empeg:/empeg/bin# **mkswap /dev/hdc6**</pre>

## Initialising the dynamic data partition

<pre>empeg:/empeg/bin# **dd if=/dev/zero of=/dev/hdc3**
dd: /dev/hdc3: No space left on device
33265+0 records in
33264+0 records out</pre>

This writes as many zeros as possible to the dynamic data partition by reading them from /dev/zero (which is an infinite-sized source of zero bytes). It'll stop when it hits the end of the dynamic data partition.

## Creating directories on the music partition

The empeg player software stores its database files in /empeg/var, which is usually a symlink to /drive0/var. If this directory doesn't exist, emplode will fail with a file not found error (0xC0041002). Both disks also need a <tt>fids</tt> directory, otherwise emplode won't see the extra space. <tt>/empeg/fids0</tt> is a symlink to <tt>/drive0/fids</tt> and <tt>/empeg/fids1</tt> is a symlink to <tt>/drive1/fids</tt>.

So, once you've formatted the partition, mount it somewhere (if you're using the empeg, `/drive1` is a good place):

<pre># **mount -n /dev/hdc4 /drive1**</pre>

Then you can create the directories:

<pre># **mkdir /drive1/fids**
# **mkdir /drive1/var**</pre>

Note that these instructions assume that you're using your empeg to format a second disk (so you've installed it as the slave). If you've installed the disk as master, you'll need to change the above instructions so that "drive1" is "drive0".

## Installing a player image

The steps so far have prepared the disk only for holding music. If you're planning on using the disk as the primary disk in your empeg, you'll need to install a player image once you've fitted it.