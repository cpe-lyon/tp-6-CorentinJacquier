
# 👨‍💻 TP6 - Services réseau

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

Avec la commande `ip a` on voit que l'interface `lo` correspond à l'adresse `loopback`  (127.0.0.1/8) pour le local. 

<img src="https://cdn.discordapp.com/attachments/1017478318934724638/1023856617554460723/unknown.png"> 

Pour supprimer le paquet, on fait `sudo apt remove cloud-init` sur les deux machines.

On change le hostname avec la commande `hostnamectl set-hostname tpadmin.local --static` sur le serveur. 

On entre 1 et le mot de passe User. 

On verifie avec `hostname status` ou `hostname`, si ça a bien enregristré après redémarrage de la VM. Et c'est bon. 

## Exercice 3 - Installation du serveur DHCP

On installe le paquet  `isc-dhcp-server`, avec la commande `sudo apt install isc-dhcp-server`. 

La commande `systemctl status isc-dhcp-server` retourne : `Unit ics-dhcp-server. service could not be found`. 

Dans le dossier `/etc/netplan` du serveur, on écrit dans le fichier `50-cloud-init.yaml` :

network: 
  version: 2 
  renderer: networkd 
  ethernets: 
    ens192: 
      dhcp4: true 
    ense224 
      adresses: 
         - 192.168.100.1/24

On fait `sudo netplan try` pour vérifier que la configuration fonctionne. Puis, on fait `sudo netplan apply`  pour l'appliquer. Et on active l'interface : `sudo ip link set ens224 up`.

Puis on fait de meme avec le client, avec les données suivantes dans le fichier `50-cloud-init.yaml` :

network: 
  version: 2 
  renderer: networkd 
    ethernets: 
      ens192 
      adresses: 
         - 192.168.100.2/24

D'abord on fait une backup du fichier `dhcp.conf` avec `sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak`.

Ensuite, on fait la configuration du serveur DHCP en éditant le fichier `dhcp.conf` avec : `sudo nano /etc/dhcp/dhcpd.conf`. 

On écrit les paramètres demandés dans le fichier.

Et on modifie également le fichier `/etc/default/isc-dhcp-server` pour donner l'interface à utiliser : `ens224` dans ipV4.

On fait `dhcpd -t` pour vérifier la config :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024210729227780136/unknown.png">

Et le status avec la commande : 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024210899013210162/unknown.png">

Maintenant sur le client on change le hostname avec `hostnamectl set-hostname client.tpadmin.local --static` . 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024210596377395200/unknown.png">

On ajoute `optionnal: true` dans le yaml `/etc/netplan` pour rendre le démarage plus rapide du client.

On fait la commande `tail -f /var/log/syslog` :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024215565394522142/unknown.png">

Par exemple DHCPDISCOVER  est la réponse des machines. En effet, on retoruve l'adresse MAC du client.

Le fichier `/var/lib/dhcp/dhcpd.leases` liste toutes les connexions faites du serveur DHCP. 

Et la commande `dhcp-lease-list` affiche la meme liste mais dans le terminal sous forme de tableau :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024218815992713216/unknown.png"> 

On ping le client depuis le serveur : 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024218304182759514/unknown.png">

Et du client vers le serveur :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024218450429743104/unknown.png">

On fait les modifications suivantes dans le fichier `dhcpd.conf` (sans oublier le 's' à client) :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024221494093549629/unknown.png">

Pour fixer problèmes d'adresse IP et internet du serveur : `sudo dhclient ens192`. 

En suite, on fait `dhcp apply` pour appliquer la nouvelle configuration puis `dhclient -v`.

On restart `isc-dhcp-server` avec `systemctl restart isc-dhcp-server`.d

Le client possède donc l'ip `192.168.100.20/24` :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024230722745159740/unknown.png">


## Exercice 4 - Donner un accès à Internet au client

On configure les `iptables` du serveur : 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1024235301872341033/unknown.png">

Puis, on essaye de ping 1.1.1.1 (google) depuis le client en passant par notre serveur dhcp. 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026408053563740160/unknown.png">

## Exercice 5 - Installation du serveur DNS

On installe `bind9` avec apt, et on vérifie qu'il est est bien installé avec `named -v` : 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026408875081089104/unknown.png">

On edit le fichier de configuration `/etc/bind/named.conf.options.` comme ce qui suit :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026410026224590929/unknown.png">

Puis, on restart le service (avec systmctl). 

Ce quu permet depuis le client de ping `www.google.fr` : 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026410224246083644/unknown.png">

On installe Lynx avec `sudo apt-get lynx`.

On navigue sur Lynx qui est un navigateur dans le terminal :

<img 
src="https://cdn.discordapp.com/attachments/750759699942735884/1026411105410613288/unknown.png">

## Exercice 6 - Configuration du serveur DNS pour la zone tpadmin.local

On ajoute dans le fichier `/etc/bind/named.conf.local`, ce qui suit : 

<img src="https://media.discordapp.net/attachments/750759699942735884/1026413295739093023/unknown.png">

On fait un mv de `db.local` vers `db.tpadmin.local`. 

On édite le nouveau fichier, comme ceci : 

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026413975749005342/unknown.png">

On ajoute à la fin du fichier `named.conf.local`, ce qui suit :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026415731295924254/unknown.png">

Ensuite on fait `sudo mv db.127 db.192.168.100`.

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026417231099334656/unknown.png">

Donc, peut faire les commandes demandées afin de vérfier la configuration suivante :

<img src="https://cdn.discordapp.com/attachments/750759699942735884/1026417826887634954/unknown.png">

On est voit bien OK, le dns est bien configuré.




