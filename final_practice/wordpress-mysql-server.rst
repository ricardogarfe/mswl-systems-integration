## IP configuration

Establecer ip estática en los **guests**, configurar DNS, añadir la línea **nameserver 192.168.122.1** con la ip del host en el fichero **/etc/resolvconf/resolv.conf.d/base**

```shell
sudo pico /etc/network/interfaces
```

### Apache raid-vm01

Fichero de configuración `/etc/network/interfaces`.

```shell
auto eth0
iface eth0 inet static
    address 192.168.122.80
    netmask 255.255.255.0
    broadcast 192.168.122.255
    gateway 192.168.122.1
```

Restar servicio de redes:

```shell
# sudo service networking restart
```

### raid-vm02 Mysql

Fichero de configuración `/etc/network/interfaces`.

```shell
auto eth0
iface eth0 inet static
    address 192.168.122.81
    netmask 255.255.255.0
    broadcast 192.168.122.255
    gateway 192.168.122.1
```
Restar servicio de redes:

```shell
# sudo service networking restart
```

## Wordpress

Accedemos vía **SSH**:

```shell
ssh vm01@192.168.122.80
```

1.- Instalamos apache2, mysql y php. 

```shell
sudo apt-get install apache2 php5 php5-mysql mysql-server phpmyadmin
```

2.- Descargamos la última versión de wordpress

```shell
cd /var/www
wget http://wordpress.org/latest.tar.gz
```

3.- Descomprimimos y descompactamos

```shell
tar -zxvf  latest.tar.gz
```

4.- Creamos la base de datos wordpress en mysql (primero se ha de instalar **Mysql** en **raid-vm02**)

5.- (Actualizado) Damos premisos de escritura a nuestro servidor web, para ello hacemos:

```shell
chown -R www-data:www-data /var/www
```

6.- Accedemos con el navegador a la web y creamos el fichero de configuración:

* http://<host>/wordpress/

## Mysql DB

Accedemos vía **SSH**:

```shell
ssh vm02@192.168.122.81
```

En el terminal escribimos `mysql -u root -padmin` nos pedirá una contraseña y escribimos la contraseña que le asignamos al instalar mysql-server-5.1. Una vez dentro de mysql escribimos

```mysql
CREATE DATABASE wordpress;
```

Ahora crearemos el usuario wordpressuser escribiendo:

```mysql
CREATE USER wordpressuser;
```

luego le asignaremos una contraseña este usuario, para esto escribimos:

```mysql
SET PASSWORD FOR wordpressuser = PASSWORD("admin");
```

Luego le damos privilegios a este usuario con:

```mysql
GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@'%' IDENTIFIED BY 'admin';
```

## MYSQL acceso externo

Acceder al fichero `/etc/mysql/my.cnf` y comentar la línea **bind-address**.

```file
[mysqld]
# bind-address  = 192.168.122.81
port            = 3306
user		    = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp
language        = /usr/share/mysql/English
# skip-networking
# skip-external-locking
```

Guardar el fichero y restart mysql:

```shell
service mysql restart
```

Añadir los permisos para que acceda el usuario **root**:

```shell
    mysql -uroot -padmin
    CREATE DATABASE mydb;
    # Grant permission to root from any host:
    GRANT ALL ON *.* TO root@'%' IDENTIFIED BY 'admin';
    Open Up MySQL Remote Ports

Conexión remota:

```
    /sbin/iptables -A INPUT -i eth0 -p tcp --destination-port 3306 -j ACCEPT
    mysql -h 192.168.122.81 -u root -padmin
```

## Host IP

Configurar el dominio de nombres en el ordenador para que acceda a através de la ip correspondiente:

```shell
$ sudo vi /etc/hosts
```

Añadir la línea de conversión entre IP y nombre:

```shell
192.168.122.80 system-integration.ricardogarfe.org
```

Acceder a la dirección de wordpress desde el host y configurar wordpress.

    https://system-integration.ricardogarfe.org/wordpress/

