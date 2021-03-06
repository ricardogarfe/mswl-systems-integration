## Pool

Crear el pool dentro de la consola de `virsh`:

```shell
virsh # pool-define-as --name ws01-install --type dir --target /var/lib/libvirt/final-raid/ws01-install/
virsh # pool-define-as --name db01-install --type dir --target /var/lib/libvirt/final-raid/db01-install/
virsh # pool-define-as --name ws01-www --type dir --target /var/lib/libvirt/final-raid/ws01-www/
virsh # pool-define-as --name db01-mysql --type dir --target /var/lib/libvirt/final-raid/db01-mysql/
```

Arrancar el pool:

```shell
virsh # pool-start ws01-install
virsh # pool-start db01-install
virsh # pool-start ws01-www
virsh # pool-start db01-mysql
```

## Volúmenes

Volúmenes, crear el volúmen vm01.img (vol-create siempre persistente) para la máquina virtual:

```shell
virsh # vol-create-as --pool ws01-install --name vm01.img --capacity 2200M --allocation 2200M --format raw
virsh # vol-create-as --pool db01-install --name vm02.img --capacity 2200M --allocation 2200M --format raw
virsh # vol-create-as --pool ws01-www --name vm01www.img --capacity 150M --allocation 150M --format raw
virsh # vol-create-as --pool db01-mysql --name vm02mysql.img --capacity 150M --allocation 150M --format raw
```

Mostrar la configuración xml del volúmen del pool:

* vm01:

```xml
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
```

* vm02:

```xml
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
```

## Redes

Entrar en la consola virsh:

```shell
$ sudo virsh
virsh # net-list --all
Nombre               Estado     Inicio automático
-----------------------------------------
default              activo     si        
```

Se ha de crear un xml para la configuración.

```xml
<network>
  <name>nat</name>
    <bridge name="virbr10" />
    <forward mode="nat"/>
    <ip address="192.168.122.1" netmask="255.255.255.0">
      <dhcp>
        <range start="192.168.122.100" end="192.168.122.254" />
      </dhcp>
    </ip>
</network>
```

Definir la red nat:

```shell
virsh # net-define /home/ricardo/projects/git/mswl/mswl-systems-integration/tools/kvm-config/net-nat.xml
virsh # net-start nat
virsh # net-destroy default
virsh # net-autostart --disable default
```

## Definir la Máquina virst-install

Funciona correctamente sin errores con el comando `virt-install`:

* raid-vm01:

```shell
$ virt-install --connect qemu:///system -n raid-vm01 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/final-raid/ws01-install/vm01.img -c /var/lib/libvirt/images/ubuntu-12.10-server-amd64.iso --vnc --noautoconsole --os-type linux --os-variant ubuntuprecise --accelerate -v --network network:nat --hvm --force
```

Instalar a través de vnc:

```shell
virt-viewer raid-vm01 &
```

* raid-vm02:

```shell
$ virt-install --connect qemu:///system -n raid-vm02 -r 1024 --vcpus=1 --disk path=/var/lib/libvirt/final-raid/db01-install/vm02.img -c /var/lib/libvirt/images/ubuntu-12.10-server-amd64.iso --vnc --noautoconsole --os-type linux --os-variant ubuntuprecise --accelerate -v --network network:nat --hvm --force
```

Instalar a través de vnc:

```shell
virt-viewer raid-vm02 &
```

### Discos duros

Como Dispositivo externo después de haber creado las imágenes.

* Parar las máquinas virtuales:

```shell
virsh # shutdown raid-vm01
virsh # shutdown raid-vm02
```

* Adjuntar el nuevo disco:

```shell
virsh # attach-disk raid-vm01 /var/lib/libvirt/final-raid/ws01-www/vm01www.img --target vdb --persistent
virsh # attach-disk raid-vm02 /var/lib/libvirt/final-raid/db01-mysql/vm02mysql.img --target vdb --persistent
```

* Levantar las máquinas virtuales:

```shell
virsh # start raid-vm01
virsh # start raid-vm02
```

* Ejecutar el comando **fdisk -l** para comprobar que se ha iniciado correctamente:

```shell
Disco /dev/vda: 2306 MB, 2306867200 bytes
16 cabezas, 63 sectores/pista, 4469 cilindros, 4505600 sectores en total
Unidades = sectores de 1 * 512 = 512 bytes
Tamaño de sector (lógico / físico): 512 bytes / 512 bytes
Tamaño E/S (mínimo/óptimo): 512 bytes / 512 bytes
Identificador del disco: 0x00005919

Dispositivo Inicio    Comienzo      Fin      Bloques  Id  Sistema
/dev/vda1   *        2048     3905535     1951744   83  Linux
/dev/vda2         3907582     4503551      297985    5  Extendida
/dev/vda5         3907584     4503551      297984   82  Linux swap / Solaris

Disco /dev/vdb: 157 MB, 157286400 bytes
16 cabezas, 63 sectores/pista, 304 cilindros, 307200 sectores en total
Unidades = sectores de 1 * 512 = 512 bytes
Tamaño de sector (lógico / físico): 512 bytes / 512 bytes
Tamaño E/S (mínimo/óptimo): 512 bytes / 512 bytes
Identificador del disco: 0x00000000

El disco /dev/vdb no contiene una tabla de particiones válida
```

* Dar formato al disco:

```shell
fdisk /dev/vdb    
$ configure new partition using 'n' option and 't' to define linux type '83'. And **w**.
$ mkfs.ext4 /dev/vdb1
```

* Montar el dispositivo:

```shell
$ mount /dev/vdb1 /var/www
```

* Definir en fstab para que se inicialice automáticamente mediante UUID con el comando **blkid**:

```shell
root@ubuntuvm01:/home/vm01# blkid
/dev/vda1: UUID="472e4de5-4e12-42e3-85b9-c815f5ceb8d0" TYPE="ext4" 
/dev/vda5: UUID="65144bfe-9651-48f7-8ebb-6d8e3894ef37" TYPE="swap" 
/dev/vdb1: UUID="62eb7abe-cd39-4150-8db2-4a12f7f6e74d" TYPE="ext4" 
```

* fstab:
```shell
UUID=472e4de5-4e12-42e3-85b9-c815f5ceb8d0 /               ext4    errors=remount-ro 0       1
# swap was on /dev/vda5 during installation
UUID=65144bfe-9651-48f7-8ebb-6d8e3894ef37 none            swap    sw              0       0
UUID=62eb7abe-cd39-4150-8db2-4a12f7f6e74d	/var/www	ext4	defaults	0	0
```

## User/Password

ubuntuvm01:

* vm01 vm01

ubuntuvm02:

* vm02 vm02


## Virsh dominios

Apagar:

```shell
virsh # shutdown apachevm01
El dominio apachevm01 está siendo apagado

virsh # shutdown db-vm02
El dominio db-vm02 está siendo apagado
```

