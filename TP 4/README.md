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



on peut voir que le DORA à bien fonctionné et je peux même le voir dans mon wireshark voir [pcap](https://github.com/StevenDias33/B2_Network/blob/master/TP%204/dora.pcapng) . 

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

Pour faire de l'arp spoof il y'a plusieurs maniere de procéder par exmple on peut utiliser arpspoof sur kali 

En premier lieu on check la passerelle avec :
ip route

On scan le réseau:
nmap  192.168.1.0/24

Now start the ARP Poisoning/Spoofing:
arpspoof -i eth0 -t Ipvictime -r Gateway
-i interface.
-t target.
-r default gateway.


Une fois cela fait l'arpspoof est fonctionnel et les messages passent par l'attaquant voir arpspoof.pcap


#### DHCP Spoofing

``` ettercap -Tzq -M dhcp:/255.255.255.0/10.5.30.11 ```
```
DHCP : [00:0C:29:FF:09:88] DISCOVER 
DHCP: [00:0C:29:FF:09:88] REQUEST 10.5.20.142
DHCP spoofing: fake ACK [00:0C:29:FF:09:88] assigned to 10.5.20.142
DHCP: [10.5.20.142] ACK : 192.168.80.1 255.255.255.0 10.5.20.254 DNS 10.5.30.11 
DHCP: [10.5.20.253] ACK : 192.168.80.1 255.255.255.0 10.5.20.254 DNS 10.5.30.11 "tp4.b2"
```



### B. Defensive

#### ARP Spoofing / DHCP Spoofing / Messing up with VLANs

Pour mettre en place une défense contre l'arp spoof le DHCP spoof et le fait qu'une personne force un switch à négocier des trunks on fait 

```
conf t 
ip dhcp snooping
ip dhcp snooping vlan 10 20 30
interface fastethernet 0/1
no ip dhcp snooping trust
ip verify source vlan dhcp-snooping

```

on peut aussi configurer une ip static bindé sur le port 
```
ip source binding mac_address vlan vlan-id ip-address interface interface_name
```

à la fin de la configuration on fait un show ip verify source pour bien voir que la configuration est appliqué. 
