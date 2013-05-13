https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Administration_Guide/sect-Virtualization-Virtualized_block_devices-Adding_storage_devices_to_guests.html#sect-Virtualization-Adding_storage_devices_to_guests-Adding_file_based_storage_to_a_guest

12.3. Adding storage devices to guests
This section covers adding storage devices to a guest. Additional storage can only be added as needed.
12.3.1. Adding file based storage to a guest
File-based storage is a collection of files that are stored on the hosts file system that act as virtualized hard drives for guests. To add file-based storage, perform the following steps:
Procedure 12.1. Adding file-based storage

Create a storage file or use an existing file (such as an ISO file). Note that both of the following commands create a 4GB file which can be used as additional storage for a guest:
Pre-allocated files are recommended for file-based storage images. Create a pre-allocated file using the following dd command as shown:
# dd if=/dev/zero of=/var/lib/libvirt/images/FileName.iso bs=1M count=4096
Alternatively, create a sparse file instead of a pre-allocated file. Sparse files are created much faster and can be used for testing, but are not recommended for production environments due to data integrity and performance issues.
# dd if=/dev/zero of=/var/lib/libvirt/images/FileName.iso bs=1M seek=4096 count=0
Create the additional storage by writing a <disk> element in a new file. In this example, this file will be known as NewStorage.xml.
A <disk> element describes the source of the disk, and a device name for the virtual block device. The device name should be unique across all devices in the guest, and identifies the bus on which the guest will find the virtual block device. The following example defines a virtio block device whose source is a file-based storage container named FileName.img:
<disk type='file' device='disk'>
   <driver name='qemu' type='raw' cache='none'/>
   <source file='/var/lib/libvirt/images/FileName.img'/>
   <target dev='vdb'/>
</disk>
Device names can also start with "hd" or "sd", identifying respectively an IDE and a SCSI disk. The configuration file can also contain an <address> sub-element that specifies the position on the bus for the new device. In the case of virtio block devices, this should be a PCI address. Omitting the <address> sub-element lets libvirt locate and assign the next available PCI slot.
Attach the CD-ROM as follows:
<disk type='file' device='cdrom'>
   <driver name='qemu' type='raw' cache='none'/>
   <source file='/var/lib/libvirt/images/FileName.iso'/>
   <readonly/>
   <target dev='hdc'/>
</disk >
Add the device defined in NewStorage.xml with your guest (Guest1):
# virsh attach-device --config Guest1 ~/NewStorage.xml
Note
This change will only apply after the guest has been destroyed and restarted. In addition, persistent devices can only be added to a persistent domain, that is a domain whose configuration has been saved with  virsh define  command.
If the guest is running, and you want the new device to be added temporarily until the guest is destroyed, omit the --config option:
# virsh attach-device Guest1 ~/NewStorage.xml
Note
The virsh command allows for an attach-disk command that can set a limited number of parameters with a simpler syntax and without the need to create an XML file. The attach-disk command is used in a similar manner to the attach-device command mentioned previously, as shown:
# virsh attach-disk Guest1 /var/lib/libvirt/images/FileName.iso vdb --cache none
Note that the virsh attach-disk command also accepts the --config option.
Start the guest machine (if it is currently not running):
# virsh start Guest1
Note
The following steps are Linux guest specific. Other operating systems handle new storage devices in different ways. For other systems, refer to that operating system's documentation.
Partitioning the disk drive
The guest now has a hard disk device called /dev/vdb. If required, partition this disk drive and format the partitions. If you do not see the device that you added, then it indicates that there is an issue with the disk hotplug in your guest's operating system.
Start fdisk for the new device:
# fdisk /dev/vdb
Command (m for help):
Type n for a new partition.
The following appears:
Command action
e   extended
p   primary partition (1-4)
Type p for a primary partition.
Choose an available partition number. In this example, the first partition is chosen by entering 1.
Partition number (1-4): 1
Enter the default first cylinder by pressing Enter.
First cylinder (1-400, default 1):
Select the size of the partition. In this example the entire disk is allocated by pressing Enter.
Last cylinder or +size or +sizeM or +sizeK (2-400, default 400):
Enter t to configure the partition type.
Command (m for help): t
Select the partition you created in the previous steps. In this example, the partition number is 1 as there was only one partition created and fdisk automatically selected partition 1.
Partition number (1-4): 1
Enter 83 for a Linux partition.
Hex code (type L to list codes): 83
Enter w to write changes and quit.
Command (m for help): w
Format the new partition with the ext3 file system.
# mke2fs -j /dev/vdb1
Create a mount directory, and mount the disk on the guest. In this example, the directory is located in myfiles.
# mkdir /myfiles
# mount /dev/vdb1 /myfiles
The guest now has an additional virtualized file-based storage device. Note however, that this storage will not mount persistently across reboot unless defined in the guest's /etc/fstab file:
/dev/vdb1    /myfiles    ext3     defaults    0 0
