
# 👨‍💻 TP6 - - Services réseau

CPE Lyon - 4ETI, 3IRC & 3ICS - Année 2022/2023 Administration Système

Corentin JACQUIER ICS

## Exercice 1 - Adressage IP 

On a donc les 7 sous réseaux suivants avec 3 bits (au moins /26).

Il nous reste 6 bits par machines donc, 2^6 -2 = 62 machines. 

172.16.0.0/26
172.16.0000000**0.00**000000
52 - 000 - 1

172.16.0.64/26
172.16.0000000**0.01**000000
38 - 001 - 2

172.16.0.128/26
172.16.0000000**0.10**000000
37 - 010 - 3

172.16.0.192/26
172.16.0000000**0.11**000000
37 - 011 - 4

172.16.1.0/26
172.16.0000000**1.00**000000
35 - 100 - 5

172.16.1.64/26
172.16.0000000**1.01**000000
33 - 101 - 6

172.16.1.128/27
172.16.0000000**1.100**00000
25 - 110 - 7 -> VLSM (sous réseau dans sous-réseau plus petit) 

Et après 172.16.1.160/27....

Alors : 

Sous-réseau 1 : 172.16.0.0/26  
Adresse de broadcast : 172.16.0.63
Première adresse de machine : 172.16.0.1
Dernière adresse de machine : 172.16.0.62  

Sous-réseau 2 :  172.16.0.64/27
Adresse de broadcast : 172.16.0.127
Première adresse de machine : 172.16.0.65
Dernière adresse de machine : 172.16.0.126  

Sous-réseau 3 : 172.16.0.128/26  
Adresse de broadcast : 172.16.0.191  
Première adresse de machine : 172.16.0.129  
Dernière adresse de machine : 172.16.0.190  

Sous-réseau 4 : 172.16.0.192/27  
Adresse de broadcast : 172.16.0.255  
Première adresse de machine : 172.16.0.193  
Dernière adresse de machine : 172.16.0.254  

Sous-réseau 5 : 172.16.1.0/26 
Adresse de broadcast : 172.16.1.63  
Première adresse de machine : 172.16.1.1  
Dernière adresse de machine : 172.16.1.62  

Sous-réseau 6 : 172.16.1.64/26  
Adresse de broadcast : 172.16.1.127  
Première adresse de machine : 172.16.1.65  
Dernière adresse de machine : 172.16.1.126  

Sous-réseau 7 : 172.16.1.128/27  
Adresse de broadcast : 172.16.1.191  
Première adresse de machine : 172.16.1.129  
Dernière adresse de machine : 172.16.1.190

## Exercice 2 - Préparation de l’environnement

Machine 1 = client et machine 2 = serveur. 

On ajoute les cartes réseaux aux machines sur l'interface Vsphere en cliquant droit sur la machine et dans modifier les paramètres. On change la carte réseau du client en  `ICS_E12_..` et on ajoute au serveur une seconde carte `ICS_E12_..`.

On éteint les machines, et on met en place l'environement de commande. 

Avec la commande `ip a` on voit que l'interface `lo` correspond à l'addresse `loopback`  (127.0.0.1/8) pour le local. 

<img src="https://cdn.discordapp.com/attachments/1017478318934724638/1023856617554460723/unknown.png"> 

Pour supprimer le paquet, on fait `sudo apt remove cloud-init` sur les deux machines.

On change le hostname avec la commande `hostnamectl set-hostname tpadmin.local --static` sur le serveur. 

On entre 1 et le mot de passe User. 

On verifie avec `hostname status` ou `hostname`, si ça a bien enregristré après redémarrage de la VM. Et c'est bon. 

## Exercice 3 - Installation du serveur DHCP

On installe le paquet  `isc-dhcp-server`, avec la commande `sudo apt install isc-dhcp-server`. 

La commande `systemctl status isc-dhcp-server` retourne : `Unit ics-dhcp-server. service could not be found`. 

Dans le dossier `/etc/netplan`, on créer le fichier `01-netcfg.yaml` dans lequel on écrit :

network : 
   version : 2 
   renderer : networkd 
   ethernets :
      ens224 : 
      addresses : 
         − 192.168.100.1/24

On fait `sudo netplan try` pour vérifier que la configuration fonctionne. Puis, on fait `sudo netplan apply`  pour l'appliquer. 

D'abord on fait une backup du fichier `dhcp.conf` avec `sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak`.

Ensuite, on fait la configuration du serveur DHCP en éditant le fichier `dhcp.conf` avec : `sudo nano /etc/dhcp/dhcpd.conf`. 

On écrit les paramètres dans le fichier.

Et on modifie également le fichier `/etc/default/isc-dhcp-server` pour donner l'interface à utiliser : ens224.





