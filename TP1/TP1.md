# TP1 : Programmatic provisioning
## Sommaire

- [TP1 : Programmatic provisioning](#tp1--programmatic-provisioning)
  - [Sommaire](#sommaire)
- [I. Une première VM](#i-une-première-vm)
  - [1. ez startup](#1-ez-startup)
  - [2. Un peu de conf](#2-un-peu-de-conf)
- [II. Initialization script](#ii-initialization-script)
- [III. Repackaging](#iii-repackaging)
- [IV. Multi VM](#iv-multi-vm)
- [V. cloud-init](#v-cloud-init)



# I. Une première VM

## 1. ez startup

🌞 **`Vagrantfile` dans le dépôt git de rendu SVP !**

```bash
dorian@Air-de-Dorian cloud % cat Vagrantfile 
Vagrant.configure("2") do |config|
  config.vm.box = "bento/rockylinux-9-arm64"
end
```

 - [basic-conf-vagrant](./basic-conf-vagrant)

## 2. Un peu de conf

🌞 **Ajustez le `Vagrantfile` pour que la VM créée** :

```bash
dorian@Air-de-Dorian cloud % cat Vagrantfile 
Vagrant.configure("2") do |config|
  config.vm.box = "bento/rockylinux-9-arm64"
  config.vm.network "private_network", ip: "10.1.1.11", netmask: "255.255.255.0"
  config.vm.hostname = "ezconf.tp1.efrei"
  config.vm.provider "vmware_desktop" do |v|
    v.vmx["memsize"] = "2048"
    v.vmx["name"] = "ezconf.tp1.efrei"
    # Impossible d'allouer une taille spécifique de disque sur fusion 🙃
  end
end
```

 - [easiest-conf-vagrant](./easiest-conf-vagrant)


# II. Initialization script

🌞 **Ajustez le `Vagrantfile`** :

> Vagrantfile and script [here](./init-script/)

# III. Repackaging

🌞 **Repackager la VM créée précédemment**

> Utilisation du Vagrantfile et du script contenu dans [init-script](./init-script/)

```bash
vagrant package --output rocky-pkg.box

vagrant box add rocky-pkg rocky-pkg.box

dorian@Air-de-Dorian cloud % vagrant box list                                    
bento/rockylinux-9-arm64 (vmware_desktop, 202401.31.0, (arm64))
hashicorp/bionic64       (vmware_desktop, 1.0.282)
rocky-pkg              (vmware_desktop, 0)
```

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

- première étape, créer un fichier texte nommé `meta-data` avec le contenu suivant

```yml
---
local-hostname: cloud-init-test.tp1.efrei
```

- ensuite, créer un fichier texte nommé `meta-data` avec le contenu suivant

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
genisoimage -output cloud-init.iso -volid cidata -joliet -r meta-data meta-data
```

![No magic](./img/cloud-init.png)