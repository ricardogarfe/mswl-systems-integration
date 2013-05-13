===========================
MSWL - Systems Integration
===========================

Virtualización
===============

Almacenamiento, redes y máquinas virtuales.

Almacenamiento por Red
=======================

NAS - Network-Attached Storage. Orientado a ficheros, por lo cual, lo que se exporta es el fichero o el sistema de ficheros completo. SAMBA o SMB y el protocolo CIFS.

SAN - Storage Area Network. Se exporta un dispositivo de bloques sin sistema de ficheros. Al importar el dispositivo de bloques podemos aplicar el sistema de ficheros que queramos. iSCSI, Fiber Channel, SCSI.

iSCSI
------

Almacenamiento pro red basado en IP.

* Clientes: initiators.
* Servidores: target.

No requiere infraestuctura o cableado especial como Fiber Channel pero no llega a su potencial.

ATA sobre Ethernet
-------------------

Barato, barato. 

NAS vs SAN
--------------

NAS - Almacenamiento y sistema de ficheros.
SAN - Almacenamiento y nostros definimos el sistema de ficheros.

Distintos protocolos pero no son excluyentes por lo que pueden combinarse, no es muy frecuente.

Ejemplo: SAN exporta ZFS para definir otro sistema de ficheros.


FreeNAS
--------

IXS

Replicación del almacenamiento
===============================

El método básico es el mirror de disco (similar a RAID-1). 

  * Replicación síncrona: sin pérdida ("zero data loss") mediante operaciones atómicas de escritura (ACK).
  * Replicación asíncrona: larga distancia, latencia alta. Incrementa el rendimiento, pero en caso de pérdida, no se garantiza que el almacenamiento remoto tenga copia actualizada de los datos.
  
Independiente al sistema de fichero ya que hace copia bloque a bloque.

    * DRBD: Distributed Replicated Block Device: mirror RAID-1 de discos sobre red local LAN.
    * HAST: Highly Available STorage. Específico para BSD. Activo-Pasivo con dos nodos igual que DRBD.
    * AVS: Sun StorageTek Available Suite. Replicación remota con Solaris.

Sistemas de ficheros distribuidos
==================================

No se puede montar al mismo tiempo la misa partición en ambos equipos mediante DRBD.
Para eso se utliza un File System tipo _cluster_, _distribuido_, _paralelo_. A diferencia de NFS que exporta el directorio, se exporta como un filesystem completo.

Desventajas:

* Rendimiento inferior.
* Sistemas más complejos de configurar y mantener.

GlusterFS
----------

Evita el cuello de botella por múltiple concurrencia (descomprimir un tar.gz). No se compromete el rendimiento.
Muy simple, apto para Cloud.


Swift
------

Componente de OpenStack (IaaS). Rackspace aportó este componente que ya estaba bastante maduro.
El objeto (este es el nombre utilizado) se distribuye en n zonas. Se escribe en distintas zonas.

Backups
========

**Poder volver a un estado anterior**


No estructurados: caseros cds, dvds, discos duros.
Incrementales: sucesivos que contienen cambios, delta.
Diferenciales: los cambios en ficheros desde el último backup.
Completo:
Protección continua de datos: el sistema registra cada uno de los cambios.

rsync
------

Sincronizar ficheros y directorios de una máquina a otra y puede ejecutarse con SSH.
Puede ejecutar todos los métodos menos el de protección contiua de datos.

Bacula
-------

Cliente/Servidor de nivel empresarial que gestiona backups, restauración y verificación de ficheros a través de una red.

    * http://www.bacula.org/es/


El Arte de la Virtualización
=============================

Multiplexación de un recurso físico. Popek y Goldberg:

    * Equivalencia/Fidelidad.
    * Seguridad y control de recursos. Software de virtualización.
    * Eficiencia/Rendimiento.

Conceptos Básicos
------------------

**Anfitrión (host)**: El sistema operativo que ejecuta el software de virtualización. El anfitrión controla el hardware real.
**Invitado (guest)**: Sistema operativo virtualizado. No deben interferir entre ellos ni con el anfitrión. Aislamiento.

Software:

    * Hipervisor.
    * Virtual Machine Manager WMM.

Instacia se conoce como Máquina Virtual. Los SO se ejecutan en la máquina virtual.

Los Virtual Machine Monitor (hipervisores) permiten aislamiento.

Hipervisores

    * Tipo 1 (nativo/bar-metal) independiente del SO, antiguos.
    * Tipo 2 (hosted) una capa de software que corre en sobre el sistema operativo anfitrión.

Emulación sobre la misma arquitectura.

KVM
====

Comprobar la virtualización para intel y amd:

    egrep '(vmx|svm)' /proc/cpuinfo

Si es 64 bits:

    egrep ' lm ' /proc/cpuinfo

Comprobar kernel 64 bits:

    uname -m

Install:

    sudo apt-get install kvm libvirt-bin bridge-utils virt-viewer

Usuarios:

    adduser `id -un` libvirtd
    adduser `id -un` kvm

Comprobar que está bien instalado, muestra la lista de máquinas virtuales:

    virsh -c qemu:///system list

libvirt
--------

help `nombre comando`

Gestión de: 

    * pools almacenamiento.
    * Volúmenes.
    * Redes.
    * Máquinas Virtuales (dominios).

Volumen -> Pool -> Red.

Pool para guardar discos duros de las imágenes. Directorio del host para guardar las imágenes de las VM.

Directorio de configuraciones de virtsh: `cd /var/lib/libvirt/`.

* pool-destroy = stop.
* pool-start = start
* pool-create = no persistente.
* pool-define = persistente en base de datos.
* pool-undefine = sacar de la base de datos pero no destruir.

Crear el pool dentro de la consola de `virsh`:

    virsh # pool-define-as --name mypool --type dir --target /var/lib/libvirt/mypool/
    Se ha definido el grupo mypool
    virsh # pool-define-as --name mypool01 --type dir --target /var/lib/libvirt/mypool01/
    Se ha definido el grupo mypool

Arrancar el pool:

    virsh # pool-start mypool
    virsh # pool-start mypool01

Volúmenes, crear el volúmen vm01.img (vol-create siempre persistente) para la máquina virtual:

    virsh # vol-create-as --pool mypool01 --name vm01.img --capacity 3G --allocation 3G --format raw

Mostrar la configuración xml del volúmen del pool:

    virsh # vol-dumpxml vm01.img --pool mypool
    <volume>
      <name>vm01.img</name>
      <key>/var/lib/libvirt/mypool/vm01.img</key>
      <source>
      </source>
      <capacity>5368709120</capacity>
      <allocation>5368713216</allocation>
      <target>
        <path>/var/lib/libvirt/mypool/vm01.img</path>
        <format type='raw'/>
        <permissions>
          <mode>0600</mode>
          <owner>0</owner>
          <group>0</group>
        </permissions>
      </target>
    </volume>

Disco duro `/dev/vda` (v por virtual).

Redes
------

Entrar en la consola virsh:

    $ sudo virsh
    virsh # net-list --all
    Nombre               Estado     Inicio automático
    -----------------------------------------
    default              activo     si        

Se ha de crear un xml para la configuración.

Definir la red nat:

    virsh # net-define /home/ricardo/projects/git/mswl/mswl-systems-integration/tools/kvm-config/net-nat.xml
    virsh # net-start nat
    virsh # net-destroy default
    virsh # net-autostart --disable default

Máquina virtual en XML 'vm01.xml':

    <domain type='kvm'>
      <name>vm01</name>
      <memory unit='MiB'>1024</memory>
      <vcpu>1</vcpu>
      <os>
        <type arch='x86_64'>hvm</type>
        <boot dev='cdrom'/>
        <boot dev='hd'/>
      </os>
      <features>
        <acpi/>
        <apic/>
      </features>
      <devices>
        <emulator>/usr/bin/kvm</emulator>
        <disk type='file' device='disk'>
          <source file='/var/lib/libvirt/mypool/vm01.img' />
          <target dev='vda' bus='virtio' />
        </disk>
        <disk type='file' device='cdrom'>
          <driver name='qemu' type='raw'/>
          <source file='/home/ricardo/tmp/ubuntu-12.10-server-amd64.iso'/>
          <target dev='hdc' bus='ide'/>
        </disk>
        <interface type='network'>
          <source network='nat' />
        </interface>
        <graphics type='vnc' port='-1' />
      </devices>
    </domain>

Definir la máquina:

    define /home/ricardo/projects/git/mswl/mswl-systems-integration/tools/kvm-config/vm01.xml

Comprobar que está definida:

    virsh # list --all
     Id Nombre               Estado
    ----------------------------------
      - vm01                 apagado

Iniciar la máquina virtual:

    virsh # start vm01
    Se ha iniciado el dominio vm01

Para recargar la definición:

    virsh # undefine vm01

Sacar el cd:

    virsh # change-media vm01 hdc --eject

Logs de libvirt en el directorio '/var/log/libvirt/qemu/'.

Discos duros
-------------

A través de virsh y una configuración xml:
    
    virsh # attach-device vm01 disk.xml

Dispositivo externo:

    virsh # attach-disk vm01 7var/lib/libvirt/mydata/vm01-data.img --target vdb --config

Definir en fstab para que se inicialice automáticamente.


Definir la Máquina virst-install
---------------------------------

Funciona correctamente sin errores con el comando:

    virt-install --connect qemu:///system -n vm01 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/mypool01/vm01.img -c /home/ricardo/tmp/ubuntu-12.10-server-amd64.iso --vnc --noautoconsole --os-type linux --os-variant ubuntuprecise --accelerate -v --network network:nat --hvm --force

OpenSuse:

    virt-install --connect qemu:///system -n vm01 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/mypool01/vm01.img -c /home/ricardo/tmp/openSUSE-12.3-NET-x86_64.iso --vnc --noautoconsole --os-type linux --os-variant opensuse12 --accelerate -v --network network:nat --hvm --force

Nueva prueba:

    virt-install --connect qemu:///system -n os01 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/images/os01/os01.img -c /var/lib/libvirt/images/FreeBSD-9.1-RELEASE-amd64-disc1.iso --vnc --noautoconsole --os-type unix --os-variant freebsd8 --accelerate -v --network network:network1 --hvm

