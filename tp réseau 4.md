TP4 : TCP, UDP et services réseau
Dans ce TP on va explorer un peu les protocoles TCP et UDP.
La première partie est détente, vous explorez TCP et UDP un peu, en vous servant de votre PC.
La seconde partie se déroule en environnement virtuel, avec des VMs. Les VMs vont nous permettre en place des services réseau, qui reposent sur TCP et UDP.
Le but est donc de commencer à mettre les mains de plus en plus du côté administration, et pas simple client.
Dans cette seconde partie, vous étudierez donc :

le protocole SSH (contrôle de machine à distance)
le protocole DNS (résolution de noms)

essentiel au fonctionnement des réseaux modernes





Sommaire

TP4 : TCP, UDP et services réseau
Sommaire
0. Prérequis
I. First steps

II. Mise en place

1. SSH
2. Routage



III. DNS

1. Présentation
2. Setup
3. Test




0. Prérequis
➜ Pour ce TP, on va se servir de VMs Rocky Linux. On va en faire plusieurs, n'hésitez pas à diminuer la RAM (512Mo ou 1Go devraient suffire). Vous pouvez redescendre la mémoire vidéo aussi.
➜ Si vous voyez un 🦈 c'est qu'il y a un PCAP à produire et à mettre dans votre dépôt git de rendu
➜ L'emoji 🖥️ indique une VM à créer. Pour chaque VM, vous déroulerez la checklist suivante :


 Créer la machine (avec une carte host-only)

 Définir une IP statique à la VM

 Donner un hostname à la machine

 Vérifier que l'accès SSH fonctionnel

 Vérifier que le firewall est actif

 Remplir votre fichier hosts, celui de votre PC, pour accéder au VM avec un nom

 Dès que le routeur est en place, n'oubliez pas d'ajouter une route par défaut aux autres VM pour qu'elles aient internet


Toutes les commandes pour réaliser ces opérations sont dans le mémo Rocky. Aucune de ces étapes ne doit figurer dan le rendu, c'est juste la mise en place de votre environnement de travail.


I. First steps
```
Spotify :

- ip dest 142.250.179.98 

- source 443

- destination 58472

```
![trame wirechark spotify](pics/analyse_spotify.pcapng)

```
Chrome: 

- ip dest 77.111.247.15 

- source 443

- destination 58472

```
![trame wirechark opera](pics/analyse_opera.pcapng)
```
overwatch :

- ip dest 142.250.179.98 

- source 443

- destination 58472

```
![trame wirechark overwach](pics/analyse_overwach.pcapng)

```
teams: 

- ip dest 77.111.247.15 

- source 443

- destination 58472

```
![trame wirechark teams](pics/analyse_teams.pcapng)
```
Magic :

- ip dest 142.250.179.98 

- source 443

- destination 58472

```
![trame wirechark magic ](pics/analyse_magic.pcapng)

II. Mise en place

1. SSH
````
PS C:\Windows\system32> netstat -b -p tcp

Connexions actives

  Proto  Adresse locale         Adresse distante       État
  TCP    10.4.1.1:54798         10.4.1.11:ssh          ESTABLISHED
 [ssh.exe]
  TCP    10.33.16.195:53446     162.159.134.234:https  ESTABLISHED
  ````
  ![trame wirechark ssh ](pics/analyse_ssh.pcapng)
1. Routage
Ouais, un peu de répétition, ça fait jamais de mal. On va créer une machine qui sera notre routeur, et permettra à toutes les autres machines du réseau d'avoir Internet.
🖥️ Machine router.tp4.b1

n'oubliez pas de dérouler la checklist (voir les prérequis du TP)
donnez lui l'adresse IP 10.4.1.11/24 sur sa carte host-only
ajoutez-lui une carte NAT, qui permettra de donner Internet aux autres machines du réseau
référez-vous au TP précédent


Rien à remettre dans le compte-rendu pour cette partie.


III. DNS

1. Présentation
Un serveur DNS est un serveur qui est capable de répondre à des requêtes DNS.
Une requête DNS est la requête effectuée par une machine lorsqu'elle souhaite connaître l'adresse IP d'une machine, lorsqu'elle connaît son nom.
Par exemple, si vous ouvrez un navigateur web et saisissez https://www.google.com alors une requête DNS est automatiquement effectuée par votre PC pour déterminez à quelle adresse IP correspond le nom www.google.com.

La partie https:// ne fait pas partie du nom de domaine, ça indique simplement au navigateur la méthode de connexion. Ici, c'est HTTPS.

Dans cette partie, on va monter une VM qui porte un serveur DNS. Ce dernier répondra aux autres VMs du LAN quand elles auront besoin de connaître des noms. Ainsi, ce serveur pourra :

résoudre des noms locaux

vous pourrez ping node1.tp4.b1 et ça fonctionnera
mais aussi ping www.google.com et votre serveur DNS sera capable de le résoudre aussi



Dans la vraie vie, il n'est pas rare qu'une entreprise gère elle-même ses noms de domaine, voire gère elle-même son serveur DNS. C'est donc du savoir ré-utilisable pour tous qu'on voit ici.

En réalité, ce n'est pas votre serveur DNS qui pourra résoudre www.google.com, mais il sera capable de forward (faire passer) votre requête à un autre serveur DNS qui lui, connaît la réponse.



2. Setup
🖥️ Machine dns-server.tp4.b1

n'oubliez pas de dérouler la checklist (voir les prérequis du TP)
donnez lui l'adresse IP 10.4.1.201/24


Installation du serveur DNS :

# assurez-vous que votre machine est à jour
$ sudo dnf update -y

# installation du serveur DNS, son p'tit nom c'est BIND9
$ sudo dnf install -y bind bind-utils


La configuration du serveur DNS va se faire dans 3 fichiers essentiellement :


un fichier de configuration principal

/etc/named.conf
on définit les trucs généraux, comme les adresses IP et le port où on veu écouter
on définit aussi un chemin vers les autres fichiers, les fichiers de zone



un fichier de zone

/var/named/tp4.b1.db
je vous préviens, la syntaxe fait mal
on peut y définir des correspondances IP ---> nom




un fichier de zone inverse

/var/named/tp4.b1.rev
on peut y définir des correspondances nom ---> IP




➜ Allooooons-y, fichier de conf principal

# éditez le fichier de config principal pour qu'il ressemble à :
$ sudo cat /etc/named.conf
options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
[...]
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        recursion yes;
[...]
# référence vers notre fichier de zone
zone "tp4.b1" IN {
     type master;
     file "tp4.b1.db";
     allow-update { none; };
     allow-query {any; };
};
# référence vers notre fichier de zone inverse
zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp4.b1.rev";
     allow-update { none; };
     allow-query { any; };
};


➜ Et pour les fichiers de zone

# Fichier de zone pour nom -> IP

$ sudo cat /var/named/tp4.b1.db

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns-server IN A 10.4.1.201
node1      IN A 10.4.1.11



# Fichier de zone inverse pour IP -> nom

$ sudo cat /var/named/tp4.b1.rev

$TTL 86400
@ IN SOA dns-server.tp4.b1. admin.tp4.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns-server.tp4.b1.

;Reverse lookup for Name Server
201 IN PTR dns-server.tp4.b1.
11 IN PTR node1.tp4.b1.


➜ Une fois ces 3 fichiers en place, démarrez le service DNS

# Démarrez le service tout de suite
$ sudo systemctl start named

# Faire en sorte que le service démarre tout seul quand la VM s'allume
$ sudo systemctl enable named

# Obtenir des infos sur le service
$ sudo systemctl status named

# Obtenir des logs en cas de probème
$ sudo journalctl -xe -u named


🌞 Dans le rendu, je veux

un cat des fichiers de conf
un systemctl status named qui prouve que le service tourne bien
une commande ss qui prouve que le service écoute bien sur un port

🌞 Ouvrez le bon port dans le firewall

grâce à la commande ss vous devrez avoir repéré sur quel port tourne le service

vous l'avez écrit dans la conf aussi toute façon :)


ouvrez ce port dans le firewall de la machine dns-server.tp4.b1 (voir le mémo réseau Rocky)


3. Test
🌞 Sur la machine node1.tp4.b1

configurez la machine pour qu'elle utilise votre serveur DNS quand elle a besoin de résoudre des noms
assurez vous que vous pouvez :

résoudre des noms comme node1.tp4.b1 et dns-server.tp4.b1

mais aussi des noms comme www.google.com




🌞 Sur votre PC

utilisez une commande pour résoudre le nom node1.tp4.b1 en utilisant 10.4.1.201 comme serveur DNS


Le fait que votre serveur DNS puisse résoudre un nom comme www.google.com, ça s'appelle la récursivité et c'est activé avec la ligne recursion yes; dans le fichier de conf.

🦈 Capture d'une requête DNS vers le nom node1.tp4.b1 ainsi que la réponse