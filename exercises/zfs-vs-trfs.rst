=============
ZFS vs btrfs
=============

Exercise 1
===========

_Search about and compare ZFS with btrfs features. Examine and discuss advantages and disadvantages in different areas and argue which one is better._

ZFS
----

from http://www.freebsd.org/doc/en/books/handbook/filesystems-zfs.html:

    "The Z file system, originally developed by Sun™, is designed to use a pooled storage method in that space is only used as it is needed for data storage. It is also designed for maximum data integrity, supporting data snapshots, multiple copies, and data checksums. It uses a software data replication model, known as RAID-Z. RAID-Z provides redundancy similar to hardware RAID, but is designed to prevent data write corruption and to overcome some of the limitations of hardware RAID."

Btrfs
------

from https://btrfs.wiki.kernel.org/index.php/Main_Page

    "Btrfs is a new copy on write (CoW) filesystem for Linux aimed at implementing advanced features while focusing on fault tolerance, repair and easy administration. Jointly developed at Oracle, Red Hat, Fujitsu, Intel, SUSE, STRATO and many others, Btrfs is licensed under the GPL and open for contribution from anyone."

Comparision
------------

Reading official definitions we have ZFS is focused on fiability in storage and eficient storage (only uses what is needed) in the other hand, Btrfs said that is focused on copy on write, more fiability and all the companies that are developing this filesystem.

ZFS is focused more technically on its advantages in prevent data corruption and redundacy with its RAID-Z. Advantages and easy configuration for system administration  and pool 'automatic' management (http://docs.oracle.com/cd/E19253-01/819-5461/zfsover-2/index.htm).

Btrfs is an ambicious work to port ZFS desing into Linux systems, because of scalability and agility demand on these days with data storage. Implement some of key features of ZFS and it's included in Linux Kernel 3.6.0 as an extra option to test.

Due to licensing problems ZFS (Sun License) couldn't be included into Linux Kernel so this need created the way to port this filesystem into Linux machines.

Nowadays Btrfs is preparing to be in production environments, niche of ZFS. In production environments ZFS has got enought market as they could have because their file system is available on Solaris, BSD and derivated distributions. Their market is lesser than Linux distributions that doesn't have ZFS. Because of that IMHO I think that all the efforts to implement a distributed file system as powerful as ZFS into Linux distributions is very important, but because is more expensive technically to change OS to have ZFS than implement a similiar file system into current distribution. But if you have a ZFS environment you don't have to be affraid because is more powerful that other implementations of btrfs.

References
-----------

# Comparison ZFS better than Btrfs: http://rudd-o.com/linux-and-free-software/ways-in-which-zfs-is-better-than-btrfs
# ZFS and Btrfs benchmarks: http://www.linux-mag.com/id/7308/
# Oracle ZFS: http://docs.oracle.com/cd/E19253-01/819-5461/zfsover-2/index.html

