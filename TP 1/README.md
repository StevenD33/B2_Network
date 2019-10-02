# Rendu Tp 1 Léo 


## Gather informations
| Cartes réseaux  | Nom | Adresse mac | IP| 
|:--------:| -------------:|  -------------:|  -------------:| 
| NAT | ens33 |  00:0c:29:d4:29:8a   |   192.168.178.130   | |
| Host-only | ens37 | 00:0c:29:d4:29:94  |   192.168.216.129   | |


Les cartes réseaux ont récupéré une ip via le DHCP que j'ai config via vmware.

Baux DHCP Host-only

	ADDRESS=192.168.216.129
	NETMASK=255.255.255.0
	SERVER_ADDRESS=192.168.216.254
	NEXT_SERVER=192.168.216.254
	BROADCAST=192.168.216.255
	T1=900
	T2=1575
	LIFETIME=1800
	DNS=192.168.216.1
	DOMAINNAME=localdomain
	CLIENTID=01000c29d42994

Baux DHCP NAT

	ADDRESS=192.168.178.130
	NETMASK=255.255.255.0
	ROUTER=192.168.178.2
	SERVER_ADDRESS=192.168.178.254
	NEXT_SERVER=192.168.178.254
	BROADCAST=192.168.178.255
	T1=900
	T2=1575
	LIFETIME=1800
	DNS=192.168.178.2
	DOMAINNAME=localdomain
	CLIENTID=01000c29d4298a

Table ARP 

	_gateway (192.168.178.2) at 00:50:56:f4:91:bc [ether] on ens33
	
	connexion entre la gateway et ens33
	? (192.168.216.1) at 00:50:56:c0:00:01 [ether] on ens37
	? (192.168.216.254) at 00:50:56:e7:e0:66 [ether] on ens37

Table de routage 

	192.168.178.0/24 dev ens33 proto kernel scope link src 192.168.178.130 metric 102
	192.168.216.0/24 dev ens37 proto kernel scope link src 192.168.216.129 metric 101


Les deux routes dans la table routage permette de se connecter avec l'host et pour la premiere route de se connecter à internet 

	Active Internet connections (only servers)
	Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
	tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      919/sshd
	tcp6       0      0 :::22                   :::*                    LISTEN      919/sshd
	udp        0      0 127.0.0.1:323           0.0.0.0:*                           822/chronyd
	udp        0      0 192.168.178.130:68      0.0.0.0:*                           905/NetworkManager
	udp        0      0 192.168.216.129:68      0.0.0.0:*                           905/NetworkManager
	udp6       0      0 ::1:323                 :::*                                822/chronyd

Le port en écoute est le port 22 qui sert à la connexion SSH.

	search localdomain
	nameserver 192.168.178.2
	nameserver 192.168.216.1

Les serveurs DNS sont la gateway de chaque réseau qui est l’hôte 
Ensuite j'ai fait un dig de reddit.com car c'est important d'avoir les bonnes adresses voici le résultat :

	dig reddit.com

	; <<>> DiG 9.11.4-P2-RedHat-9.11.4-17.P2.el8_0.1 <<>> reddit.com
	;; global options: +cmd
	;; Got answer:
	;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 30005
	;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 4, ADDITIONAL: 1

	;; OPT PSEUDOSECTION:
	; EDNS: version: 0, flags:; MBZ: 0x0005, udp: 4096
	;; QUESTION SECTION:
	;reddit.com.                    IN      A

	;; ANSWER SECTION:
	reddit.com.             5       IN      A       151.101.193.140
	reddit.com.             5       IN      A       151.101.65.140
	reddit.com.             5       IN      A       151.101.1.140
	reddit.com.             5       IN      A       151.101.129.140

	;; AUTHORITY SECTION:
	reddit.com.             5       IN      NS      ns-1887.awsdns-43.co.uk.
	reddit.com.             5       IN      NS      ns-1029.awsdns-00.org.
	reddit.com.             5       IN      NS      ns-557.awsdns-05.net.
	reddit.com.             5       IN      NS      ns-378.awsdns-47.com.

	;; Query time: 23 msec
	;; SERVER: 192.168.178.2#53(192.168.178.2)
	;; WHEN: Thu Sep 26 15:00:17 CEST 2019
	;; MSG SIZE  rcvd: 240

avec la commande firewall-cmd --list-all
je peux afficher la config actuelle de mon firewall et s'il est activé ou non.
	
	public (active)
	  target: default
	  icmp-block-inversion: no
	  interfaces: ens33 ens37
	  sources:
	  services: cockpit dhcpv6-client ssh
	  ports:
	  protocols:
	  masquerade: no
	  forward-ports:
	  source-ports:
	  icmp-blocks:
	  rich rules:


	NetidState   Recv-Q  Send-Q            Local Address:Port   Peer Address:Port
	udp  UNCONN  0       0                     127.0.0.1:323         0.0.0.0:*     users:(("chronyd",pid=822,fd=6))
	udp  UNCONN  0       0         192.168.178.130%ens33:68          0.0.0.0:*     users:(("NetworkManager",pid=905,fd=19))
	udp  UNCONN  0       0         192.168.216.129%ens37:68          0.0.0.0:*     users:(("NetworkManager",pid=905,fd=22))
	udp  UNCONN  0       0                         [::1]:323            [::]:*     users:(("chronyd",pid=822,fd=7))
	tcp  LISTEN  0       128                     0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=919,fd=6))
	tcp  LISTEN  0       128                        [::]:22             [::]:*     users:(("sshd",pid=919,fd=8))

Changement de configuration de la carte host only 
Création du fichier ifcfg-ens37 dans `etc/sysconfig/network-script`
	DEVICE=ens37
	
	BOOTPROTO=static
	IPADDR=192.168.216.130
	ONBOOT=yes

vérification de la config de la carte ens37 


	3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
	    link/ether 00:0c:29:d4:29:94 brd ff:ff:ff:ff:ff:ff
	    inet 192.168.216.130/24 brd 192.168.216.255 scope global noprefixroute ens37
	       valid_lft forever preferred_lft forever
	    inet6 fe80::20c:29ff:fed4:2994/64 scope link
	       valid_lft forever preferred_lft forever 


Ajout d'une nouvelle carte réseau host-only 



	2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
	    link/ether 00:0c:29:d4:29:8a brd ff:ff:ff:ff:ff:ff
	    inet 192.168.178.130/24 brd 192.168.178.255 scope global dynamic noprefixroute ens33
	       valid_lft 1606sec preferred_lft 1606sec
	    inet6 fe80::6660:3735:ee36:ca29/64 scope link noprefixroute
	       valid_lft forever preferred_lft forever
	3: ens37: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
	    link/ether 00:0c:29:d4:29:94 brd ff:ff:ff:ff:ff:ff
	    inet 192.168.216.130/24 brd 192.168.216.255 scope global noprefixroute ens37
	       valid_lft forever preferred_lft forever
	    inet6 fe80::20c:29ff:fed4:2994/64 scope link
	       valid_lft forever preferred_lft forever
	4: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
	    link/ether 00:0c:29:d4:29:9e brd ff:ff:ff:ff:ff:ff
	    inet 192.168.142.128/24 brd 192.168.142.255 scope global dynamic noprefixroute ens38
	       valid_lft 1799sec preferred_lft 1799sec
	    inet6 fe80::2126:8457:3f5c:e269/64 scope link noprefixroute
	       valid_lft forever preferred_lft forever


Cela à ajouter une nouvelle route sur la table de routage 


	default via 192.168.178.2 dev ens33 proto dhcp metric 106
	192.168.142.0/24 dev ens38 proto kernel scope link src 192.168.142.128 metric 107
	192.168.178.0/24 dev ens33 proto kernel scope link src 192.168.178.130 metric 106
	192.168.216.0/24 dev ens37 proto kernel scope link src 192.168.216.130 metric 103

et voici la nouvelle table arp 

	_gateway (192.168.178.2) at 00:50:56:f4:91:bc [ether] on ens33
	? (192.168.216.1) at 00:50:56:c0:00:01 [ether] on ens37
	? (192.168.178.254) at 00:50:56:f6:8f:96 [ether] on ens33
	? (192.168.216.254) at 00:50:56:e7:e0:66 [ether] on ens37

ensuite j'ai modif la config du ssh sur le port 2222 et j'ai fait `ss -tulnp` pour vérifier son fonctionnement voici le résultat 

	ss -tulnp
	NetidState   Recv-Q  Send-Q            Local Address:Port   Peer Address:Port
	udp  UNCONN  0       0                     127.0.0.1:323         0.0.0.0:*     users:(("chronyd",pid=822,fd=6))
	udp  UNCONN  0       0         192.168.142.128%ens38:68          0.0.0.0:*     users:(("NetworkManager",pid=905,fd=25))
	udp  UNCONN  0       0         192.168.178.130%ens33:68          0.0.0.0:*     users:(("NetworkManager",pid=905,fd=19))
	udp  UNCONN  0       0                         [::1]:323            [::]:*     users:(("chronyd",pid=822,fd=7))
	tcp  LISTEN  0       128                     0.0.0.0:2222        0.0.0.0:*     users:(("sshd",pid=3132,fd=6))
	tcp  LISTEN  0       128                        [::]:2222           [::]:*     users:(("sshd",pid=3132,fd=8))

Le service sshd est en LISTEN sur le port 2222.

 
 ## Routage

| Machines | 192.168.200.0/24 | 192.168.150.0/24 | NAT  |
| :------: | :--------------: | :--------------: | :--: |
|    R1    | 192.168.200.254  | 192.168.150.254  | DHCP |
|  1   |  192.168.100.12   |        X         |  X   |
|  2   |        X         |  192.168.200.12  |  X   |



- Machine 1  :

		DEVICE=ens37
		BOOTPROTO=static
		IPADDR=192.168.100.12
		ONBOOT=yes

- Machine 2 :

		DEVICE=ens37
		BOOTPROTO=static
		IPADDR=192.168.200.12
		ONBOOT=yes

- Routeur 1 (R1) :
```cisco
R1(config)#int g0/0
R1(config-if)#ip add dhcp
R1(config-if)#int g1/0
R1(config-if)#ip add 192.168.100.254 255.255.255.0
R1(config-if)#int g2/0
R1(config-if)#ip add 192.168.200.254 255.255.255.0
R1(config)#do sh ip int br
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES unset  administratively down down
GigabitEthernet0/0     10.0.0.145 YES DHCP   up                    up
GigabitEthernet1/0     192.168.100.254 YES manual up                    up
GigabitEthernet2/0     192.168.200.254 YES manual up                    up
```
Une fois la config centos de faite je peux ping google en passant par le routeur r1 
j'ai aussi fait la manip avec vagrant pour tester sauf que je n'ai pas trouvé de box centos8
du coup je l'ai fait sur centos7 et ça fonctionne bien j'ai up 3 VM sur 2 network  avec la vm master en Routeur pour donner internet  aux 2 autres 
## Autres applications et métrologie 


iftop permet de voir toutes les connexion de la machine et de représenter visuellement le débit 
cela permet de voir quel tache ou process pompe de la connexion  
il existe différentes options qui permettent de trier les données 

comme par exemple -t -   qui regroupe les débits entrants et sortants d'une connexion en une seule ligne 
après man sur iftop pour voir les autres 

### Cockpit 

Cockpit est un outil pour faire de l'admin sys et permet d'utiliser une interface web, je l'ai déja utiliser en stage mais je préfère zabbix car il permet de gérer plusieurs vm en même temps et de tout regrouper sur une même interface mais c'est completement différent car zabbix est un outils de monitoring, cockpit permet d'intéragir avec la vm directement via l'interface web, 
par exemple on peut désactiver Selinux ou gérer les différent services. 

l'intéret dans le réseau de cockpit c'est de manager plus rapidement et facilement une vm et de faire du monitoring pour récup de la data et prendre de l'info. 
```
 dnf -y install cockpit
sudo systemctl enable --now cockpit.socket
sudo firewall-cmd --add-service=cockpit --permanent
```

### Netdata 


Netdata permet de faire du monitoring en temps réel sur des applis et des systeme j'ai pu aussi l'utiliser lors de mon stage. 
Le gros avantage c'est qu'il est open source et tourne sur beaucoup de système et ne pompe pas beaucoup de ressources 
Comme cockpit on peut faire plein de truc comme avoir un systeme de notification qui pop une alerte si le CPU est surchargé par exemple. 


