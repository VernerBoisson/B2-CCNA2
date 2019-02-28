# TP 1 - Remise dans le bain

[Enoncé du TP](https://github.com/It4lik/B2-Reseau-2018/tree/master/tp/1)

## I. Exploration du réseau d'une machine CentOS

### 1. Mise en place

#### Configuration de VirtualBox

Création de net1 : 10.1.1.0/24
* Combien y a-t-il d'adresses disponibles dans un /24 ?

Il y a 254 adresses mais la première 10.1.1.0 qui est l'adresse réseau et la dernière 10.1.1.255 le broadcast sont réservées donc 252 sont disponibles.

* Combien y a-t-il d'adresses disponibles dans un /30 ?

De la même façon, il y'en a 2 adresses disponibles.

* Quelle est l'utilité d'un /30 ?

L'utilité de n'avoir que 2 adresses disponibles permet de créer un LAN avec uniquement un client et un serveur.

#### Allumage et configuration de la VM

S'assurer que les 3 cartes réseaux fonctionnent  
NAT:  
`[root@vm1 ~]# ping google.com`  
`PING google.com (216.58.204.110) 56(84) bytes of data.
64 bytes from par10s28-in-f110.1e100.net (216.58.204.110): icmp_seq=1 ttl=63 time=179 ms`

net1 :  
`[root@vm1 ~]# ping interface1`  
`PING interface1 (10.1.1.1) 56(84) bytes of data.
64 bytes from interface1 (10.1.1.1): icmp_seq=1 ttl=64 time=0.160 ms`

net2 :  
`[root@vm1 ~]# ping interface2`  
`PING interface2 (10.1.2.1) 56(84) bytes of data.
64 bytes from interface2 (10.1.2.1): icmp_seq=1 ttl=64 time=0.160 ms
`

### 2. Basics

#### Routes

`default via 10.0.2.2 dev enp0s3 proto dhcp metric 100 `  
Pour joindre n'importe quel réseau, il faut passer par la passerelle ici 10.0.2.2 et il faut utiliser l'interface réseau enp0s3.

`10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100`  
Pour joindre le réseau 10.0.2.0, il faut utilisé l'interface enp0s3.

`10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 101`  
Pour joindre le réseau 10.1.1.0, il faut utilisé l'interface enp0s8.

`10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 102`  
Pour joindre le réseau 10.1.2.0, il faut utilisé l'interface enp0s9.

Pour supprime la route par défaut :   
`[root@vm1 ~]# ip route delete default`  
`[root@vm1 ~]# ping google.com`  
`ping: google.com: Nom ou service inconnu`

Pour la remettre et tester que ça fonctionne de nouveau :  
`[root@vm1 ~]# ip route add default via 10.0.2.2 dev enp0s3`  
`[root@vm1 ~]# ping google.com`  
`PING google.com (216.58.204.110) 56(84) bytes of data.64 bytes from par10s28-in-f110.1e100.net (216.58.204.110): icmp_seq=1 ttl=63 time=48.3 ms`

#### Tables ARP


`[root@vm1 ~]# ip neigh show`  
`10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:01 REACHABLE`  
`10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE`

Le voisin `10.1.1.1` qui peut être contacter par l'interface réseau `enp0s8` a pour adresse MAC `0a:00:27:00:00:01`.
Le voisin `10.0.2.2` qui peut être contacter par l'interface réseau `enp0s3` a pour adresse MAC `52:54:00:12:35:02`.

`10.1.2.1 dev enp0s9 lladdr 0a:00:27:00:00:02 REACHABLE`.

Le voisin `10.1.2.1` qui peut être contacter par l'interface réseau `enp0s9` a pour adresse MAC `0a:00:27:00:00:02`.

L'état "NUD" `REACHABLE` signifie que le voisin est directement joignable.

#### Capture réseau

La machine (10.1.1.2) envoie une requête ARP à Broadcast en demandant qui est 10.1.1.1  
Broadcast lui donne l'adresse MAC correspondant à 10.1.1.1 (ici : 0a:00:27:00:00:01)  
Une fois l'adresse MAC dans la table ARP
10.1.1.2 envoie 4 PING à 10.1.1.1 et en réponse 10.1.1.1 envoie 4 PING en retour.

Si l'adresse MAC est déjà connu dans la table ARP, la requête à Broadcast et sa réponse n'ont pas lieu.

## II. Communication simple entre deux machines

### 2. Basics

#### ping et ARP

Dans l'analyse Wireshark, il y a des echanges ARP supplémentaires.

#### netcat
#### UDP

Dans Wireshark, la data est envoyé d'une adresse ip et un port à une adresse ip et un port. Contrairement aux ping précédent, il n'y a pas d'accusé de reception.

#### TCP

Dans Wireshark, on peut constater que lors de la création de la connexion TCP, il y a un "3-way handshake", le client envoie un 'SYN' ensuite le serveur envoie un 'ACK-SYN' et enfin le client envoie un 'ACK'. contrairement à l'UDP, à chaque message envoyés, il y a un accusé de reception : la source envoie 'PSH, ACK' et le destinataire envoie 'ACK'.

L'UDP permet un envoie plus rapide de message car il n'y a pas de retour mais cela signifie qu'on peut perdre des paquets, le TCP a un envoie de message plus lent car il attend un accusé de reception à chaque message mais garanti que les messages sont bien envoyés.

#### Firewall

`Destination unreachable (Host administratively prohibited)`

## III. Routage statique simple

`[root@client2 ~]# ping 10.1.2.1`  
`PING 10.1.2.1 (10.1.2.1) 56(84) bytes of data.`    
`64 bytes from 10.1.2.1: icmp_seq=1 ttl=63 time=0.589 ms`

`[root@client2 ~]# traceroute 10.1.2.1`  
`traceroute to 10.1.2.1 (10.1.2.1), 30 hops max, 60 byte packets`  
` 1  gateway (10.0.2.2)  0.250 ms  0.131 ms  0.119 ms`  
` 2  10.1.2.1 (10.1.2.1)  0.415 ms  0.521 ms  0.455 ms`
