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

Ici, dans le cadre du TP, vous allez :

- ajouter `cloud-init` à la box que vous avez repackagée
- créer un fichier `.iso` qui contient les données `cloud-init` de notre choix
  - comme une création d'utilisateurs
- on pourra ensuite lancer une VM, qui se base sur cette box, et qui s'autoconfigurera toute seule au boot

## 1. Repackaging

🌞 **Repackager une box Vagrant**

- cette box doit contenir le paquet `cloud-init` pré-installé
- il faut aussi avoir saisi `systemctl enable cloud-init` après avoir l'installation du paquet
  - cela permet à `cloud-init` de démarrer automatiquement au prochain boot de la machine

```bash
script.sh

dnf update -y
dnf install -y cloud-init
systemctl enable cloud-init
```

```ruby
Vagrantfile

Vagrant.configure("2") do |config|
  config.vm.box = "bento/rockylinux-9-arm64"
  config.vm.provision "shell", path: "script.sh" 
end
```

## 2. Test

### C. Ajout d'un périphérique CD/DVD sur une VM Debian via Fusion

- Lancement VM Debian
- Installation paquet cloud-init et activation du service
- Extinction de la machine puis clonage
- Ajout périph sur le clone + ton ISO dans le périph
- Lancement du clone

🌞 **Tester !**

```bash
root@cloud:/home/dorian$ cat /etc/shadow
root:!:19710:0:99999:7:::
[...]
dorian:$y$j9T$AV09lHt6OrBxPfCZwMRcj0$woKO0suisr.i.M/nYTz.WFf5IYi4voDlurXG/Jlslj1:19710:0:99999:7:::
lala:$6$bV62paDqH/ZQSVFb$jiBgcgpkuzmmoZSvvLPwpd4gjwvnKQEWTE119tMNTnICtMcJ6dyPcDCVaTur8j5UQFuxAAM6eTimGdr97Nagh1:19816:0:99999:7:::
```

```bash
root@cloud:~# cloud-init status
status: done
````
