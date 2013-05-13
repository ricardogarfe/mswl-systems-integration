===========================
MSWL - Systems Integration
===========================

StaaS - Storage as a Service
=============================

Clusters, virtualización y cloud computing.
Virtualización del almacenamiento.

RAID
-----

Redundant Array of Independent Disks.

Para distribuir o replicar datos a través de esos discos en una estructura local.

    * Evita pérdida de datos.
    * Minimiza tiempos de caída.
    * Puede incrementar el rendimiento o la redundancia.
    * Vía hardware o vía software.

Strinpping de datos.
Duplicar datos en mirrors.

RAID 0
-------

Necesita dos o más discos de igual tamaño sin paridad ni mirroring. Haciendo la vez de un solo disco.

    * Tolerancia a fallos 0.

RAID 1
-------

Redundancia: Cada bloque se escribe en ambos discos.

RAID 4
-------

Al menos 3 discos. Un disco entero a información de paridad.

RAID 5
-------

Rendimiento y Redundancia. Modelo stripping y paridad. Mínimo 3 discos. Puede fallar un disco.

Ejercicio
==========

Un dispositivo virtual, raid, devices y dirección:

    mdadm --create /dev/md0 --level=raid1 --raid-devices=2 /dev/sdc/ /dev/sdd/

Resultado por consola:

    mdadm: /dev/sdb appears to be part of a raid array:
        level=raid0 devices=0 ctime=Thu Jan  1 01:00:00 1970
    mdadm: partition table exists on /dev/sdb but will be lost or
           meaningless after creating array
    mdadm: Note: this array has metadata at the start and
        may not be suitable as a boot device.  If you plan to
        store '/boot' on this device please ensure that
        your boot-loader understands md/v1.x metadata, or use
        --metadata=0.90
    mdadm: /dev/sdc appears to be part of a raid array:
        level=raid0 devices=0 ctime=Thu Jan  1 01:00:00 1970
    mdadm: partition table exists on /dev/sdc but will be lost or
           meaningless after creating array
    mdadm: largest drive (/dev/sdb) exceeds size (1789120K) by more than 1%
    Continue creating array? yes
    mdadm: Defaulting to version 1.2 metadata
    mdadm: array /dev/md0 started.

A partir de ahora se ha de operar sobre **md0**. El tamaño se obtiene del disco más pequeño.

Comprobar que está activo:

    # cat /proc/mdstat 
    Personalities : [raid1] 
    md0 : active raid1 sdc[1] sdb[0]
          1789120 blocks super 1.2 [2/2] [UU]
          [===============>.....]  resync = 78.5% (1405952/1789120) finish=1.7min speed=3586K/sec      
    unused devices: <none>

Más detalles del dispositivo virtualizado 'mdadm --detail /dev/md0':

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc

Formatear el md0 con make file system:

    # mkfs.ext4 /dev/md0

Montar el disco RAID:

    # mount /dev/md0 /mnt/

Pruebas
========

Quitar un USB y ejecutar 'mdadm --detail /dev/md0':

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       0        0        1      removed

Pinchar el USB y reconstruir:

    * comprobar el disco: **ls /dev/sd**
    * Añadir al md0: **mdadm /dev/md0 --add /dev/sdc1**.

Volúmenes Lógicos - LVM
========================

Gestión de volúmenes lógicos. Un esquema más flexible asignando el espacio en disco dinámicamente sin interrupción de sistema.

Una de las formas de Virtualizar el almacenamiento.

Desmontar el RAID:

    * umount /mnt
    * mdadm --stop /dev/md0

Instalar lvm2 **apt-get install lvm2**.
* Comprobar con `pvdisplay`.
* `vgdisplay` Comprueba LVM
* `lvdisplay` lógicos.

Crear espacio físico con `pvcreate /dev/sdc` y comprobar con `pvdisplay`:

    # pvdisplay 
    "/dev/sdc1" is a new physical volume of "1,71 GiB"
    --- NEW Physical volume ---
    PV Name               /dev/sdc1
    VG Name               
    PV Size               1,71 GiB
    Allocatable           NO
    PE Size               0   
    Total PE              0
    Free PE               0
    Allocated PE          0
    PV UUID               E4yOma-uf4y-hYw3-y8iS-E5og-85ju-FPlatp

Crear un grupo de volúmenes con la suma de ambos:

    * vgcreate vg00 /dev/sdc /dev/sdb

  Volume group "vg00" successfully created

Crear el volúmen lógico y asignar el tamaño:

    * lvcreate -L500M -n test vg00

  Logical volume "test" created

Un nuevo volúmen lógico:

    * lvcreate -L1G -n test2 vg00

Dar formato al volúmen lógico:

    * mkfs.ext3 /dev/vg00/test
    * mkfs.ext3 /dev/vg00/test2

Montar los volúmenes `mount /dev/vg00/test /mnt/`.

Aumentar tamaño lógico:

    * lvresize -L+500M /dev/vg00/test

Hay que dar formato para obtener más espacio con el formato ext3. Desmontar el sistema de ficheros.

    * resize2fs /dev/vg00/test

Salida por pantalla:

    resize2fs 1.42 (29-Nov-2011)
    Por favor ejecute antes 'e2fsck -f /dev/vg00/test'.

Por lo que hay que ejecutar el comando y volver a ejecutar resize.

    The filesystem on /dev/vg00/test is now 614400 blocks long.

XFS es un sistema de ficheros que puede crecer en caliente sin tener que desmontar el dispositivo.

Montar el dispositivo y comprobar con df -h

    S.ficheros            Tamaño Usados  Disp Uso% Montado en
    /dev/sda6                94G    76G   14G  85% /
    udev                    1,9G   4,0K  1,9G   1% /dev
    tmpfs                   778M   1,1M  777M   1% /run
    none                    5,0M   8,0K  5,0M   1% /run/lock
    none                    1,9G   3,6M  1,9G   1% /run/shm
    cgroup                  1,9G      0  1,9G   0% /sys/fs/cgroup
    /dev/mapper/vg00-test   582M    11M  541M   2% /mnt

Sistemas de Ficheros
=====================

Enfoque Unix: todo es un fichero.

Journaling
-----------

Incrementar la fiabilidad frente apagones mediante un seguimiento de los cambios para evitar las pérdidas de datos. Ext3 en adelante.

B-tree filesystem y ZFS, ambas pertencen a ORACLE después de la compra de SUN.

SmartOS
========

Ejecutar comando para copiar la iso en el USB: `dd if=smartOs.img of=/dev/sdd bs=1024`

http://smartos.org/

SmartOS unites four extraordinary technologies to revolutionize the datacenter:

    * ZFS + DTrace + Zones + KVM

Todas las mejores tecnologías de virtualización y ZFS.

ZFS
----

Sistema de ficheros potente y escalable 128bit. Moderno y escrito desde cero. Z (z-bite) File System.

Fiable utilizado por JoyEnt (smartOS) y ORACLE (Solaris privativo).

Está diseñado pensando en el almacenamiento. Integrar RAID, Volúmenes y Sistema de ficheros. Conectar y usar como al ampliar la memoria RAM.

    * Pool de almacenamiento: elimina por completo el viejo concepto de volúmen lógico como capa aparte.
    * Siempre va a ser consistente.
    * Detecta y corrige silenciosamente los errores
    * Crear SNAPSHOTS, cloenes, rollbacks, deduplicación, compresión, replicación, cifrado, compartición nativa vía nfs, cifs o iscsi...
    * Deduplicación: punteros entre contenidos a nivel de bloque (10101) para no repetir bloques, es decir enlaza con punteros para no repetir bloques de esta manera ahorra el espacio del disco.

Pools de Almacenamiento: **zpools** formados por **vdevs** mediante **zfs**.

RAID-Z : Se asemeja a RAID-5, la paridad se distribuye por los discos.














