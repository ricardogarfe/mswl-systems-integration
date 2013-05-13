====
RAID
====

Virtualización
===============

Crear el pool dentro de la consola de `virsh`:

    virsh # pool-define-as --name ws01-install --type dir --target /var/lib/libvirt/final-raid/ws01-install/
    virsh # pool-define-as --name db01-install --type dir --target /var/lib/libvirt/final-raid/db01-install/

Arrancar el pool:

    virsh # pool-start ws01-install
    virsh # pool-start db01-install

Volúmenes, crear el volúmen vm01.img (vol-create siempre persistente) para la máquina virtual:

    virsh # vol-create-as --pool ws01-install --name vm01.img --capacity 2000M --allocation 2000M --format raw
    virsh # vol-create-as --pool db01-install --name vm02.img --capacity 2000M --allocation 2000M --format raw

Mostrar la configuración xml del volúmen del pool:

virsh # vol-dumpxml vm01.img --pool ws01-install
<volume>
  <name>vm01.img</name>
  <key>/var/lib/libvirt/final-raid/ws01-install/vm01.img</key>
  <source>
  </source>
  <capacity>2097152000</capacity>
  <allocation>2099204096</allocation>
  <target>
    <path>/var/lib/libvirt/final-raid/ws01-install/vm01.img</path>
    <format type='raw'/>
    <permissions>
      <mode>0600</mode>
      <owner>0</owner>
      <group>0</group>
    </permissions>
  </target>
</volume>

virsh # vol-dumpxml vm02.img --pool db01-install
<volume>
  <name>vm02.img</name>
  <key>/var/lib/libvirt/final-raid/db01-install/vm02.img</key>
  <source>
  </source>
  <capacity>2097152000</capacity>
  <allocation>2099204096</allocation>
  <target>
    <path>/var/lib/libvirt/final-raid/db01-install/vm02.img</path>
    <format type='raw'/>
    <permissions>
      <mode>0600</mode>
      <owner>0</owner>
      <group>0</group>
    </permissions>
  </target>
</volume>

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

Definir la Máquina virst-install
---------------------------------

FreeBSD-9.1-RELEASE-amd64-disc1.iso

Funciona correctamente sin errores con el comando:

    # virt-install --connect qemu:///system -n raid-vm01 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/final-raid/ws01-install/vm01.img -c /var/lib/libvirt/images/ubuntu-12.10-server-amd64.iso --vnc --noautoconsole --os-type linux --os-variant ubuntuprecise --accelerate -v --network network:nat --hvm --force

virt-viewer raid-vm01 &
ubuntuvm01

    # virt-install --connect qemu:///system -n raid-vm02 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/final-raid/db01-install/vm02.img -c /var/lib/libvirt/images/ubuntu-12.10-server-amd64.iso --vnc --noautoconsole --os-type linux --os-variant ubuntuprecise --accelerate -v --network network:nat --hvm --force

virt-viewer raid-vm02 &
ubuntuvm02

Discos duros
-------------

<disk type='file' device='disk'>
    <driver name='tap' type='aio'/>
    <source file='/var/lib/libvirt/images/Guest1.img'/>
    <target dev='xvda'/>
</disk>

A través de virsh y una configuración xml:
    
    virsh # attach-device vm01 disk.xml

Dispositivo externo:

    virsh # attach-disk vm01 /var/lib/libvirt/mydata/vm01-data.img --target vdb --config

Definir en fstab para que se inicialice automáticamente.

User/Password
--------------

ubuntuapache

    * apachevm01 apache

ubuntumysql:

    * mysqlvm02 mysql

OpenSuse
---------

    virt-install --connect qemu:///system -n vm01 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/mypool01/vm01.img -c /home/ricardo/tmp/openSUSE-12.3-NET-x86_64.iso --vnc --noautoconsole --os-type linux --os-variant opensuse12 --accelerate -v --network network:nat --hvm --force

vm02 - vm02
mysql - root - admin

Virsh dominios
----------------

Apagar:

    virsh # shutdown apachevm01
    El dominio apachevm01 está siendo apagado

    virsh # shutdown db-vm02
    El dominio db-vm02 está siendo apagado

