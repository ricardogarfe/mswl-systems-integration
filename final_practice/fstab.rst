FSTAB
======

# sudo blkid

/dev/sda1: SEC_TYPE="msdos" LABEL="DellUtility" UUID="3030-3030" TYPE="vfat" 
/dev/sda2: LABEL="RECOVERY" UUID="9C425B98425B764C" TYPE="ntfs" 
/dev/sda3: LABEL="OS" UUID="F8965E78965E36FA" TYPE="ntfs" 
/dev/sda5: LABEL="disco_datos" UUID="01CBBC054CF42C40" TYPE="ntfs" 
/dev/sda6: UUID="71141e97-bb11-4cc0-89ca-43e7d0363963" TYPE="ext4" 
/dev/sda7: UUID="d20b9cf1-533e-44be-bc2d-b028c0f40a5c" TYPE="swap" 
/dev/sdc: UUID="414cf565-4aa4-4148-a12a-c79b02ed332f" UUID_SUB="4b9f4dde-e7d6-cda2-5bef-5f8c916f2997" LABEL="ricardo-ubuntu:1" TYPE="linux_raid_member" 
/dev/md1: UUID="0a9cce04-6fda-4016-aca1-d6630f26a5be" TYPE="ext4" 
/dev/sde: UUID="414cf565-4aa4-4148-a12a-c79b02ed332f" UUID_SUB="27c1fd97-b85c-360f-dc6c-99587fd64c19" LABEL="ricardo-ubuntu:1" TYPE="linux_raid_member" 
/dev/md1p1: UUID="ZFR4sf-aC8Y-iqrN-c03M-zhDh-hdeJ-ZEKi5i" TYPE="LVM2_member" 
/dev/md1p2: UUID="AHWh8n-Zzur-wkXr-yUpw-tGOZ-aP2s-8AmuST" TYPE="LVM2_member" 
/dev/mapper/ws01-ws01--install: UUID="e398326a-fe81-4895-a511-79e4c8c7b7b8" TYPE="ext4" 
/dev/mapper/ws01-ws01--www: UUID="a231cc15-fce7-4bbb-8fff-cb8b39dbe0ee" TYPE="ext4" 
/dev/mapper/db01-db01--install: UUID="d882d847-0047-442b-8852-1dbacc9cba1c" TYPE="ext4" 
/dev/mapper/db01-db01--mysql: UUID="ba90e1b7-95ff-41e5-8f12-407b956ac5ac" TYPE="ext4" 


Recuperar después del reinicio
-------------------------------

# mdadm -R /dev/md1

# mdadm --misc --detail /dev/md1

mount /dev/ws01/ws01-install /var/lib/libvirt/final-raid/ws01-install/
mount /dev/ws01/ws01-www /var/lib/libvirt/final-raid/ws01-www/
mount /dev/db01/db01-install /var/lib/libvirt/final-raid/db01-install/
mount /dev/db01/db01-mysql /var/lib/libvirt/final-raid/db01-mysql/


blkid
------

/dev/sda1: SEC_TYPE="msdos" LABEL="DellUtility" UUID="3030-3030" TYPE="vfat" 
/dev/sda2: LABEL="RECOVERY" UUID="9C425B98425B764C" TYPE="ntfs" 
/dev/sda3: LABEL="OS" UUID="F8965E78965E36FA" TYPE="ntfs" 
/dev/sda5: LABEL="disco_datos" UUID="01CBBC054CF42C40" TYPE="ntfs" 
/dev/sda6: UUID="71141e97-bb11-4cc0-89ca-43e7d0363963" TYPE="ext4" 
/dev/sda7: UUID="d20b9cf1-533e-44be-bc2d-b028c0f40a5c" TYPE="swap" 
/dev/sdb1: UUID="f6dd5a56-7a51-44d4-934e-eea2690d17c4" SEC_TYPE="ext2" TYPE="ext3" 
/dev/md1: UUID="0a9cce04-6fda-4016-aca1-d6630f26a5be" TYPE="ext4" 
/dev/md1p1: UUID="ZFR4sf-aC8Y-iqrN-c03M-zhDh-hdeJ-ZEKi5i" TYPE="LVM2_member" 
/dev/md1p2: UUID="AHWh8n-Zzur-wkXr-yUpw-tGOZ-aP2s-8AmuST" TYPE="LVM2_member" 
/dev/mapper/db01-db01--install: UUID="d882d847-0047-442b-8852-1dbacc9cba1c" TYPE="ext4" 
/dev/mapper/db01-db01--mysql: UUID="ba90e1b7-95ff-41e5-8f12-407b956ac5ac" TYPE="ext4" 
/dev/mapper/ws01-ws01--install: UUID="e398326a-fe81-4895-a511-79e4c8c7b7b8" TYPE="ext4" 
/dev/mapper/ws01-ws01--www: UUID="a231cc15-fce7-4bbb-8fff-cb8b39dbe0ee" TYPE="ext4" 
/dev/sdc: UUID="414cf565-4aa4-4148-a12a-c79b02ed332f" UUID_SUB="77ee88ee-bf8d-fc5a-e07c-a9756c2a0d55" LABEL="ricardo-ubuntu:1" TYPE="linux_raid_member" 

