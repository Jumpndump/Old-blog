---
layout: default
permalink: /malwarelab/tuto/mettre-en-place-lab
title: Mettre en place un lab d'analyse basique
tag: tuto_malware
category: post
---

## Mettre en place un lab d'analyse basique

Afin d'analyser des malwares, il est nécessaire de se monter un environnement d'analyse si l'on ne veut pas mettre en péril tout le réseau... Il existe plusieurs façon de conçevoir son architecture (cf. références). Mais on parlera ici d'un environnement d'analyse très simpliste et basique, rapide et facile à mettre en place, et donc pour les petits malwares pas trop tricky non plus... Si vous voulez analyser de la bonne grosse APT, il vous faudra certainement quelque chose de plus robuste (attention à la détection de l'hyperviseur, et donc à l'exploitation des ses vulnérabilités)!

## Configurer le réseau de la machine virtuelle d'analyse

Afin de séparer la machine d'analyse de notre réseau et de ne pas griller notre IP, nous allons rediriger le trafic sortant vers un réseau anonymisant comme par exemple Tor.

Pour ce faire, il est nécessaire d'installer les les outils suivants :
```
sudo apt-get install bridge-utils dnsmasq tor
```

### Configuration de l'interface réseau

Nous allons d'abord configurer l'interface de notre machine d'analyse, que nous nommerons ```vnetO```, en bridge. Ainsi, la machine virtuelle obtiendra Internet directement via la carte réseau et non en passant par l'hôte (pour d'éviter tout lien logique entre les deux machines) :

Dans le fichier **/etc/network/interfaces**
```
#Virtualbox NAT Bridge
auto vnet0
iface vnet0 inet static
 address 172.16.0.1
 netmask 255.255.255.0
 bridge_ports none
 bridge_maxwait 0
 bridge_fd 1

 up iptables -t nat -I POSTROUTING -s 172.16.0.0/24 -j MASQUERADE
 down iptables -t nat -D POSTROUTING -s 172.16.0.0/24 -j MASQUERADE
```

Une fois notre interface configurée, il nous faut l'activer :
```sh
ifup vnet0
```
### Configuration du Dnsmasq

Maintenant que nous avons notre interface, il nous faut lui attribuer un range d'IP possible pour le DHCP.

Dans **/etc/dnsmasq.conf**
```
interface=vnet0
dhcp-range=172.16.0.2,172.16.0.254,1h
```

### Configuration de Tor

Enfin, nous configurons Tor afin de lui dire quel réseau on veut anonymiser.

Dans **/etc/tor/torrc**
```
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsOnResolve 1
TransPort 9040
TransListenAddress 172.16.0.1
DNSPort 53
DNSListenAddress 172.16.0.1
```

## Créer la machine virtuelle d'analyse Windows

C'est la machine qui va nous permettre d'analyser le malware. Comme la plupart ciblent des machines Windows, il est préférable d'opter pour la version la plus récente afin d'être sûre qu'il soit compatible (cela peut paraître évident, mais l'architecture des Windows changeant légèrement d'une version à une autre, un malware qui fonctionne sous Windows 10 ne tournera pas forcement sous Windows 7).

Il vous faut donc créer une machine virtuelle Windows et configurer son réseau en mode bridge sur l'interface ```vnet0``` que nous venons de configurer.

Sur virtualbox :
![vbox_bridge](/img/bridge_malwarelab.png)

## Ya plus qu'à !

Maintenant que vous avez configuré votre interface, la redirection et la machine virtuelle, il ne vous reste plus qu'à lancer les services cités plus haut et allumer votre VM.

```
sudo systemctl start dnsmasq && sudo systemctl start bridge-utils && sudo systemctl start tor
```

Vérifiez si vos services se sont correctement lancés avec un ```sudo systemctl status <service>```.

Pour les plus fénéants, un script d'automatisation est disponible à cet endroit : https://github.com/DevanShellc0de/torrifier.git ;)

<br><br>
_References_

---
https://www.howtoforge.com/how-to-set-up-a-tor-middlebox-routing-all-virtualbox-virtual-machine-traffic-over-the-tor-network
