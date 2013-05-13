===========================
MSWL - Systems Integration
===========================

Firewalls - FW
===============

Nat, bridge, al final lo que gestionan es **iptables**.

Direfencias entre un FW ADSL y un FW profesional.

DMZ
====

Zona Desmirilitarizada, aisla con un puente entre dos FWs las conexiones a la red interna.

Filtrado de paquetes
=======================

* Filtrado: filtrar por IP, internas, externas, etc... 
* Modificación:
* NAT: redirigir tráfico

Filtrar por IP
---------------

Capa de red layer-3. No ha de saber nada de lo que hay debajo de la red (aplicaciones) de esta área se encargan los proxies.

Conceptos
----------

* Aceptar o Denegar - **default deny**
* Inspección de paquetes: estado o sin estado - stateful vs stateless. Una conexión de salida vuelve y por lo tanto sabe de donde viene.
* Spoofing: registra las conexión de I/O.
* Inspección de estado guarda registros.
* Establecimiento de TCP/IP:
    * SYN packet
    * SYN + ACK packet sincronización y acuse de recibo del servidor
    * ACK packet: acuse de recibo del cliente.

Herramientas
-------------

* **iptables** Linux.
* **ipfilter** Solaris, derivados, FreeBSD, Linux.
* **PF Packet Filter** OpenBSD, FreeBSD, DragonFly, MacOSX, Blackberry.


