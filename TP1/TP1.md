# TP1 : Programmatic provisioning

**Le terme *provisioning* correspond simplement au fait de créer des machines virtuelles, et leur appliquer une configuration basique.**

> *Derrière le terme de *provisioning* peuvent se cacher, suivant le contexte oui l'interlocuteur, quelques subtilités.*

On parle donc de *VM provisioning*, ou *infrastructure provisioning* : on réserve des ressources afin d'y exécuter des VMs.

**Dans le monde du cloud, on utilise des approches programmatiques pour gérer le provisioning.**  
C'est à dire qu'on écrit du code, et les VMs sont créées à l'exécution de notre code.

> Plutôt que de faire des clics clics, on écrit du code. **On parle d'infrastructure as code ou IaC.**

**Pour vous faire prendre en main un outil pratique, et commencer à mettre les mains dans l'infrastructure as code, on va jouer avec Vagrant dans ce TP.**

Vagrant permet de piloter, via du code, un hyperviseur local comme VirtualBox ou Hyper-V.

> Nous passerons sûrement sur Terraform + Azure dans un deuxième temps. Le principe reste exactement le même.

## Sommaire

- [TP1 : Programmatic provisioning](#tp1--programmatic-provisioning)
  - [Sommaire](#sommaire)
- [0. Setup](#0-setup)
- [I. Une première VM](#i-une-première-vm)
  - [1. ez startup](#1-ez-startup)
  - [2. Un peu de conf](#2-un-peu-de-conf)
- [II. Initialization script](#ii-initialization-script)
- [III. Repackaging](#iii-repackaging)
- [IV. Multi VM](#iv-multi-vm)
- [V. cloud-init](#v-cloud-init)

# 0. Setup

➜ **[VirtualBox](https://www.virtualbox.org/)**

Pour ce TP, pour ce qui est de l'hyperviseur local, je vous recommande d'utiliser VirtualBox comme hyperviseur (opensource et cross-platform).  
Vagrant supporte aussi VMWare Workstation et Microsoft Hyper-V, vous êtes libres de les utiliser.

➜ **[Vagrant](https://www.vagrantup.com/)**

Il faudra télécharger Vagrant sur votre poste, afin de piloter votre hyperviseur local.

# I. Une première VM

## 1. ez startup

> Vagrant s'utilise depuis un terminal sur votre poste. Je vous recommande Git Bash (et pas Powershell, pour une fois) si vous êtes sur Windows.

Dans ce TP, vous êtes complètement libres du choix de l'OS, mais si vous voulez que tout roule, vous pouvez juste rester avec les images que je recommande.

Pour cette première VM, on va lancer un Rocky 9 (basé sur RHEL, avec sécurité et robustessee en tête).

➜ **Préparons le terrain pour la première VM**

```bash
# déplacez-vous dans un répertoire de travail
$ mkdir /path/to/workdir
$ cd /path/to/workdir

# générez un fichier Vagrantfile initial
$ vagrant init generic/rocky9

# un fichier Vagrantfile a été créé dans le dossier courant
# explorez son contenu, c'est quasiment que des commentaires quand on le génère comme ça
$ cat Vagrantfile
```

Avec ce fichier basique, il est possible d'allumer une simple VM. On peut trouver en commentaires les lignes de conf souvent utilisées (mais ce n'est pas exhaustif). Comme par exemple la configuration d'une IP statique sur une machine.

Une fois vous avez un peu lu, **SUPPRIMEZ TOUS LES COMMENTAIRES** svpliz pour voir clair dans votre fichier.

---

Pour le moment, la conf, on s'en branle, on va juste allumer la VM ! Komsa :

```bash
# on allume la VM
# ça marche que si on est dans un dossier qui contient un Vagrantfile
$ vagrant up

# voir le statut de la VM
$ vagrant status

# une fois terminé, on peut SSH vers notre VM facilement avec :
$ vagrant ssh

# si vous voulez voir les paramètres de la commande SSH effectuée par Vagrant vous pouvez utiliser
$ vagrant ssh-config

# enfin vous pouvez éteindre et détruire la VM avec
$ vagrant halt
$ vagrant destroy -f # -f permet de détruire même si on a pas éteint (halt) la VM avant
```

➜ *Vous pouvez aussi constater que la VM est allumée dans votre hyperviseur.*

🌞 **`Vagrantfile` dans le dépôt git de rendu SVP !**

> **Y'aura plusieurs Vagrantfile dans le TP**, alors hésitez pas à les renommer dans le rendu, faire des dossiers, toussa, clean quoi !

![Vagrant up](./img/vagrant.png)

## 2. Un peu de conf

Avec Vagrant, il est possible de gérer un certains nombres de paramètres de la VM.

🌞 **Ajustez le `Vagrantfile` pour que la VM créée** :

- ait l'IP `10.1.1.11/24`
- porte le hostname `ezconf.tp1.efrei`
- porte le nom (pour Vagrant) `ezconf.tp1.efrei` (ce n'est pas le hostname de la machine)
- ait 2G de RAM
- ait un disque dur de 20G

Je vous laisse vous aventurer sur le grand internet pour trouver les lignes de conf à ajouter au `Vagrantfile` pour ça.

> N'hésitez surtout pas à m'appeler si vous galérez avec ça, cherchez d'abord, mais ne galérez pas 10 plombes sur des soucis de syntaxe !

# II. Initialization script

Quand Vagrant allume une VM, il peut lui ordonner d'exécuter un script une fois le démarrage terminé.

> On se rapproche donc d'un réel provisioning programmatique avec la création de la VM + une configuration élémentaire.

Ici, on va rester simples : un ptit script shell qui installera quelques paquets.

🌞 **Ajustez le `Vagrantfile`** :

- quand la VM démarre, elle doit exécuter un script bash
- le script installe les paquets `vim` et `python3`
- il met aussi à jour le système avec un `dnf update -y` (si c'est trop long avec le réseau de l'école, zappez cette étape)
- ça se fait avec une ligne comme celle-ci :

```Vagrantfile
# on suppose que "script.sh" existe juste à côté du Vagrantfile
config.vm.provision "shell", path: "script.sh" 
```

# III. Repackaging

Pour accélérer le déploiement, mais aussi pour intégrer une conf dès le boot de la VM, on peut **repackager les boxes avec Vagrant.**

C'est à dire qu'on peut créer un template de VM quoi : **on crée une image custom qui contient déjà la conf, et on a plus qu'à allumer des VMs à partir de cette image.**

Une fois qu'une VM est allumée, qu'on y a fait un peu de conf, on peut à tout moment la transformer en un nouveau template (une nouvelle "box" au sens de Vagrant).
Et on pourra donc créer de nouvelles VMs qui contiennent déjà cette conf.

La marche à suivre pour faire ça est la suivante :

```bash
# toujours depuis le même répertoire, avec la VM allumée
$ vagrant package --output rocky-efrei.box

# on ajoute le fichier .box produit à la liste des box que gère Vagrant
$ vagrant box add rocky-efrei rocky-efrei.box

# la box est visible dans la liste des box Vagrant
$ vagrant box list
```

> Vous l'avez compris une "box" c'est juste une VM qui est prête à être clonée. Il existe un répertoire public de box, où n'importe qui peut y mettre sa ptite box, c'est le [Vagrant Cloud](https://app.vagrantup.com/boxes/search).

🌞 **Repackager la VM créée précédemment**

- comme ça vous aurez une box qui contient un OS déjà à jour, avec quelques paquets préinstallés
- donnez moi la suite de commande dans le compte-rendu de TP

> Cette idée d'utiliser un template pour provisionner des VM par la suite est extrêmement répandue. Utile par exemple pour avoir des machines qui sont conformes dès leur installation à une politique de sécurité.

# IV. Multi VM

Il est -évidemment- possible de créer plusieurs VMs à l'aide d'un seul `Vagrantfile` et donc de une seule commande `vagrant up`.

Il y a deux façons de faire ça dans le `Vagrantfile` :

- on duplique tout le bloc qui permet de lancer une VM
  - à l'aide des mots-clés `config.vm.define`
  - c'est bourrin un peu, mais ça permet de gérer tous les cas
- on fait une boucle, qui, à chaque itération, crée une VM
  - élégant, et on peut faire des trucs genre attribuer une IP qui s'incrémente à chaque itération de la boucle
  - mais on peut pas tout faire parfois

Idem ici, vous choisissez la méthode que vous voulez, go internet pour la syntaxe.

🌞 **Un deuxième `Vagrantfile` qui définit** :

- une VM `node1.tp1.efrei`
  - IP `10.1.1.101/24`
  - 2G de RAM
- une VM `node2.tp1.efrei`
  - IP `10.1.1.102/24`
  - 1G de RAM

🌞 **Une fois les VMs allumées, assurez-vous que vous pouvez ping `10.1.1.102` depuis `node1`**

# V. cloud-init

Vous vous souvenez de lui ? Non ? Bah tant mieux, petite partie pour vous rafraîchir la mémoire justement :D

**`cloud-init` est un outil qui permet à une VM de s'autoconfigurer dès le premier boot.**

Ca permet d'avoir une syntaxe standard pour définir des trucs standards (créer un user, poser une clé SSH, installer un paquet, etc), plutôt que de reposer sur des scripts shell super spécifiques, et prompts à l'erreur.

De plus, énormément d'OS l'ont adopté, c'est devenu un techno de choix chez la plupart des hébergeurs cloud dans les chaînes de provisioning des opérateurs Cloud.

> *Toutes les plateformes cloud comme Azure ou AWS ou d'autres utilisent `cloud-init` pour créer des VMs avec un user et une clé SSH déposés pour vous.*

---

Ici, dans le cadre du TP, vous allez :

- ajouter `cloud-init` à la box que vous avez repackagée
- créer un fichier `.iso` qui contient les données `cloud-init` de notre choix
  - comme une création d'utilisateurs
- on pourra ensuite lancer une VM, qui se base sur cette box, et qui s'autoconfigurera toute seule au boot

🌞 **Repackager une box Vagrant**

- cette box doit contenir le paquet `cloud-init` pré-installé
- il faut aussi avoir saisi `systemctl enable cloud-init` après avoir l'instalaltion du paquet
  - cela permet à `cloud-init` de démarrer automatiquement au prochain boot de la machine

🌞 **Tester !**

- écrire un `Vagrantfile` qui utilise la box repackagée
- il faudra ajouter un CD-ROM (un `.iso`) à la VM qui contient nos données `cloud-init`
  - uiui un `.iso` c'est un CD-ROM virtuel, et ui c'est la méthode plutôt standard avec `cloud-init`
  - référez-vous aux instructions juste en dessous pour savoir comment construire ce `.iso`
- allumez la VM avec `vagrant up` et vérifiez que `cloud-init` a bien créé l'utilisateur, avec le bon password, et la bonne clé SSH

➜ **Construire le `.iso` qui contient les données `cloud-init`**

- première étape, créer un fichier texte nommé `user-data` avec le contenu suivant

```yml
---
local-hostname: cloud-init-test.tp1.efrei
```

- ensuite, créer un fichier texte nommé `user-data` avec le contenu suivant

```yml
---
users:
  - name: nom_de_ton_user
    primary_group: nom_de_ton_groupe # pareil que le user généralement
    groups: wheel # sur un système redhat, t'as full accès à sudo si t'es membre du groupe wheel
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL # on fait les forceurs sur sudo :D
    lock_passwd: false
    passwd: <HASH_DU_PASSWORD_MEME_FORMAT_QUE_/etc/shadow>
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1l3R4CNTE5AAAAIMO/JQ3AtA3k8iXJWlkdUKSHDh215OKyLR0vauzD7BgA # mettez votre propre clé
```

- enfin, on peut générer le `.iso` à partir de ces deux fichiers avec la commande suivante

```bash
# rien à changer dans cette commande, à part le nom de l'iso de sortie si vous souhaitez
# il faut OBLIGATOIREMENT laisser le volid à "cidata" : c'est grâce à ce tag que cloud-init reconnaît ce disque
genisoimage -output cloud-init.iso -volid cidata -joliet -r meta-data user-data
```

![No magic](./img/cloud-init.png)