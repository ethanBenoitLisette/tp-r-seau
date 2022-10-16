# TP3 : On va router des trucs


## I. ARP

Première partie simple, on va avoir besoin de 2 VMs.

| Machine  | `10.3.1.0/24` |
|----------|---------------|
| `john`   | `10.3.1.11`   |
| `marcel` | `10.3.1.12`   |

```schema
   john               marcel
  ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     │
  └─────┘    └───┘    └─────┘
```

> Référez-vous au [mémo Réseau Rocky](../../cours/memo/rocky_network.md) pour connaître les commandes nécessaire à la réalisation de cette partie.

### 1. Echange ARP

🌞**Générer des requêtes ARP**

- effectuer un `ping` d'une machine à l'autre

```
[root@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=1.25 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.856 ms
```
- observer les tables ARP des deux machines

Côté john
```
[root@localhost ~]$ ip neigh show dev enp0s6
10.3.1.12 lladdr 08:00:27:a8:58:c9 STALE
```

Côté marcel
```
[root@localhost ~]$ ip neigh show dev enp0s6
10.3.1.11 lladdr 08:00:27:25:d8:f5 STALE
```
- repérer l'adresse MAC de `john` dans la table ARP de `marcel` et vice-versa

Adresse MAC de `john` : 08:00:27:a8:58:c9

Adresse MAC de `marcel` : 08:00:27:25:d8:f5

- prouvez que l'info est correcte (que l'adresse MAC que vous voyez dans la table est bien celle de la machine correspondante)
  - une commande pour voir la MAC de `marcel` dans la table ARP de `john`
```
[root@localhost ~]$ ip neigh show 10.3.1.12
```
  - et une commande pour afficher la MAC de `marcel`, depuis `marcel`

```
[root@localhost ~]$ ip link show

2: enp0s6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000ot found
    link/ether 08:00:27:25:d8:f5 brd ff:ff:ff:ff:ff:ff
```

### 2. Analyse de trames

🌞**Analyse de trames**

- utilisez la commande `tcpdump` pour réaliser une capture de trame

```
[root@localhost ~]$ tcpdump -i enp0s6 -c 10 -w tp3_arp.pcap not port 22
```
- videz vos tables ARP, sur les deux machines, puis effectuez un `ping`

```
[root@localhost ~]$ ip neigh flush all
```

🦈 **Capture réseau `tp3_arp.pcapng`** qui contient un ARP request et un ARP reply

> **Si vous ne savez pas comment récupérer votre fichier `.pcapng`** sur votre hôte afin de l'ouvrir dans Wireshark, et me le livrer en rendu, demandez-moi.

## II. Routage

Vous aurez besoin de 3 VMs pour cette partie. **Réutilisez les deux VMs précédentes.**

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |

> Je les appelés `marcel` et `john` PASKON EN A MAR des noms nuls en réseau 🌻

```schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘
```

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

```
[root@localhost ~]$ firewall-cmd --add-masquerade --zone=public
success

[root@localhost ~]$ firewall-cmd --list-all
  masquerade: yes
```

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

- il faut ajouter une seule route des deux côtés

Côté `john`
```
[root@localhost ~]$ ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s6
```

Côté `marcel`
```
[root@localhost ~]$ ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s6
```
- une fois les routes en place, vérifiez avec un `ping` que les deux machines peuvent se joindre

```
[root@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=2.58 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.65 ms
```

```
[root@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=1.09 ms
```


### 2. Analyse de trames

🌞**Analyse des échanges ARP**

- videz les tables ARP des trois noeuds
- effectuez un `ping` de `john` vers `marcel`
- regardez les tables ARP des trois noeuds
- essayez de déduire un peu les échanges ARP qui ont eu lieu
- répétez l'opération précédente (vider les tables, puis `ping`), en lançant `tcpdump` sur `marcel`
- **écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames** utiles pour l'échange


| ordre | type trame  | IP source | MAC source                   | IP destination | MAC destination               |
|-------|-------------|-----------|------------------------------|----------------|-------------------------------|
| 1     | Requête ARP | x         | `router` `08:00:27:82:43:fc` | x              | Broadcast `ff:ff:ff:ff:ff:ff` |
| 2     | Réponse ARP | x         | `marcel` `08:00:27:25:d8:f5` | x              | `router` `08:00:27:82:43:fc`  |
| 3     | Ping        | 10.3.2.254| 08:00:27:82:43:fc            | 10.3.2.12      | 08:00:27:25:d8:f5             |
| 4     | Pong        | 10.3.2.12 | 08:00:27:25:d8:f5            | 10.3.2.254     | 08:00:27:82:43:fc             |
| 5     | Requête ARP | x         | `marcel` `08:00:27:25:d8:f5` | x              | `router` `08:00:27:82:43:fc`  |
| 6     | Réponse ARP | x         | `router` `08:00:27:82:43:fc` | x              | `marcel` `08:00:27:25:d8:f5`  |


> Vous pourriez, par curiosité, lancer la capture sur `john` aussi, pour voir l'échange qu'il a effectué de son côté.

🦈 **Capture réseau `tp3_routage_marcel.pcapng`**

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

- ajoutez une carte NAT en 3ème inteface sur le `router` pour qu'il ait un accès internet

```
[root@localhost ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:7d:ad:70 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 84739sec preferred_lft 84739sec
    inet6 fe80::a00:27ff:fe7d:ad70/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

3: enp0s6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000

4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
```

- ajoutez une route par défaut à `john` et `marcel`

Côté `john`
```
[root@localhost ~]$ ip route add default via 10.3.1.254 dev enp0s6
```

Côté `marcel`
```
[root@localhost ~]$ ip route add default via 10.3.2.254 dev enp0s6
```
  - vérifiez que vous avez accès internet avec un `ping`

Côté `john`
```
[root@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=23,9 ms
```

Côté `marcel`
```
[root@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=22,6 ms
```
  - le `ping` doit être vers une IP, PAS un nom de domaine
- donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser
  - vérifiez que vous avez une résolution de noms qui fonctionne avec `dig`
  - puis avec un `ping` vers un nom de domaine

🌞**Analyse de trames**

- effectuez un `ping 8.8.8.8` depuis `john`
- capturez le ping depuis `john` avec `tcpdump`
- analysez un ping aller et le retour qui correspond et mettez dans un tableau :

| ordre | type trame | IP source          | MAC source                 | IP destination     | MAC destination            |
|-------|------------|--------------------|----------------------------|--------------------|----------------------------|
| 1     | ping       | `john` `10.3.1.11` | `john` `08:00:27:a8:58:c9` | `8.8.8.8`          | `08:00:27:27:f6:5a`        |
| 2     | pong       | `8.8.8.8`          | `08:00:27:27:f6:5a`        | `john` `10.3.1.11` | `john` `08:00:27:a8:58:c9` |

🦈 **Capture réseau `tp3_routage_internet.pcapng`**

## III. DHCP

On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests.

| Machine  | `10.3.1.0/24`              | `10.3.2.0/24` |
|----------|----------------------------|---------------|
| `router` | `10.3.1.254`               | `10.3.2.254`  |
| `john`   | `10.3.1.11`                | no            |
| `bob`    | oui mais pas d'IP statique | no            |
| `marcel` | no                         | `10.3.2.12`   |

```schema
   bob                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └─┬─┘    └─────┘    └───┘    └─────┘
   dhcp        │
  ┌─────┐      │
  │     │      │
  │     ├──────┘
  └─────┘
```

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

- installation du serveur sur `john`

**ATTENTION** : Beaucoup de lignes inutiles mais c'est censé prouver que le serveur DHCP marche 
```
[root@localhost ~]$ journalctl -xeu dhcpd.service
Oct 16 14:25:47 localhost.localdomain systemd[1]: Starting DHCPv4 Server Daemon...
░░ Subject: A start job for unit dhcpd.service has begun execution
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░
░░ A start job for unit dhcpd.service has begun execution.
░░
░░ The job identifier is 1670.
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Internet Systems Consortium DHCP Server 4.4.2b1
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Copyright 2004-2019 Internet Systems Consortium.
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: All rights reserved.
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: For info, please visit https://www.isc.org/software/dhcp/
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: ldap_gssapi_principal is not set,GSSAPI Authentication for LDAP will>
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Not searching LDAP since ldap-server, ldap-port and ldap-base-dn wer>
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Not searching LDAP since ldap-server, ldap-port and ldap-base-dn wer>
Oct 16 14:25:47 localhost.localdomain systemd[1]: Starting DHCPv4 Server Daemon...
░░ Subject: A start job for unit dhcpd.service has begun execution
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░
░░ A start job for unit dhcpd.service has begun execution.
░░
░░ The job identifier is 1670.
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Internet Systems Consortium DHCP Server 4.4.2b1
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Copyright 2004-2019 Internet Systems Consortium.
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: All rights reserved.
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: For info, please visit https://www.isc.org/software/dhcp/
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: ldap_gssapi_principal is not set,GSSAPI Authentication for LDAP will>
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Not searching LDAP since ldap-server, ldap-port and ldap-base-dn wer>
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Config file: /etc/dhcp/dhcpd.conf
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Database file: /var/lib/dhcpd/dhcpd.leases
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: PID file: /var/run/dhcpd.pid
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Source compiled to use binary-leases
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Wrote 0 leases to leases file.
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Listening on LPF/enp0s6/08:00:27:a8:58:c9/10.3.1.0/24
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Sending on   LPF/enp0s6/08:00:27:a8:58:c9/10.3.1.0/24
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Sending on   Socket/fallback/fallback-net
Oct 16 14:25:47 localhost.localdomain dhcpd[1429]: Server starting service.
Oct 16 14:25:47 localhost.localdomain systemd[1]: Started DHCPv4 Server Daemon.
░░ Subject: A start job for unit dhcpd.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░
░░ A start job for unit dhcpd.service has finished successfully.
```
- créer une machine `bob`
- faites lui récupérer une IP en DHCP à l'aide de votre serveur

```
[root@localhost ~]$ ip a
2: enp0s6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:6a:05:b5 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s6
```

> Il est possible d'utiliser la commande `dhclient` pour forcer à la main, depuis la ligne de commande, la demande d'une IP en DHCP, ou renouveler complètement l'échange DHCP (voir `dhclient -h` puis call me et/ou Google si besoin d'aide).

🌞**Améliorer la configuration du DHCP**

- ajoutez de la configuration à votre DHCP pour qu'il donne aux clients, en plus de leur IP :
  - une route par défaut
  - un serveur DNS à utiliser

Ajout des lignes dans le fichier dhcpd.conf :
```
option routers 10.3.1.254;
option subnet-mask 255.255.255.0;
option domain-name-servers 8.8.8.8;
```
- récupérez de nouveau une IP en DHCP sur `bob` pour tester :

Pour libérer l'adresse IP actuelle
```
[root@localhost ~]$ dhclient -r :
```
Pour obtenir une nouvelle adresse IP :
```
[root@localhost ~]$ dhclient
```
  - `bob` doit avoir une IP
    - vérifier avec une commande qu'il a récupéré son IP

    ```
    [root@localhost ~]$ ip a
    2: enp0s6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
        link/ether 08:00:27:6a:05:b5 brd ff:ff:ff:ff:ff:ff
        inet 10.3.1.2/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s6
          valid_lft 529sec preferred_lft 529sec
    ```
    - vérifier qu'il peut `ping` sa passerelle

    ```
    [root@localhost ~]$ ping 10.3.1.254
    PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
    64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.553 ms
    ```
  - il doit avoir une route par défaut
    - vérifier la présence de la route avec une commande

    ```
    [root@localhost ~]$ ip r s
    default via 10.3.1.254 dev enp0s6 proto dhcp src 10.3.1.2 metric 100
    10.3.1.0/24 dev enp0s6 proto kernel scope link src 10.3.1.2 metric 100
    ```
    - vérifier que la route fonctionne avec un `ping` vers une IP

    ```
    [root@localhost ~]$ ping 10.3.2.12
    PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
    64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=1.24 ms
    ```
  - il doit connaître l'adresse d'un serveur DNS pour avoir de la résolution de noms
    - vérifier avec la commande `dig` que ça fonctionne

    ```
    [root@localhost ~]$ dig google.com
    ;; ANSWER SECTION:
    google.com.             300     IN      A       216.58.209.238
    ```
    - vérifier un `ping` vers un nom de domaine

    ```
    [root@localhost ~]$ ping 216.58.209.238
    PING 216.58.209.238 (216.58.209.238) 56(84) bytes of data.
    64 bytes from 216.58.209.238: icmp_seq=1 ttl=247 time=25.4 ms
    64 bytes from 216.58.209.238: icmp_seq=2 ttl=247 time=23.6 ms
    ```

### 2. Analyse de trames

🌞**Analyse de trames**

- lancer une capture à l'aide de `tcpdump` afin de capturer un échange DHCP
- demander une nouvelle IP afin de générer un échange DHCP
- exportez le fichier `.pcapng`

🦈 **Capture réseau `tp3_dhcp.pcapng`**
