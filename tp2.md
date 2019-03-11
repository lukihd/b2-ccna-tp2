# TP 2 - Routage statique et services d'infrastructure

# I. Mise en place du lab

## 1. Création des VMs et adressage IP

## 2. Routage statique

### Ping du client1 vers server1

```bash
[lukihd@client1 ~]$ ping 10.2.2.10
PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
64 bytes from 10.2.2.10: icmp_seq=1 ttl=62 time=1.65 ms
64 bytes from 10.2.2.10: icmp_seq=2 ttl=62 time=1.21 ms
64 bytes from 10.2.2.10: icmp_seq=3 ttl=62 time=1.25 ms
64 bytes from 10.2.2.10: icmp_seq=4 ttl=62 time=1.07 ms
--- 10.2.2.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3011ms
rtt min/avg/max/mdev = 1.074/1.299/1.654/0.218 ms
```

### Pourquoi le ping de router1 vers server1 ne fonctionne pas ?

Le ping depuis le router1 vers server1 ne fonctionne pas tout simplement car la route n'existe pas dans l'autre sens, le paquet provenant de server1 ne peut donc pas être reçu par router1.

De la même façon quand router2 essai de ping client1 il ne peut pas pour la même raison, la route n'existe pas dans le sens du retour. 

## 3. Visualisation du routage avec Wireshark

### Ping Wireshark sur 10.2.2.254

![](https://raw.githubusercontent.com/lukihd/b2-net-tp1/master/images/ping_net_2.PNG)
### Ping Wireshark sur 10.2.12.3

![](https://raw.githubusercontent.com/lukihd/b2-net-tp1/master/images/ping_net_12.PNG)

### Expliquer la différences

La différence réside dans l'adresse mac renvoyé par les cartes.

# II. NAT et services d'infra

## 1. Mise en place du NAT

### Vérification des routes pour chaque VM

Router 1 :
```bash
[lukihd@router1 ~]$ curl google.fr
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.fr/">here</A>.
</BODY></HTML>
```

Router 2 :
```bash
[lukihd@router2 ~]$ curl google.fr
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.fr/">here</A>.
</BODY></HTML>
```

Client 1 :
```bash
[lukihd@client1 ~]$ curl google.fr
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.fr/">here</A>.
</BODY></HTML>
```

Server 1 :
```bash
[lukihd@server1 ~]$ curl google.fr
curl: (6) Could not resolve host: google.fr; Unknown error
```

Le serveur 1 n'est donc pas capable d'accéder à internet car il n'as pas de route vers la NAT de Router 1.

## 2. DHCP Server

### Résultat de l'adressage dynamique pour Client 2

```bash
# On fait la commande pour attribuer une adresse par le dhcp
[lukihd@client2 ~]$ sudo dhclient
# On vérifie si les routes sont à jour
[lukihd@client2 ~]$ ip r s
default via 10.2.1.254 dev ens33
10.2.1.0/24 dev ens33 proto kernel scope link src 10.2.1.50
10.2.2.0/24 via 10.2.1.254 dev ens33
169.254.0.0/16 dev ens33 scope link metric 1002
# On vérifie que l'adresse ip est bien attribuée
[lukihd@client2 ~]$ ip a
				[...]
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:c2:67:5d brd ff:ff:ff:ff:ff:ff
    inet 10.2.1.50/24 brd 10.2.1.255 scope global dynamic ens33
       valid_lft 459sec preferred_lft 459sec
    inet6 fe80::20c:29ff:fec2:675d/64 scope link
       valid_lft forever preferred_lft forever
```

## 3. NTP server

Crony n'étant pas installer, j'ai du le mettre sur chaque machine et créer une route de façon temporaire sur server 1. (On à donc tapé la commande au lieu de l'inscrire dans le fichier).

J'ai pas mis les commande parceque ce n'est pas intéressant. Mais ce qu'on peut remarquer c'est que les serveurs avec l'heure son horodaté avec 1h de retard alors que on à pris les serveurs en France.

Toutefois ils ont tous la même heure c'est le principal.

## 4. Web Server

Problème de ping à cause des restrictions (pare feu surrement) des différents routeur. En effet je peux ping de routeur 2 vers le serveur web mais pas depuis les autres points du réseau. Le site fonctionne il n'est juste pas accessible. Surement à cause des Zones. J'ai tenté de créer mes propres routes mais rien n'y change.
