===========================
MSWL - Systems Integration
===========================

Tecnologías de Virtualización
==============================

Xen es un hypervisor: software que nos permite crear máquinas virtuales. Ha sido portado a varios sistemas operativos.

Es ajeno al kernel. Permite virtualización en hardware que no está preparado para ello.

    * dom0: dominio de control, hypervisor y API de xen.
    * dom1, domn (domU): máquinas virtuales con dispositivos virtualizados.

El hypervisor se carga en el arranque a todos los niveles, como si estuviese 'incrustado' habiendo modificado el kernel tanto del dom0 como domU para colaborar entre ellas.

Xen-HVM
--------

Virtualización completa para poder virtualizar sistemas operativos no modificados (windows, etc...).

FreeBSD - Jails
----------------

Un único kernel a partir del que se crean instancias separadas. Cada una su ip, usuarios, etc. Aunque el grado de aislamiento es menor en comparación a la virtualización completa.

hypervisor BSD
---------------

bhyve - Virtualización completa a través'de un hypervisor. Reciente y usable en FreeBSD 9.

Sólo funciona con procesadores Intel. Equivalente a KVM en Linux aunque no soporta todas las opciones ni es tan completo.

illumos
--------

Contenedores y Zonas para la virtualización ligera. A partir de un Kernel instanciamos cada máquina virtual.

    * Nivel de aislamiento menor que virtualización completa.
    * Si se busca el modelo granja es una opción válida.
    * Las instancias han de ser del mismo sistema operativo.

Cloud Computing
================

SaaS - Software - usuarios.
PaaS - Platform - desarrolladores.
IaaS - Infraestructure - sysadmin.

Muy importante la eficiencia energética asociada al consumo eléctrico.

IaaS
-----

Públicas - Amazon AWS, Joyent.
Privadas - desplegadas en el interior de nuestra organización.
Híbrido - combinan ambos modelos mediante las APIs. OpenStack con Amazon WS.

OpenStack
----------

Hypervisor kvm y más, incluso privativos como vmware y uno de microsoft.
Utiliza estándares abiertos.
Compatibilidad mediante la API pública.






























