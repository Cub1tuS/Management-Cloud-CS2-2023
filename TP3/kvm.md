# II. 2. Noeuds KV

## Sommaire

- [II. 2. Noeuds KVM](#ii-2-noeuds-kvm)
  - [Sommaire](#sommaire)
  - [A. KVM](#a-kvm)
  - [B. Système](#b-système)
  - [C. Ajout des noeuds au cluster](#c-ajout-des-noeuds-au-cluster)

## A. KVM

🌞 **Ajouter des dépôts supplémentaires**

```bash
[vagrant@rocky9 yum.repos.d]$ nano opennebula.repo
```

> J'ajoute le contenu suivant dans "opennebula.repo"

```config
[opennebula]
name=OpenNebula Community Edition
baseurl=https://downloads.opennebula.io/repo/6.8/RedHat/$releasever/$basearch
enabled=1
gpgkey=https://downloads.opennebula.io/repo/repo2.key
gpgcheck=1
repo_gpgcheck=1
```

```bash
dnf install -y epel-release
```

🌞 **Installer KVM**

```bash
dnf install opennebula-node-kvm
```

🌞 **Démarrer le service `libvirtd`**

```bash
[vagrant@rocky9 ~]$ sudo systemctl start libvirtd
[vagrant@rocky9 ~]$ sudo systemctl enable libvirtd
```


## B. Système

🌞 **Ouverture firewall**

```bash
[vagrant@rocky9 ~]$ sudo firewall-cmd --permanent --add-port=22/tcp
success
[vagrant@rocky9 ~]$ sudo firewall-cmd --permanent --add-port=8472/udp
success
[vagrant@rocky9 ~]$ sudo firewall-cmd --reload
success
```

| Port | Proto | Why ? |
|------|-------|-------|
| 22   | TCP   | SSH   |
| 8472 | UDP   | VXLAN |

🌞 **Handle SSH**


```bash
[oneadmin@rocky9 ~]$ ssh-keygen -t rsa -b 4096
```

***MANUALLY ADD PUB KEY FROM FRONTEND ON KVM NODES***

```bash
[oneadmin@rocky9 ~]$ ssh-keyscan 10.3.1.174
[oneadmin@rocky9 ~]$ ssh-keyscan 10.3.1.156
```

## C. Ajout des noeuds au cluster

![proof](proof%20on.png)