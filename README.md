MSWL Systems Integration
=========================

# Introduction

Notes and Exercises for Systems Integration Subject coursed in [Master on Libre Software (Master Universitario en Software libre)](http://master.libresoft.es/) at [Universidad Rey juan Carlos](http://www.urjc.es/).

# Requirements

* Linux distribution.
* two USB with 8G.

# Final practice

The Final Practice will intend to integrate a virtual platform with a CMS **Wordpress**. This CMS will require to install a web server (**Apache**) and a DB engine (**MySQL**) and some network tweaks.

This platform will be deployed with a virtual structure: a Linux host (for instance your laptop) with **KVM** hypervisor, installed in **NAT** mode, allowing to deploy and to host two virtual machines **VM**:

* `Web Server` **raid-vm01**
* `Database` **raid-vm01**

Virtual Machine images will be hosted in a **LVM** from host.

System Architecture:

![Host Guest Virtualization](https://raw.github.com/ricardogarfe/mswl-systems-integration/master/images/final_practice.png)

Network Architecture: Linux Host (Dom-0) will have a private IP (at your choice). Virtual Machines will be inside a private subnet not accessed from outside the network.

Summarizing, with this practice we will have the following platform:
* Host (Dom-0):  private ip, Xen/KVM server (192.168.122.1) with Linux (such as Debian/Ubuntu)
* `raid-vm01` (DomU): Services (Apache), private ip (192.168.122.80), public access to the ports `80` y `22001`.
* `raid-vm02` (DomU): database (MySQL), private ip (192.168.122.81), accessed to MySQL port from
`raid-vm01`. Public access only to the ssh port `22002`.

## Extras

Extra issues to increase score:

* +2 points: Firewall in **Dom0** to filter/forward packets from outside the network, and it will allow virtual machines to access outside. The firewall has to allow connections to the ports `80`, `22001` and `22002`, forwarding the web requests to the virtual machine with a web server (`raid-vm01`). The firewall will also forward the ssh connections to both virtual machines (via `22001` and `22002` ports).
* +1 point: Virtual Machines will have to mount `LVM filesystems`. So different volumes (LVMs) for different VM mountpoints:
    * `/var/www` volume mounted in `raid-vm01`.
    * `/var/lib/mysql` volume mounted in `raid-vm02`.

* +1 point: Using `RAID 1`: LVM mounted with two (or more) physical hard disks (for instance two USB flash drives).
* +0.5 point: `HTTPS support` in web server (`raid-vm01`).

# Documentation

Create a wiki with all documentation extracted from `final_practice` folder in repostiory:

```shell
final_practice/
├── 001-logical-disc-layer.md
├── 002-virtualizacion.md
├── 003-wordpress-mysql-server.md
├── 004-secure-apache-ssl.md
├── 005-iptables.md
└── 006-fstab.md
```

Wiki [Systems Integration Documentation](https://github.com/ricardogarfe/mswl-systems-integration/wiki).

# License

<a href="http://creativecommons.org/licenses/by/3.0/" rel="Creative Commons Attribution 3.0">![Foo](http://i.creativecommons.org/l/by/3.0/88x31.png)</a>

This work by Ricardo Gracía Fernández - ricardogarfe [at] gmail [dot] com is licensed under a [Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/).

