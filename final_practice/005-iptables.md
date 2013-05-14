## IPTables

## Port Forwarding

Puertos `80` para web en `raid-vm01` y `22001`, `22002` para ssh en `raid-vm01` y `raid-vm02` respectivamente:

```shell
iptables -t nat -I PREROUTING -p tcp -d 192.168.1.39 --dport 80 -j DNAT --to-destination 192.168.122.80:80
iptables -t nat -I PREROUTING -p tcp -d 192.168.1.39 --dport 22001 -j DNAT --to-destination 192.168.122.80:22
iptables -t nat -I PREROUTING -p tcp -d 192.168.1.39 --dport 22002 -j DNAT --to-destination 192.168.122.81:22
iptables -I FORWARD -m state -d 192.168.122.0/24 --state NEW,RELATED,ESTABLISHED -j ACCEPT
```

Port 443:

```shell
iptables -t nat -I PREROUTING -p tcp -d 192.168.1.39 --dport 443 -j DNAT --to-destination 192.168.122.80:443
```

Guardar los cambios:

```shell
$ iptables-save
```

No es necesario abrir los puertos en el host:

```shell
    Allow connection to ports 22001 and 22002:
    * iptables -A INPUT -i wlan0 -p tcp --dport 22001 -j ACCEPT
    * iptables -A INPUT -i wlan0 -p tcp --dport 22002 -j ACCEPT
```

### Filtro por interfaz

eth0:

* iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j DNAT --to 192.168.122.81:80
* iptables -A FORWARD -p tcp -d 192.168.122.80 --dport 8080 -j ACCEPT

wlan0:

* iptables -A PREROUTING -t nat -i wlan0 -p tcp --dport 80 -j DNAT --to 192.168.122.80:80
* iptables -A FORWARD -p tcp -d 192.168.122.80 --dport 80 -j ACCEPT

* iptables -D PREROUTING -t nat -i wlan0 -p tcp --dport 80 -j DNAT --to 192.168.122.80:80
* iptables -D FORWARD -p tcp -d 192.168.122.80 --dport 80 -j ACCEPT

## Comprobar tráfico puerto 80

Utilizando `tcpdump` para ver el tráfico entre ips, puertos, redes, interfaces, etc..

```shell
tcpdump -i virbr10 host 192.168.122.80
```

## SSH

Reglas:

```shell
$ iptables -I FORWARD -m state -d 192.168.122.0/24 --state NEW,RELATED,ESTABLISHED -j ACCEPT
$ iptables -t nat -I PREROUTING -p tcp --dport 22001 -j DNAT --to-destination 192.168.122.80:22
```

Probar la conexión `ssh` a través de otro dispositivo conectado a la misma red que el `host`:

```shell
ssh -p 22001 vm01@192.168.1.39
ssh -p 22002 vm02@192.168.1.39
```

## Borrar reglas

Para borrar las reglas se ha de ejecutar el mismo comando pero cambiando la **-A** por **-D** (de delete), por ejemplo:

```shell
iptables -A INPUT -i eth0 -p tcp --dport 443 -j ACCEPT
```

Se convierte en:

```shell
iptables -D INPUT -i eth0 -p tcp --dport 443 -j ACCEPT
```

Guardar los cambios:

```shell
$ iptables-save
```

## Backup Iptables

Existen dos comandos para hacer esto: iptables-save y iptables-restore. Ambas leen y escriben la entrada y salida estándares. Ejemplos:

```shell
root# iptables-save > iptables.txt  
 
root# iptables-restore < iptables.txt
```

