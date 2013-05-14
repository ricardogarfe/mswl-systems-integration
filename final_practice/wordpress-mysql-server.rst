IP configuration
=================

inet addr:192.168.122.80 Bcast:192.168.122.255 Mask:255.255.255.0

sudo pico /etc/network/interfaces

configurar DNS, añadir la línea **nameserver 192.168.122.1** con la ip del host en el fichero **/etc/resolvconf/resolv.conf.d/base**

Apache ws01
------------

ifconfig eth0 192.168.122.80 netmask 255.255.255.0 broadcast 192.168.122.255

auto eth0
iface eth0 inet static
    address 192.168.122.80
    netmask 255.255.255.0
    broadcast 192.168.122.255
    gateway 192.168.122.1

service networking restart

Mysql db01
-----------

ifconfig eth0 192.168.122.81 netmask 255.255.255.0 broadcast 192.168.122.255

auto eth0
iface eth0 inet static
    address 192.168.122.81
    netmask 255.255.255.0
    broadcast 192.168.122.255
    gateway 192.168.122.1

service networking restart

Wordpress
==========

ssh vm01@192.168.122.80

Instalar wordpress en un servidor ubuntu 12.04 - Dinux 7 con apache y mysql
1.- Instalamos apache2, mysql y php. 

sudo apt-get install apache2 php5 php5-mysql mysql-server phpmyadmin

Nota 1: Al instalar mysql si te lo pide hay que elegir apache2 como servidor web.

Nota 2: Si ya tenéis instalado apache o mysql y no os funciona desinstalarlo y volverlo a instalar.

apt-get remove --purge phpmyadmin 
apt-get install phpmyadmin 

2.- Descargamos la última versión de wordpress

cd /var/www
wget http://wordpress.org/latest.tar.gz

3.- Descomprimimos y descompactamos

tar -zxvf  latest.tar.gz

4.- Creamos la base de datos wordpress en mysql

http://localhost/phpmyadmin
Crear base de datos
nombre: wordpress con cotejamiento y crear 

5.- (Actualizado) Damos premisos de escritura a nuestro servidor web, para ello hacemos:
chown -R www-data:www-data /var/www

6.- Accedemos con el navegador a la web y creamos el fichero de configuración:

http://localhost/wordpress/

BBDD
=====

ssh vm02@192.168.122.81

En el terminal escribimos `mysql -u root -padmin` nos pedirá una contraseña y escribimos la contraseña que le asignamos al instalar mysql-server-5.1. Una vez dentro de mysql escribimos

    CREATE DATABASE wordpress;

Ahora crearemos el usuario wordpressuser escribiendo:

    CREATE USER wordpressuser;

luego le asignaremos una contraseña este usuario, para esto escribimos:

    SET PASSWORD FOR wordpressuser = PASSWORD("admin");

Luego le damos privilegios a este usuario con:

    GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@'%' IDENTIFIED BY 'admin';

MYSQL-SERVER
=============

Replace mysqld Defaults

You can either set this up as a new connection or override the default, in this case, I replaced the default connection with my own remote connection settings. Keep in mind if you want to be creative and bind to localhost without interfering with other settings or services you can bind to 0.0.0.0 or another alias.

comment bind-address definition in /etc/mysql/my.cnf

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

After your done editing, just save and close the file and restart mysql services:

    # Ubuntu
    service mysql restart
    # Other Unix
    /sbin/init.d/mysql restart
    # Test that your connection is allowed with telnet on your local machine:
    telnet 255.112.324.12 3306
    Granting Remote Access to MySQL Users

Now we’ve created our remote config for MySQL, we have to grant access to this server to other machines.

    mysql -uroot -padmin
    CREATE DATABASE mydb;
    # Grant permission to root from any host:
    GRANT ALL ON *.* TO root@'%' IDENTIFIED BY 'admin';
    Open Up MySQL Remote Ports

Now that our user has been granted access from any host, all thats left is to make sure our OS will allow connections to the default MySQL port

    /sbin/iptables -A INPUT -i eth0 -p tcp --destination-port 3306 -j ACCEPT
    And now we should be able to login to our server from our local machine:

    mysql -h192.168.122.81 -u root -padmin

http://blog.vinhkhoa.com/article/Set-up-a-Railo-Apache-MySQL-host-on-Ubuntu-Part-2-install-MySQL-and-enable-remote-access
http://endpoint.co/technology/enable-remote-access-mysql

IP
===

Configurar el dominio de nombres en el ordenador para que acceda a através de la ip correspondiente:

    $ sudo vi /etc/hosts

Añadir la línea de conversión entre IP y nombre:

    192.168.122.80 system-integration.ricardogarfe.org

Acceder a la dirección de wordpress:

    https://system-integration.ricardogarfe.org/wordpress/

