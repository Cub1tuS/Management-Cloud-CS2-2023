# TP 1.5 : Proposer une image renforcée

## Sommaire

- [TP 1.5 : Proposer une image renforcée](#tp-15--proposer-une-image-renforcée)
  - [Sommaire](#sommaire)
  - [1. Guides CIS](#1-guides-cis)
  - [2. Conf SSH](#2-conf-ssh)
  - [3. DoT](#3-dot)

## 1. Guides CIS

🌞 **Suivre un guide CIS**

- *vous pouvez faire toutes ces confs à la main, avant de repackager*
- téléchargez le guide CIS de Rocky 9 [ici](https://downloads.cisecurity.org/#/)
- vous devez faire :
  - la section 2.1
  - les sections 3.1 3.2 et 3.3
  - toute la section 5.2 Configure SSH Server
  - au moins 10 points dans la section 6.1 System File Permissions
  - au moins 10 points ailleurs sur un truc que vous trouvez utile

> Le but c'est pas de rush mais *comprendre* ce que vous faites, comprendre ici pourquoi c'est important de vérifier que ces trucs sont activés ou désactivés. Et très bon pour votre culture.

- 2.1
```bash
[vagrant@localhost ~]$ chronyc sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^+ amy.chl.la                    2   6   377    59    -19ms[  -19ms] +/-   20ms
^* karma.rsx11.net               2   6   377    61    -12ms[  -36ms] +/-   22ms
^- main.arcanite.ch              2   6   377    59    -22ms[  -22ms] +/-   54ms
^- ns3166978.ip-51-178-79.eu     2   6   377    59    -21ms[  -21ms] +/-   56ms
```

- 3.1
```bash

```

## 2. Conf SSH

![SSH](./img/ssh.jpg)

🌞 **Chiffrement fort côté serveur**

- *vous pouvez faire toutes ces confs à la main, avant de repackager*
- trouver une ressource de confiance (je veux le lien en compte-rendu) qui indique quelle configuration vous devriez mettre en place
- configurer le serveur SSH pour qu'il utilise des paramètres forts en terme de chiffrement (je veux le fichier de conf dans le compte-rendu)
  - des lignes de conf à ajouter dans le fichier de conf
  - regénérer des clés pour le serveur ?
  - regénérer les paramètres Diffie-Hellman ? (se renseigner sur Diffie-Hellman ?)

🌞 **Clés de chiffrement fortes pour le client**

- *vous déposerez votre clé avec `cloud-init` dans la VM lors de son premier boot*
- trouver une ressource de confiance (je veux le lien en compte-rendu) qui indique quel chiffrement vous devriez utiliser
- générez-vous une paire de clés qui utilise un chiffrement fort et une passphrase
- ne soyez pas non plus absurdes dans le choix du chiffrement quand je dis "fort" (genre pas de RSA avec une clé de taile 98789080932083209 bytes, il faut que ça reste réaliste et utile)

🌞 **Connectez-vous en SSH à votre VM avec cette paire de clés**

- prouvez en ajoutant `-vvvv` sur la commande `ssh` de connexion que vous utilisez bien cette clé là

## 3. DoT

Ca commence à faire quelques années maintenant que plusieurs acteurs poussent pour qu'on fasse du DNS chiffré, et qu'on arrête d'envoyer des requêtes DNS en clair dans tous les sens.

En effet, par défaut dans la plupart des utilisations, les requêtes DNS émises par un client sont envoyées en clair. Ainsi n'importe qui peut observer quelles requêtes DNS sont effectuées par un client donné, et éventuellement jouer avec.

**Le DoT est une techno qui permet de se protéger de ça : DoT pour DNS over TLS. On fait nos requêtes DNS dans des tunnels chiffrés avec le protocole TLS.**

🌞 **Configurer la machine pour qu'elle fasse du DoT**

- *vous pouvez faire toutes ces confs à la main, avant de repackager*
- installez `systemd-resolved` sur la machine pour ça
- activez aussi DNSSEC tant qu'on y est
- [référez-vous à cette doc qui est cool par exemple](https://wiki.archlinux.org/title/systemd-resolved)
- utilisez le serveur public de CloudFlare : 1.1.1.1 (il supporte le DoT)

> *Donc normalement : install du paquet, modif du fichier `/etc/systemd/resolved.conf` pour activer le DoT, activer DNSSEC et utiliser `1.1.1.1`, puis une commande pour modifier le contenu de `/etc/resolv.conf`, et enfin, redémarrer le service `systemd-resoved`.*

🌞 **Vérifiez que les requêtes DNS effectuées par la machine...**

- ont une réponse qui provient du serveur que vous avez conf (normalement c'est `127.0.0.1` avec `systemd-networkd` qui tourne)
  - quand on fait un `dig ynov.com` on voit en bas quel serveur a répondu
- mais qu'en réalité, la requête a été forward vers 1.1.1.1 avec du TLS
  - vous pouvez (*devriez*) utiliser **Wireshark** pour vérifier ça : voir que la requête DNS ne circule pas en clair

![Spying](./img/spy.png)