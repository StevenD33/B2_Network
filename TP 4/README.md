# TP4 : Buffet à volonté

## La topo

### Schéma GNS



### Tableau des réseaux

| Name     | Address        | VLAN |
|----------|----------------|------|
| `admins` | `10.5.10.0/24` | 10   |
| `guests` | `10.5.20.0/24` | 20   |
| `infra`  | `10.5.30.0/24` | 30   |

### Tableau d'adressage

| Machine  | `admins`      | `guests`      | `infra`       |
|----------|---------------|---------------|---------------|
| `r1`     | `10.5.10.254` | `10.5.20.254` | `10.5.30.254` |
| `admin1` | `10.5.10.11`  | x             | x             |
| `admin2` | `10.5.10.12`  | x             | x             |
| `admin3` | `10.5.10.13`  | x             | x             |
| `guest1` | x             | `10.5.20.11`  | x             |
| `guest2` | x             | `10.5.20.12`  | x             |
| `guest3` | x             | `10.5.20.13`  | x             |
| `dhcp`   | x             | `10.5.20.253` | x             |
| `dns`    | x             | x             | `10.5.30.11`  |
| `web`    | x             | x             | `10.5.30.12`  |


### Checklist

- [x] **1.** setup la topo dans GNS  
- [x] **2.** mettre en place les VLANs  
- [x] **3.** définir les IPs statiques des admins, guests et routeurs  
- [x] **4.** (optionnel-mais-conseillé) mettre en place le serveur DHCP  
- [x] **5.** (optionnel-mais-conseillé) mettre en place le serveur DNS  
- [x] **6.** (optionnel) mettre en place le serveur Web  



### 1. Setup la Topo dans GNS 

![oui](https://github.com/StevenDias33/B2_Network/blob/master/TP%204/images/topo.png)





### 2. mettre en place les VLANs 

Pour commencer il faut d'abord créer nos sous-interfaces sur R1. 


```
interface FastEthernet1/0.30
 encapsulation dot1Q 30
 ip address 10.5.30.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
!
interface FastEthernet1/1.10
 encapsulation dot1Q 10
 ip address 10.5.10.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly in
!
interface FastEthernet1/1.20
 encapsulation dot1Q 20
 ip address 10.5.20.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly in

```

Une fois les sous interfaces configuré on peut mettre en place les vlan sur les différents switch. 


Config Switch client-sw1
Config Switch client-sw2
Config Switch client-sw3
Config Switch infra-sw1


Config R1 



### 3. définir les IPs statiques des admins, guests et routeurs


Guest 3 : 

```
NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
guest3 10.5.20.13/24        10.5.20.254       00:50:79:66:68:04  20039  127.0.0.1:20040
       fe80::250:79ff:fe66:6804/64


```
Admin 3 :
```
NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
admin3 10.5.10.13/24        10.5.10.254       00:50:79:66:68:03  20037  127.0.0.1:20038
       fe80::250:79ff:fe66:6803/64

```

Guest 2 : 

```

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
guest2 10.5.20.12/24        10.5.20.254       00:50:79:66:68:00  10005  127.0.0.1:10006
       fe80::250:79ff:fe66:6800/64

```

Admin 2 : 

```
NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
admin2 10.5.10.12/24        10.5.10.254       00:50:79:66:68:02  20033  127.0.0.1:20034
       fe80::250:79ff:fe66:6802/64

```

guest 1 : 

```

NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
guest1 10.5.20.11/24        10.5.20.254       00:50:79:66:68:01  20031  127.0.0.1:20032
       fe80::250:79ff:fe66:6801/64

```

admin 1 : 

```
NAME   IP/MASK              GATEWAY           MAC                LPORT  RHOST:PORT
admin1 10.5.10.11/24        10.5.10.254       00:50:79:66:68:00  20035  127.0.0.1:20036
       fe80::250:79ff:fe66:6800/64

```


### 4. mettre en place le serveur DHCP


Pour mettre en place le serveur dhcp il faut d'abord installer le dhcp-server package. 

```
dnf update
dnf install dhcp-server
```

Ensuite il faut configurer le fichier. `/etc/dhcp/dhcpd.conf`

Une fois cela fait on start le service. 

`systemctl start dhcpd`


et je confgure ensuite un des vpcs pour vérifier que le dhcp fonctionne.



on peut voir que le DORA à bien fonctionné et je peux même le voir dans mon wireshark voir pcap. 

```
guest3> ip dhcp
DORA IP 192.168.200.131/24 GW 10.5.20.254

```


### 5. mettre en place le serveur DNS

En ce qui concerne le serveur DNS il faut d'abord installer les paquets. 


`dnf install bind bind-utils`

Ensuite on config les fichiers : 

`/etc/named.conf`
`/var/named/tp4.b2.db`
`/var/named/20.5.10.db`

on ouvre le port 53 du firewall avec notre amis firewall-cmd et on démarre le service.


### 6. mettre en place le serveur Web

Installer epel-release et nginx


```
dnf install epel-release
dnf install nginx
```
on démarre le service et on ouvre le port 80 avec firewall-cmd.

et lors que je curl j'ai bien la page par défaut de nginx. 




## 2. Sécurité attack & defense : spoofing, VLAN attack

**But : hackers gonna hack (or not ?)**
* mettre en place des attaques réseau
* mettre en place des contremesures

### A. Offensive 

#### ARP Spoofing

Usurpation d'identité L2/L3 : on usurpe l'adresse IP d'une machine du réseau en empoisonnant la table ARP de la victime.

Je recommande :
* [la commande `arping`](https://sandilands.info/sgordon/arp-spoofing-on-wired-lan)
* la librairie Python `scapy`
  * vous trouverez BEAUCOUP d'exemples sur internet

Le concept :
* l'attaquant envoie un ARP request (en broadcast donc)
  * il demande l'IP de la victime et précise
    * sa vraie MAC en source
    * une fausse IP en source
* la victime reçoit le message
  * la victime répond à l'attaquant avec un ARP reply (l'attaquant obtient l'adresse MAC de la victime)
  * **la victime inscrit dans sa table ARP les infos de la trame reçue** :
    * désormais la victime pense que l'IP envoyée (fausse) correspond à la MAC envoyée (celle de l'attaquant)
* répéter l'échange pour maintenir l'attaque en place
  * les entrées dans les tables ARP sont temporaires (de 60 à 120 secondes en général)

Pour mettre en place un man-in-the middle :
* l'attaquant usurpe l'IP de la victime auprès de la passerelle du réseau
* l'attaquant usurpe l'IP de la passerelle auprès de la victime
* tous les messages que la victime envoie vers d'autres reéseaux (internet par exemple) passent par l'attaquant

#### DHCP Spoofing

Mettre en place un rogue DHCP, aussi appelé "DHCP server of the doom" *(padutou)*.  
On met en place un serveur DHCP, et on file des IPs aux clients. On tire alors profit des *options DHCP* afin de fournir d'autres fausses infos aux clients. Principalement : adresse du serveur DNS du réseau, adresse de la passerelle.

On peut donc définir arbitrairement l'adresse de passerelle et celle du DNS pour les clients. Cela mène à des attaques Man-in-the-middle et DNS spoofing.

#### DNS Spoofing

Il est nécessaire de faire l'ARP spoofing ou le DHCP spoofing pour ça.

Effectuer des réponses DNS fallacieuses. Une fois l'identité du serveur DNS ou de la passerelle d'un réseau usurpée, on peut répondre aux requêtes DNS des clients. 

#### Messing up with VLANs

Forcer un switch à négocier un trunk pour récupérer tout le trafic et envoyer des messages à tout le réseau.

Une fois le trunk négocié, il est possible de :
* voir toutes les trames broadcast envoyées (entre autres)
  * donc on récupère des MAC, des IPs, et le VLAN associé
* envoyer des trames dans n'importe quel VLAN
  * enfin, n'importe quel VLAN auquel le switch avec qui on a négocié a accès

### B. Defensive

#### ARP Spoofing

Mise en place d'un *IP source guard*.

#### DHCP Spoofing

Mise en place de *DHCP snooping* qui permet d'interdire des trames DHCP sur les ports non-autorisés.

#### Messing up with VLANs

Mise en place de contre-mesures contre les attaques liées aux VLANs :
* désactiver explicitement les ports non utilisés
  * *a minima* désactiver la négociation
* le mécanisme *IP source guard* peut aussi aider
