# TP2 : Network boot

## Sommaire

- [TP2 : Network boot](#tp2--network-boot)
  - [Sommaire](#sommaire)
- [I. Installation d'un serveur DHCP](#i-installation-dun-serveur-dhcp)
- [II. Installation d'un serveur TFTP](#ii-installation-dun-serveur-tftp)
- [III. Un peu de conf](#iii-un-peu-de-conf)
- [IV. Installation d'un serveur Apache](#iv-installation-dun-serveur-apache)
- [V. Test](#v-test)

# I. Installation d'un serveur DHCP

🌞 **Installer le paquet `dhcp-server`**

```bash
sudo dnf install dhcp-server
```

🌞 **Configurer le serveur DHCP**

```conf
[root@rockypxe ~]# cat /etc/dhcp/dhcpd.conf 
default-lease-time 600;
max-lease-time 7200;
authoritative;

# PXE specifics
option space pxelinux;
option pxelinux.magic code 208 = string;
option pxelinux.configfile code 209 = text;
option pxelinux.pathprefix code 210 = text;
option pxelinux.reboottime code 211 = unsigned integer 32;
option architecture-type code 93 = unsigned integer 16;

subnet 10.1.1.0 netmask 255.255.255.0 {
    # définition de la range pour que votre DHCP attribue des IP entre <FIRST_IP> <LAST_IP>
    range dynamic-bootp 10.1.1.10 10.1.1.20;
    
    # add follows
    class "pxeclients" {
        match if substring (option vendor-class-identifier, 0, 9) = "PXEClient";
        next-server 10.1.1.1;

        if option architecture-type = 00:07 {
            filename "BOOTX64.EFI";
        }
        else {
            filename "pxelinux.0";
        }
    }
}
```

🌞 **Démarrer le serveur DHCP**

```bash
systemctl restart dhcpd
```

🌞 **Ouvrir le bon port firewall**

```bash
[root@rockypxe ~]# firewall-cmd --add-service=dhcp --permanent
success
```

```bash
[root@rockypxe ~]# firewall-cmd --reload
success
```

# II. Installation d'un serveur TFTP

🌞 **Installer le paquet `tftp-server`**

```bash
[root@rockypxe ~]# dnf install -y tftp-server
```

🌞 **Démarrer le socket TFTP**

```bash
[root@rockypxe ~]# systemctl enable --now tftp.socket
Created symlink /etc/systemd/system/sockets.target.wants/tftp.socket → /usr/lib/systemd/system/tftp.socket.
```

🌞 **Ouvrir le bon port firewall**

```bash
[root@rockypxe ~]# firewall-cmd --add-service=tftp --permanent
success
```

```bash
[root@rockypxe ~]# firewall-cmd --reload
success
```

# III. Un peu de conf

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