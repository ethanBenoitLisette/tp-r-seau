TP1 - Premier pas réseau
Le terme réseau désigne au sens large toutes les fonctionnalités d'un PC permettant de se connecter à d'autres machines.
Le protocole IP est très important, il est central dans l'utilisation du réseau moderne.

On va voir IPv4 en cours, il existe aussi IPv6, plus récent, qui fonctionne sur les mêmes principes. Nous en parlerons aussi en cours.


Lorsque l'on parle de réseau, on désigne souvent par le terme client tout équipement qui porte une adresse IP.
Donc vos PCs sont des clients, et on va explorer leur réseau dans ce TP.


Sommaire

TP1 - Premier pas réseau
Sommaire
Déroulement et rendu du TP

I. Exploration locale en solo


1. Affichage d'informations sur la pile TCP/IP locale

En ligne de commande
En graphique (GUI : Graphical User Interface)



2. Modifications des informations

A. Modification d'adresse IP (part 1)





II. Exploration locale en duo

1. Prérequis
2. Câblage
Création du réseau (oupa)
3. Modification d'adresse IP
4. Utilisation d'un des deux comme gateway
5. Petit chat privé
6. Firewall



III. Manipulations d'autres outils/protocoles côté client

1. DHCP
2. DNS


IV. Wireshark
Bilan


Déroulement et rendu du TP

Groupe de 2 jusqu'à 4 personnes. Il faut au moins deux PCs avec une prise RJ45 (Ethernet) par groupe
Un câble RJ45 (fourni) pour connecter les deux PCs

Un compte-rendu par personne

vu que vous travaillez en groupe, aucun problème pour copier/coller les parties à faire à plusieurs (tout le II.)
une bonne partie est à faire de façon individuelle malgré tout (tout le I. et le III.)


Le rendu doit :

comporter des réponses aux questions explicites
comporter la marche à suivre pour réaliser les étapes demandées :

en ligne de commande, copier/coller des commandes et leurs résultat : JE NE VEUX AUCUN SCREEN DE LIGNE DE COMMANDE

en interface graphique, screenshots ou nom des menus où cliquer (sinon ça peut vite faire 1000 screenshots)


par exemple, pour la partie 1.A. je veux le la commande tapée et le résultat
de façon générale, tout ce que vous faites et qui fait partie du TP, vous me le mettez :)



⚠️ ⚠️ Désactivez votre firewall pour ce TP. ⚠️ ⚠️

I. Exploration locale en solo

1. Affichage d'informations sur la pile TCP/IP locale

En ligne de commande
En utilisant la ligne de commande (CLI) de votre OS :
🌞 Affichez les infos des cartes réseau de votre PC

```
Carte réseau sans fil Wi-Fi :

   Adresse physique . . . . . . . . . . . : 7C-B5-66-B2-82-
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.168(préféré)
```
```
Carte Ethernet Ethernet :

   Adresse physique . . . . . . . . . . . : 9C-2D-CD-5B-40-5F
   ```

🌞 Affichez votre gateway

 ```
Carte réseau sans fil Wi-Fi :

   Passerelle par défaut. . . . . . . . . : 10.33.19.254
```

🌞 Déterminer la MAC de la passerelle

```Interface : 10.33.16.168 --- 0xa
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique

```

🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)

```
Suffixe DNS propre à la connexion: 
Adresse physique: ‎7C-B5-66-B2-82-0C
Adresse IPv4: 10.33.16.168
Passerelle par défaut IPv4: 10.33.19.254
```
2. Modifications des informations

A. Modification d'adresse IP (part 1)
🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP :

![](https://i.imgur.com/UKES657.png)



🌞 Il est possible que vous perdiez l'accès internet. Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.

*Ci l'adrèsse est la mème alors l'une des deux seras privilègier par rapport à l'autre qui ne recevra rien donc pas d'internet*




II. Exploration locale en duo
Owkay. Vous savez à ce stade :

afficher les informations IP de votre machine
modifier les informations IP de votre machine
c'est un premier pas vers la maîtrise de votre outil de travail

On va maintenant répéter un peu ces opérations, mais en créant un réseau local de toutes pièces : entre deux PCs connectés avec un câble RJ45.

1. Prérequis

deux PCs avec ports RJ45
un câble RJ45

firewalls désactivés sur les deux PCs


2. Câblage
Ok c'est la partie tendue. Prenez un câble. Branchez-le des deux côtés. Bap.

Création du réseau (oupa)
Cette étape pourrait paraître cruciale. En réalité, elle n'existe pas à proprement parlé. On ne peut pas "créer" un réseau.
Si une machine possède une carte réseau, et si cette carte réseau porte une adresse IP, alors cette adresse IP se trouve dans un réseau (l'adresse de réseau). Ainsi, le réseau existe. De fait.
Donc il suffit juste de définir une adresse IP sur une carte réseau pour que le réseau existe ! Bap.

3. Modification d'adresse IP
🌞 Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau
panneau de configuration > Réseau et Internet > Centre Réseau et partage > Etats de Ethernet > Propriétés de Ethernet > Protocole Internet version 4(TCP/IPv4)


- Si vos PCs ont un port RJ45 alors y'a une carte réseau Ethernet associée
- choisissez une IP qui commence par "10.10.10."
  - /24 pour la longueur de masque, ou 255.255.255.0 pour le masque (suivant les OS, l'info est demandée différement, mais c'est la même chose)

🌞 Vérifier à l'aide d'une commande que votre IP a bien été changée
```
C:\Users>ipconfig
Carte Ethernet Ethernet :

   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.2
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
```

🌞 Vérifier que les deux machines se joignent

```
C:\Users>ping 10.10.10.1

Envoi d’une requête 'Ping'  10.10.10.1 avec 32 octets de données :
Réponse de 10.10.10.1 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.1 : octets=32 temps=2 ms TTL=128
Réponse de 10.10.10.1 : octets=32 temps=3 ms TTL=128
Réponse de 10.10.10.1 : octets=32 temps=3 ms TTL=128

Statistiques Ping pour 10.10.10.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 2ms, Maximum = 3ms, Moyenne = 2ms
```

- utilisez la commande ping pour tester la connectivité entre les deux machines

La commande ping est un message simple envoyé à une autre machine. Cette autre machine retournera alors un message tout aussi simple. ping utilise un protocole frère de IP : le protocole ICMP. On mesure souvent la latence réseau grâce à un ping : en mesurant la durée entre l'émission du ping et la réception du retour.

🌞 Déterminer l'adresse MAC de votre correspondant
```
C:\Users>arp -a
Interface : 10.10.10.1 --- 0x5
  Adresse Internet      Adresse physique      Type
  10.10.10.2            9c-2d-cd-5b-40-5f     dynamique
```

4. Utilisation d'un des deux comme gateway
Ca, ça peut toujours dépann irl. Comme pour donner internet à une tour sans WiFi quand y'a un PC portable à côté, par exemple.
L'idée est la suivante :

vos PCs ont deux cartes avec des adresses IP actuellement

la carte WiFi, elle permet notamment d'aller sur internet, grâce au réseau YNOV
la carte Ethernet, qui permet actuellement de joindre votre coéquipier, grâce au réseau que vous avez créé :)


si on fait un tit schéma tout moche, ça donne ça :


  Internet           Internet
     |                   |
    WiFi                WiFi
     |                   |
    PC 1 ---Ethernet--- PC 2
    
- internet joignable en direct par le PC 1
- internet joignable en direct par le PC 2



vous allez désactiver Internet sur une des deux machines, et vous servir de l'autre machine pour accéder à internet.


  Internet           Internet
     X                   |
     X                  WiFi
     |                   |
    PC 1 ---Ethernet--- PC 2
    
- internet joignable en direct par le PC 2
- internet joignable par le PC 1, en passant par le PC 2



pour ce faiiiiiire :

désactivez l'interface WiFi sur l'un des deux postes
s'assurer de la bonne connectivité entre les deux PCs à travers le câble RJ45

sur le PC qui n'a plus internet

sur la carte Ethernet, définir comme passerelle l'adresse IP de l'autre PC



sur le PC qui a toujours internet

sur Windows, il y a une option faite exprès (google it. "share internet connection windows 10" par exemple)
sur GNU/Linux, faites le en ligne de commande ou utilisez Network Manager (souvent présent sur tous les GNU/Linux communs)
sur MacOS : toute façon vous avez pas de ports RJ, si ? :o (google it sinon)


🌞Tester l'accès internet

```Réponse de 216.58.213.142 : octets=32 temps=22 ms TTL=248
Réponse de 216.58.213.142 : octets=32 temps=22 ms TTL=248
Réponse de 216.58.213.142 : octets=32 temps=22 ms TTL=248
Réponse de 216.58.213.142 : octets=32 temps=21 ms TTL=248
```

🌞 Prouver que la connexion Internet passe bien par l'autre PC
```
C:\Users\Ethan>tracert 1.1.1.1

Détermination de l’itinéraire vers one.one.one.one [1.1.1.1]
avec un maximum de 30 sauts :

  1     1 ms     1 ms     1 ms  SeigneurHugoPCMasterRace [192.168.137.1]
  2     *        *        *     Délai d’attente de la demande dépassé.
  3     6 ms     5 ms     3 ms  10.33.19.254
 12    25 ms    26 ms    23 ms  one.one.one.one [1.1.1.1]

Itinéraire déterminé.
```


5. Petit chat privé

On va créer un chat extrêmement simpliste à l'aide de netcat (abrégé nc). Il est souvent considéré comme un bon couteau-suisse quand il s'agit de faire des choses avec le réseau.
Sous GNU/Linux et MacOS vous l'avez sûrement déjà, sinon débrouillez-vous pour l'installer :). Les Windowsien, ça se passe ici (from https://eternallybored.org/misc/netcat/).
Une fois en possession de netcat, vous allez pouvoir l'utiliser en ligne de commande. Comme beaucoup de commandes sous GNU/Linux, Mac et Windows, on peut utiliser l'option -h (h pour help) pour avoir une aide sur comment utiliser la commande.
Sur un Windows, ça donne un truc comme ça :

C:\Users\It4\Desktop\netcat-win32-1.11>nc.exe -h
[v1.11 NT www.vulnwatch.org/netcat/]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [options] [hostname] [port]
options:
        -d              detach from console, background mode

        -e prog         inbound program to exec [dangerous!!]
        -g gateway      source-routing hop point[s], up to 8
        -G num          source-routing pointer: 4, 8, 12, ...
        -h              this cruft
        -i secs         delay interval for lines sent, ports scanned
        -l              listen mode, for inbound connects
        -L              listen harder, re-listen on socket close
        -n              numeric-only IP addresses, no DNS
        -o file         hex dump of traffic
        -p port         local port number
        -r              randomize local and remote ports
        -s addr         local source address
        -t              answer TELNET negotiation
        -u              UDP mode
        -v              verbose [use twice to be more verbose]
        -w secs         timeout for connects and final net reads
        -z              zero-I/O mode [used for scanning]
port numbers can be individual or ranges: m-n [inclusive]


L'idée ici est la suivante :

l'un de vous jouera le rôle d'un serveur

l'autre sera le client qui se connecte au serveur


Précisément, on va dire à netcat d'écouter sur un port. Des ports, y'en a un nombre fixe (65536, on verra ça plus tard), et c'est juste le numéro de la porte à laquelle taper si on veut communiquer avec le serveur.
Si le serveur écoute à la porte 20000, alors le client doit demander une connexion en tapant à la porte numéro 20000, simple non ?
Here we go :
🌞 sur le PC serveur avec par exemple l'IP 192.168.1.1
```
C:\Users\hugoa\netcat-1.11> .\nc.exe -l -p 8888
bonjour
bonjour
ça va ou quoi
c'est simple
nice & easy
ez

```

🌞 sur le PC client avec par exemple l'IP 192.168.1.2
```
PS C:\Users\Ethan\netcat-1.11> .\nc.exe 192.168.137.1 8888




s fd
tyo
yoyoy*
,,n,n,n
uuuuuuuuuuuuuuuuuuuuuuuuuuuiiiiiiiiiiiiiiiiiiiiiii
bonjour
bonjour
ça va ou quoi
c'est simple
nice & easy
ez
```
🌞 Visualiser la connexion en cours
```
C:\Windows\system32> netstat -a -n -b

  TCP    192.168.137.1:8888     192.168.137.2:59814    ESTABLISHED
 [nc.exe]
 ```
```
C:\Windows\system32> netstat -a -n -b
  TCP    192.168.137.2:59814    192.168.137.1:8888     ESTABLISHED
 [nc.exe]
 ```

🌞 Pour aller un peu plus loin
```
C:\Windows\system32> netstat -a -n -b | Select-String 8888

  TCP    0.0.0.0:8888           0.0.0.0:0              LISTENING
```


6. Firewall
Toujours par 2.
Le but est de configurer votre firewall plutôt que de le désactiver
🌞 Activez et configurez votre firewall

```
PS C:\Windows\system32> netsh advfirewall firewall add rule name="ICMP Allow incoming V4 echo request" protocol=icmpv4:8,any dir=in action=allow
Ok.
```
```
PS C:\Users\Ethan\netcat-1.11> ping 192.168.137.1

Envoi d’une requête 'Ping'  192.168.137.1 avec 32 octets de données :
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=4 ms TTL=128

```

autoriser le traffic sur le port qu'utilise nc

```
PS C:\Users\Ethan\netcat-1.11> .\nc.exe 192.168.137.1 8888
gvyuhujcvesjhcghuezsbch_duygv xhydu
 zfgdfg
gdfg
fgfg
ddssqerf
h
```


III. Manipulations d'autres outils/protocoles côté client

1. DHCP
Bon ok vous savez définir des IPs à la main. Mais pour être dans le réseau YNOV, vous l'avez jamais fait.
C'est le serveur DHCP d'YNOV qui vous a donné une IP.
Une fois que le serveur DHCP vous a donné une IP, vous enregistrer un fichier appelé bail DHCP qui contient, entre autres :

l'IP qu'on vous a donné
le réseau dans lequel cette IP est valable

🌞Exploration du DHCP, depuis votre PC
```
  Bail obtenu. . . . . . . . . . . . . . : mardi 4 octobre 2022 15:24:44
   Bail expirant. . . . . . . . . . . . . : mardi 4 octobre 2022 16:58:25
      Serveur DHCP . . . . . . . . . . . . . : 192.168.86.254
   ```
   

2. DNS
Le protocole DNS permet la résolution de noms de domaine vers des adresses IP. Ce protocole permet d'aller sur google.com plutôt que de devoir connaître et utiliser l'adresse IP du serveur de Google.
Un serveur DNS est un serveur à qui l'on peut poser des questions (= effectuer des requêtes) sur un nom de domaine comme google.com, afin d'obtenir les adresses IP liées au nom de domaine.
Si votre navigateur fonctionne "normalement" (il vous permet d'aller sur google.com par exemple) alors votre ordinateur connaît forcément l'adresse d'un serveur DNS. Et quand vous naviguez sur internet, il effectue toutes les requêtes DNS à votre place, de façon automatique.
```
Carte réseau sans fil Wi-Fi :
   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
                                       1.1.1.1r
```
                                       
🌞 Utiliser, en ligne de commande l'outil nslookup (Windows, MacOS) ou dig (GNU/Linux, MacOS) pour faire des requêtes DNS à la main

faites un lookup (lookup = "dis moi à quelle IP se trouve tel nom de domaine")

pour google.com

pour ynov.com

interpréter les résultats de ces commandes


déterminer l'adresse IP du serveur à qui vous venez d'effectuer ces requêtes
faites un reverse lookup (= "dis moi si tu connais un nom de domaine pour telle IP")

pour l'adresse 231.34.113.12

pour l'adresse 78.34.2.17

interpréter les résultats
si vous vous demandez, j'ai pris des adresses random :)




IV. Wireshark

1. Intro Wireshark
Wireshark est un outil qui permet de visualiser toutes les trames qui sortent et entrent d'une carte réseau.
On appelle ça un sniffer, ou analyseur de trames.

Il peut :

enregistrer le trafic réseau, pour l'analyser plus tard
afficher le trafic réseau en temps réel

On peut TOUT voir.
Un peu austère aux premiers abords, une manipulation très basique permet d'avoir une très bonne compréhension de ce qu'il se passe réellement.
➜ Téléchargez l'outil Wireshark.
🌞 Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :

un ping entre vous et votre passerelle
un netcat entre vous et votre mate, branché en RJ45
une requête DNS. Identifiez dans la capture le serveur DNS à qui vous posez la question.
prenez moi des screens des trames en question
on va prendre l'habitude d'utiliser Wireshark souvent dans les cours, pour visualiser ce qu'il se passe


2. Bonus : avant-goût TCP et UDP
TCP et UDP ce sont les deux protocoles qui utilisent des ports. Si on veut accéder à un service, sur un serveur, comme un site web :

il faut pouvoir joindre en terme d'IP le correspondant

on teste que ça fonctionne avec un ping généralement


il faut que le serveur fasse tourner un programme qu'on appelle "service" ou "serveur"

le service "écoute" sur un port TCP ou UDP : il attend la connexion d'un client
comme vous avez fait avec netcat !
sauf qu'un netcat pourri, on peut par exemple faire tourner un site web
et donc plutôt qu'un autre netcat pourri, il faudra utiliser un navigateur web pour s'y connecter


le client connaît par avance le port TCP ou UDP sur lequel le service écoute

en utilisant l'IP et le port, il peut se connecter au service en utilisant un moyen adapté :

un navigateur web pour un site web
un ncat pour se connecter à un autre ncat

et plein d'autres, de façon générale on parle d'un client, et d'un serveur





🌞 Wireshark it

déterminez à quelle IP et quel port votre PC se connecte quand vous regardez une vidéo Youtube

il sera sûrement plus simple de repérer le trafic Youtube en fermant tous les autres onglets et toutes les autres applications utilisant du réseau




Bilan
Vu pendant le TP :

visualisation de vos interfaces réseau (en GUI et en CLI)
extraction des informations IP

adresse IP et masque
calcul autour de IP : adresse de réseau, etc.


connaissances autour de/aperçu de :

un outil de diagnostic simple : ping

un outil de scan réseau : nmap

un outil qui permet d'établir des connexions "simples" (on y reviendra) : netcat

un outil pour faire des requêtes DNS : nslookup ou dig

un outil d'analyse de trafic : wireshark



manipulation simple de vos firewalls

Conclusion :

Pour permettre à un ordinateur d'être connecté en réseau, il lui faut une liaison physique (par câble ou par WiFi).
Pour réceptionner ce lien physique, l'ordinateur a besoin d'une carte réseau. La carte réseau porte une adresse MAC

Pour être membre d'un réseau particulier, une carte réseau peut porter une adresse IP.
Si deux ordinateurs reliés physiquement possèdent une adresse IP dans le même réseau, alors ils peuvent communiquer.

Un ordintateur qui possède plusieurs cartes réseau peut réceptionner du trafic sur l'une d'entre elles, et le balancer sur l'autre, servant ainsi de "pivot". Cet ordinateur est appelé routeur.
Il existe dans la plupart des réseaux, certains équipements ayant un rôle particulier :

un équipement appelé passerelle. C'est un routeur, et il nous permet de sortir du réseau actuel, pour en joindre un autre, comme Internet par exemple
un équipement qui agit comme serveur DNS : il nous permet de connaître les IP derrière des noms de domaine
un équipement qui agit comme serveur DHCP : il donne automatiquement des IP aux clients qui rejoigne le réseau
chez vous, c'est votre Box qui fait les trois :)