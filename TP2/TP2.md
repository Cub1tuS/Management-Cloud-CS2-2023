# TP2 : Network boot

**Dans ce TP on va voir une autre techno centrale dans les infras de grande taille comme typiquement les infras Cloud : PXE.**

Soit y'a du PXE, soit y'a juste du clone de VM (c'est à dire : on duplique le disque dur virtuel).

Mais bon, quand c'est pas des VMs et que c'est des serveurs physique, y'a pas de bouton clone hihi 🌻

➜ **PXE est un protocole qui permet à une machine avec un disque dur vierge (sans OS installé) de démarrer, et de lancer une installation à travers le réseau.**

Autrement dit, c'est possible de juste allumer un nouveau serveur physique ou une nouvelle VM, et sans aucune autre intervention manuelle (juste tu l'allumes), la VM est complètement configurée (OS installé, configuration élémentaire, etc.).

Dans ce TP on va donc monter un serveur PXE, dans une VM, ce qui permettra aux autres VMs de démarrer une installation d'OS à travers le réseau.

➜ **PXE repose lui sur d'autres protocoles assez standards :**

- **DHCP** : avant de commencer une install réseau, la VM cliente va devoir récup une IP en DHCP
- **TFTP et/ou HTTP** : le protocole qui sera utilisé pour télécharger les données depuis le serveur PXE

➜ **Le flow typique de PXE, c'est donc :**

- on allume une VM vierge (ou machine physique)
- elle récup une IP en DHCP
- grâce à des options DHCP, elle connaît à quelle IP faire les requêtes suivantes
- elle fait une requête HTTP pour récup de quoi commencer une installation
- l'installation démarre

> *On peut aller plus loin, et automatiser aussi l'installation elle-même. Ca se fait avec Kickstart pour les systèmes RedHat par exemple. On va pas voir ça ensemble dans ce TP, mais c'est lâché si vous voulez aller voir ;)*

![PXE](./img/pxe.jpg)

## Sommaire

- [TP2 : Network boot](#tp2--network-boot)
  - [Sommaire](#sommaire)
- [0. Setup](#0-setup)
- [I. Installation d'un serveur DHCP](#i-installation-dun-serveur-dhcp)
- [II. Installation d'un serveur TFTP](#ii-installation-dun-serveur-tftp)
- [III. Un peu de conf](#iii-un-peu-de-conf)
- [IV. Installation d'un serveur Apache](#iv-installation-dun-serveur-apache)
- [V. Test](#v-test)

# 0. Setup

➜ Je vous conseille de pop la VM avec Vagrant, mais comme vous voulez en vrai !

➜ Toujours sur Rocky Linux. Une seule VM où on installe tout.

# I. Installation d'un serveur DHCP

🌞 **Installer le paquet `dhcp-server`**

🌞 **Configurer le serveur DHCP**

- il doit filer des IPs dans la bonne range
- la conf doit aussi contenir une section spécifique pour PXE
  - dans le bloc `subnet { }`, ajoutez la conf suivante :
  - n'oubliez pas de remplacer `<IP_DU_SERVEUR_PXE>` par l'IP de votre VM

```conf
    class "pxeclients" {
        match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
        next-server <IP_DU_SERVEUR_PXE>;
        if option architecture-type = 00:07 {
            filename "BOOTX64.EFI";
        }
        else {
            filename "pxelinux.0";
        }
    }
```

🌞 **Démarrer le serveur DHCP**

🌞 **Ouvrir le bon port firewall**

- avec `sudo firewall-cmd --add-service=dhcp --permanent` suivi de `sudo firewall-cmd --reload`

# II. Installation d'un serveur TFTP

Normalement c'est que pour du legacy, mais on le fait au cas où, suivant vos setups, i don't know.

C'est vitefé.

🌞 **Installer le paquet `tftp-server`**

🌞 **Démarrer le socket TFTP**

- avec un `sudo systemctl enable --now tftp.socket`

🌞 **Ouvrir le bon port firewall**

- avec `sudo firewall-cmd --add-service=tftp --permanent` suivi de `sudo fireswall-cmd --reload`

> *C'est du port 69 en TCP ou UDP le protocole TFTP.*

# III. Un peu de conf

Dans cette section, on va récupérer certains fichier contenus dans l'ISO officiel de Rocky Linux, afin de permettre à d'autres machines de les récupérer afin de démarrer un boot sur le réseau.

Avec PXE c'est le délire : on fournit un ISO à travers le réseau, et les machines peuvent l'utiliser pour déclencher une install.

Let's go :

➜ Déjà, récupérez l'iso de Rocky Linux dans la VM.

➜ Ensuite, suivez le guide :

```bash
# on installe les bails nécessaires à l'install d'un nouveao Rocky
dnf -y install syslinux

# on déplace le fichier pxelinux.0 dans le dossier servi par le serveur HTTP/TFTP
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/

# on prépare l'environnement
mkdir -p /var/pxe/rocky9
mkdir /var/lib/tftpboot/rocky9

# adaptez avec le chemin vers l'iso de Rocky sur votre VM
mount -t iso9660 -o loop,ro /path/vers/liso/de/rocky.iso /var/pxe/rocky9

# on récupère dans l'iso de Rocky le nécessaire pour démarrer une install
cp /var/pxe/rocky9/images/pxeboot/{vmlinuz,initrd.img} /var/lib/tftpboot/rocky9/
cp /usr/share/syslinux/{menu.c32,vesamenu.c32,ldlinux.c32,libcom32.c32,libutil.c32} /var/lib/tftpboot/

# on prépare le dossier qaui va contenir les options de boot réseau
mkdir /var/lib/tftpboot/pxelinux.cfg
```

➜ Puis, déposez le contenu suivant dans le fichier `/var/lib/tftpboot/pxelinux.cfg/default` :

```conf
default vesamenu.c32
prompt 1
timeout 60

display boot.msg

label linux
  menu label ^Install Rocky Linux 9 my big boiiiiiii
  menu default
  kernel rocky9/vmlinuz
  append initrd=rocky9/initrd.img ip=dhcp inst.repo=http://<IP_DU_SERVEUR_PXE>/rocky9
label rescue
  menu label ^Rescue installed system
  kernel rocky9/vmlinuz
  append initrd=rocky9/initrd.img rescue
label local
  menu label Boot from ^local drive
  localboot 0xffff
```

# IV. Installation d'un serveur Apache

🌞 **Installer le paquet `httpd`**

🌞 **Ajouter un fichier de conf dans `/etc/httpd/conf.d/pxeboot.conf`** avec le contenu suivant :

```apache
Alias /rocky9 /var/pxe/rocky9
<Directory /var/pxe/rocky9>
    Options Indexes FollowSymLinks
    # access permission
    Require ip 127.0.0.1 10.1.1.0/24 # remplace 10.1.1.0/24 par le réseau dans lequel se trouve le serveur
</Directory>
```

🌞 **Démarrer le serveur Apache**

🌞 **Ouvrir le bon port firewall**

- avec `sudo firewall-cmd --add-port=80/tcp --permanent` suivi de `sudo fireswall-cmd --reload`

# V. Test

Pour tester, simple :

- vous ouvrez VirtualBox à la main
- vous créez une nouvelle VM
  - pas un clone
  - vous lui mettez pas d'ISO ni rien non plus
  - genre une machine avec un disque vierge
- il faut qu'elle ait une interface dans le même réseau host-only que votre serveur PXE pour pouvoir le contacter
- vous allumez la VM
- une install de Rocky est censée se lancer

🌞 **Analyser l'échange complet avec Wireshark**

- le mieux pour réaliser la capture est sûrement d'utiliser `tcpdump` depuis le serveur PXE