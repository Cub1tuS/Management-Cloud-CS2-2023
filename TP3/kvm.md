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
[vagrant@kvm1 yum.repos.d]$ nano opennebula.repo
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
[vagrant@frontend ~]$ sudo systemctl start libvirtd
[vagrant@frontend ~]$ sudo systemctl enable libvirtd
```


## B. Système

🌞 **Ouverture firewall**

```bash
[vagrant@kvm1 ~]$ sudo firewall-cmd --permanent --add-port=22/tcp
success
[vagrant@kvm1 ~]$ sudo firewall-cmd --permanent --add-port=8472/udp
success
[vagrant@kvm1 ~]$ sudo firewall-cmd --reload
success
```

| Port | Proto | Why ? |
|------|-------|-------|
| 22   | TCP   | SSH   |
| 8472 | UDP   | VXLAN |

🌞 **Handle SSH**


```bash
[oneadmin@frontend ~]$ ssh-keygen -t rsa -b 4096
```

```bash
sed -i 's/127.0.0.1 rocky9.localdomain/10.3.1.250 rocky9.localdomain/g' /etc/hosts
```

***MANUALLY ADD PUB KEY FROM FRONTEND ON KVM NODES***

```bash
[oneadmin@frontend ~]$ ssh-keyscan 10.3.1.250 10.3.1.174 10.3.1.156 rocky9.localdomain > known_hosts
[oneadmin@kvm1 ~]$ ssh-keyscan 10.3.1.250 10.3.1.174 10.3.1.156 rocky9.localdomain > known_hosts
```

## C. Ajout des noeuds au cluster

![proof](proof%20on.png)