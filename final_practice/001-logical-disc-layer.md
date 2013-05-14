## Volúmenes Lógicos

Este proceso se ha de hacer en modo `sudo`.

## Configurar disco

Iniciar **fdisk** en el nuevo dispositivo USB:

```shell
$ fdisk /dev/sdc
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
```

Format the new partition with the `ext4` file system.

```shell
$ mkfs.ext4 /dev/sdc1
```

Crear un directorio como punto de montaje para el disco:

```shell
$ mkdir /myfiles
$ mount /dev/sdc1 /myfiles
```

The guest now has an additional virtualized file-based storage device. Note however, that this storage will not mount persistently across reboot unless defined in the guest's `/etc/fstab` file:

```shell
/dev/vdb1    /myfiles    ext3     defaults    0 0
```

## RAID-1

Un dispositivo virtual, raid, devices y dirección:

```shell
$ mdadm --create /dev/md1 --level=raid1 --raid-devices=2 /dev/sdc /dev/sde
```

Resultado por consola:

```shell
mdadm: /dev/sdc appears to contain an ext2fs file system
    size=7831552K  mtime=Mon May 13 15:56:40 2013
mdadm: /dev/sdc appears to be part of a raid array:
    level=raid0 devices=0 ctime=Thu Jan  1 01:00:00 1970
mdadm: partition table exists on /dev/sdc but will be lost or
       meaningless after creating array
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: /dev/sde appears to contain an ext2fs file system
    size=8099840K  mtime=Thu Jan  1 01:00:00 1970
mdadm: /dev/sde appears to be part of a raid array:
    level=raid0 devices=0 ctime=Thu Jan  1 01:00:00 1970
mdadm: partition table exists on /dev/sde but will be lost or
       meaningless after creating array
mdadm: largest drive (/dev/sde) exceeds size (7827392K) by more than 1%
Continue creating array? yes
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

A partir de ahora se ha de operar sobre **md1**. El tamaño se obtiene del disco más pequeño.

Comprobar que está activo:

```shell
# cat /proc/mdstat 
Personalities : [raid1] 
md0 : active raid1 sdc[1] sdb[0]
      1789120 blocks super 1.2 [2/2] [UU]
      [===============>.....]  resync = 78.5% (1405952/1789120) finish=1.7min speed=3586K/sec      
unused devices: <none>
```

Más detalles del dispositivo virtualizado 'mdadm --detail /dev/md1':

```shell
# mdadm --detail /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Mon May 13 10:56:20 2013
     Raid Level : raid1
     Array Size : 7827392 (7.46 GiB 8.02 GB)
  Used Dev Size : 7827392 (7.46 GiB 8.02 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

    Update Time : Mon May 13 11:27:52 2013
          State : active 
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

           Name : ricardo-ubuntu:0  (local to host ricardo-ubuntu)
           UUID : 64c4c398:5049ec1c:44cdd2a5:ea380e6f
         Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       32        0      active sync   /dev/sdc
       1       8       48        1      active sync   /dev/sdd
```

Formatear el md0 con make file system:

```shell
# mkfs.ext4 /dev/md1
```

Montar el disco RAID:

```shell
# mount /dev/md1 /mnt/raidmd1/
```

### Pruebas

Quitar un USB y ejecutar 'mdadm --detail /dev/md1':

```shell
Number   Major   Minor   RaidDevice State
   0       8       16        0      active sync   /dev/sdb
   1       0        0        1      removed
```

Pinchar el USB y reconstruir:

* comprobar el disco: **ls /dev/sd**
* Añadir al md1: **mdadm /dev/md1 --add /dev/sdc1**.

### Montar/Desmontar array

Desmontar el RAID:

* umount /mnt/raidmd1/
* mdadm --stop /dev/md1
* ó: mdadm -S /dev/md1

Montar el RAID:

```shell
/mnt# mdadm --examine /dev/sdd
/dev/sdd:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 64c4c398:5049ec1c:44cdd2a5:ea380e6f
           Name : ricardo-ubuntu:0  (local to host ricardo-ubuntu)
  Creation Time : Mon May 13 10:56:20 2013
     Raid Level : raid1
   Raid Devices : 2

 Avail Dev Size : 16191488 (7.72 GiB 8.29 GB)
     Array Size : 7827392 (7.46 GiB 8.02 GB)
  Used Dev Size : 15654784 (7.46 GiB 8.02 GB)
    Data Offset : 8192 sectors
   Super Offset : 8 sectors
          State : clean
    Device UUID : be6a0102:ead12657:2b968add:26e817db

    Update Time : Mon May 13 11:41:43 2013
       Checksum : 57d33396 - correct
         Events : 19


   Device Role : Active device 1
   Array State : AA ('A' == active, '.' == missing)
```

Assemble:

```shell
$ mdadm /dev/md1 --assemble -u 414cf565-4aa4-4148-a12a-c79b02ed332f
$ mdadm: /dev/md1 has been started with 2 drives.
```

### Escribir en la tabla

Escribir en en fichero de mdadm:

```shell
$ mdadm --detail --scan >> /etc/mdadm/mdadm.conf 
```

## Crear Volúmenes Lógicos

### Formatear usb

```shell
$ mkfs.ext4 /dev/sdc1
```

### Volumen físico

Dos volúmenes físicos.

```shell
$ pvcreate /dev/md1p1
$ pvcreate /dev/md1p2
$ pvdisplay
```

### Grupos de volúmenes

```shell
$ vgcreate ws01 /dev/md1p1
$ vgcreate db01 /dev/md1p2
$ vgdisplay 
```

### Volúmenes Lógicos

```shell
$ lvcreate -L2500M -n ws01-install ws01
$ lvcreate -L200M -n ws01-www ws01
$ lvcreate -L2500M -n db01-install db01
$ lvcreate -L200M -n db01-mysql db01
```

### Sistema de archivos

```shell
$ mkfs.ext4 /dev/ws01/ws01-install
$ mkfs.ext4 /dev/ws01/ws01-www
$ mkfs.ext4 /dev/db01/db01-install
$ mkfs.ext4 /dev/db01/db01-mysql
$ pvdisplay
```

## Montar

```shell
$ mkdir -p /var/lib/libvirt/final-raid/ws01-install/
$ mount /dev/ws01/ws01-install /var/lib/libvirt/final-raid/ws01-install/
$ mkdir -p /var/lib/libvirt/final-raid/ws01-www/
$ mount /dev/ws01/ws01-www /var/lib/libvirt/final-raid/ws01-www/

$ mkdir -p /var/lib/libvirt/final-raid/db01-install/
$ mount /dev/db01/db01-install /var/lib/libvirt/final-raid/db01-install/
$ mkdir -p /var/lib/libvirt/final-raid/db01-mysql/
$ mount /dev/db01/db01-mysql /var/lib/libvirt/final-raid/db01-mysql/
```

## Resize

```shell
$ lvextend -L+500M /dev/vg001/ws01-install 
  Extending logical volume ws01-install to 2,25 GiB
  Logical volume ws01-install successfully resized

$ resize2fs /dev/vg001/ws01-install
resize2fs 1.42 (29-Nov-2011)
Por favor ejecute antes 'e2fsck -f /dev/vg001/ws01-install'.

$ e2fsck -f /dev/vg001/ws01-install
e2fsck 1.42 (29-Nov-2011)
Paso 1: Verificando nodos-i, bloques y tamaños
Paso 2: Verificando la estructura de directorios
Paso 3: Revisando la conectividad de directorios
Paso 4: Revisando las cuentas de referencia
Paso 5: Revisando el resumen de información de grupos
/dev/vg001/ws01-install: 11/115200 files (0.0% non-contiguous), 16121/460800 blocks

$ resize2fs /dev/vg001/ws01-install
resize2fs 1.42 (29-Nov-2011)
Resizing the filesystem on /dev/vg001/ws01-install to 588800 (4k) blocks.
The filesystem on /dev/vg001/ws01-install is now 588800 blocks long.
```

