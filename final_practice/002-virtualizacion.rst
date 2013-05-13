Virtualización
===============

Crear el pool dentro de la consola de `virsh`:

    virsh # pool-define-as --name pool-ws01-install --type dir --target /var/lib/libvirt/final-raid/ws01-install/
    virsh # pool-define-as --name pool-db01-install --type dir --target /var/lib/libvirt/final-raid/db01-install/


Arrancar el pool:

    virsh # pool-start pool-ws01-install
    virsh # pool-start pool-db01-install

Volúmenes, crear el volúmen vm01.img (vol-create siempre persistente) para la máquina virtual:

    virsh # vol-create-as --pool ws01-install --name vm01.img --capacity 1800M --allocation 1400M --format raw
    virsh # vol-create-as --pool db01-install --name vm01.img --capacity 1800M --allocation 1400M --format raw

Mostrar la configuración xml del volúmen del pool:

    virsh # vol-dumpxml vm01.img --pool pool-ws01-install
    <volume>
      <name>vm01.img</name>
      <key>/var/lib/libvirt/final/ws01-install/vm01.img</key>
      <source>
      </source>
      <capacity>1468006400</capacity>
      <allocation>1468010496</allocation>
      <target>
        <path>/var/lib/libvirt/final/ws01-install/vm01.img</path>
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

    virsh # attach-disk vm01 /var/lib/libvirt/mydata/vm01-data.img --target vdb --config

Definir en fstab para que se inicialice automáticamente.

Definir la Máquina virst-install
---------------------------------

Funciona correctamente sin errores con el comando:

    # virt-install --connect qemu:///system -n apache-vm01 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/final/ws01-install/vm01.img -c /var/lib/libvirt/images/ubuntu-12.10-server-amd64.iso --vnc --noautoconsole --os-type linux --os-variant ubuntuprecise --accelerate -v --network network:nat --hvm --force

virt-viewer apache-vm01 &

    # virt-install --connect qemu:///system -n db-vm02 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/final/db01-install/vm01.img -c /var/lib/libvirt/images/ubuntu-12.10-server-amd64.iso --vnc --noautoconsole --os-type linux --os-variant ubuntuprecise --accelerate -v --network network:nat --hvm --force

virt-viewer db-vm02 &

User/Password
--------------

ubuntuvm01

    * vm01 vm01

ubuntuvm02:

    * vm02 vm02

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

