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

🌞 **Un deuxième `Vagrantfile` qui définit** :

- une VM `node1.tp1.efrei`
  - IP `10.1.1.101/24`
  - 2G de RAM
- une VM `node2.tp1.efrei`
  - IP `10.1.1.102/24`
  - 1G de RAM

 - [multivm-conf-vagrant](./multivm-conf-vagrant)

```bash
dorian@Air-de-Dorian cloud % vagrant status 
Current machine states:

node1                     running (vmware_desktop)
node2                     running (vmware_desktop)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

🌞 **Une fois les VMs allumées, assurez-vous que vous pouvez ping `10.1.1.102` depuis `node1`**

```bash
dorian@Air-de-Dorian cloud % vagrant ssh node1

This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Tue Apr  2 16:47:50 2024 from 172.16.74.1
[vagrant@localhost ~]$ ping 10.1.1.102
PING 10.1.1.102 (10.1.1.102) 56(84) bytes of data.
64 bytes from 10.1.1.102: icmp_seq=1 ttl=64 time=1.11 ms
64 bytes from 10.1.1.102: icmp_seq=2 ttl=64 time=0.477 ms
^C
--- 10.1.1.102 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 0.477/0.793/1.110/0.316 ms
```

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

## 1. Repackaging

🌞 **Repackager une box Vagrant**

- cette box doit contenir le paquet `cloud-init` pré-installé
- il faut aussi avoir saisi `systemctl enable cloud-init` après avoir l'instalaltion du paquet
  - cela permet à `cloud-init` de démarrer automatiquement au prochain boot de la machine

## 2. Test

⚠️⚠️⚠️ **SOIT** vous le faites avec le fichier `.iso` (certains ont eu des petits soucis) **SOIT** direct avec Vagrant, **ne faites pas les deux** **(soit le A soit le B)**.

### A. Avec le fichier .iso

➜ **Construire le `.iso` qui contient les données `cloud-init`**

- première étape, créer un fichier texte nommé `meta-data` avec le contenu suivant

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

> *Vous avez été pluseurs à me faire la remarque, non ce n'est pas une erreur les `---` au début du document, c'est une bonne pratique pour le début de n'importe quel fichier `.yml`.*

- enfin, on peut générer le `.iso` à partir de ces deux fichiers avec la commande suivante

```bash
# rien à changer dans cette commande, à part le nom de l'iso de sortie si vous souhaitez
# il faut OBLIGATOIREMENT laisser le volid à "cidata" : c'est grâce à ce tag que cloud-init reconnaît ce disque
genisoimage -output cloud-init.iso -volid cidata -joliet -r meta-data user-data
```

> Si la commande `genisoimage` est pas dispo au sein de votre OS, vous pouvez récupérer [mon fichier ISO](./cloud-init.iso) (le user c'est `lala` et le password `dbc`)

🌞 **Tester !**

- écrire un `Vagrantfile` qui utilise la box repackagée
- il faudra ajouter un CD-ROM (un `.iso`) à la VM qui contient nos données `cloud-init`
  - uiui un `.iso` c'est un CD-ROM virtuel, et ui c'est la méthode plutôt standard avec `cloud-init`
  - référez-vous aux instructions juste en dessous pour savoir comment construire ce `.iso`
- allumez la VM avec `vagrant up` et vérifiez que `cloud-init` a bien créé l'utilisateur, avec le bon password, et la bonne clé SSH

### B. Directement avec Vagrant

➜ **Vagrant supporte le fait de fournir à une VM une conf `cloud-init`** (concrètement, il va lui aussi monter la conf sous la forme d'un `.iso`, vous pouvez le voir dans l'interface de VirtualBox après un `vagrant up`.

Ca peut se faire avec la ligne suivante dans un `Vagrantfile` :

```ruby
config.vm.cloud_init :user_data, content_type: "text/cloud-config", path: "user_data.yml"
```

Ce qui suppose la présence d'un fichier `user_data.yml` dans le même dossier que le Vagrantfile, avec un contenu comme :

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

🌞 **Tester !**

- écrire un `Vagrantfile` qui utilise la box repackagée
- il faudra ajouter un CD-ROM (un `.iso`) à la VM qui contient nos données `cloud-init`
  - uiui un `.iso` c'est un CD-ROM virtuel, et ui c'est la méthode plutôt standard avec `cloud-init`
  - référez-vous aux instructions juste en dessous pour savoir comment construire ce `.iso`
- allumez la VM avec `vagrant up` et vérifiez que `cloud-init` a bien créé l'utilisateur, avec le bon password, et la bonne clé SSH

> *Vous avez été pluseurs à me faire la remarque, non ce n'est pas une erreur les `---` au début du document, c'est une bonne pratique pour le début de n'importe quel fichier `.yml`.*

![No magic](./img/cloud-init.png)