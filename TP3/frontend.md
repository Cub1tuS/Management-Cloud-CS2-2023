# II.1. Setup Frontend

## Sommaire

- [II.1. Setup Frontend](#ii1-setup-frontend)
  - [Sommaire](#sommaire)
  - [A. Database](#a-database)
  - [B. OpenNebula](#b-opennebula)
  - [C. Conf système](#c-conf-système)
  - [D. Test](#d-test)

## A. Database

🌞 **Installer un serveur MySQL**

```bash
wget https://dev.mysql.com/get/mysql80-community-release-el9-5.noarch.rpm
```

```bash
sudo rpm -i mysql80-community-release-el9-5.noarch.rpm
```

```bash
sudo dnf install mysql-community-server
```

🌞 **Démarrer le serveur MySQL**

```bash
systemctl enable mysqld
```

```bash
systemctl start mysqld
```

🌞 **Setup MySQL**

```bash
sudo cat /var/log/mysqld.log 
```

```bash
[...]
e2024-04-09T10:43:53.368117Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: AH1fUjfi0<9>
[...]
```

```bash
mysql -uroot -pAH1fUjfi0<9>
```

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Jefaisunstrongpassword123!';
CREATE USER 'oneadmin' IDENTIFIED BY 'Jefaisunstrongpassword123!';
CREATE DATABASE opennebula;
GRANT ALL PRIVILEGES ON opennebula.* TO 'oneadmin';
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

## B. OpenNebula

🌞 **Ajouter les dépôts Open Nebula**

```bash
[vagrant@frontend yum.repos.d]$ nano opennebula.repo
```

> J'ajoute le contenu suivant dans "opennebula.repo"

```
[opennebula]
name=OpenNebula Community Edition
baseurl=https://downloads.opennebula.io/repo/6.8/RedHat/$releasever/$basearch
enabled=1
gpgkey=https://downloads.opennebula.io/repo/repo2.key
gpgcheck=1
repo_gpgcheck=1
```

```bash
dnf makecache -y
```

🌞 **Installer OpenNebula**

```bash
dnf install -y opennebula opennebula-sunstone opennebula-fireedge
```

🌞 **Configuration OpenNebula**

- dans le fichier `/etc/one/oned.conf`, définissez correctement les paramètres de connexion à la base de données, c'est la clause `DB =` que vous devez remplacer par

```conf
DB = [ BACKEND = "mysql",
       SERVER  = "localhost",
       PORT    = 0,
       USER    = "oneadmin",
       PASSWD  = "Jefaisunstrongpassword123! ",
       DB_NAME = "opennebula",
       CONNECTIONS = 25,
       COMPARE_BINARY = "no" ]
```

🌞 **Créer un user pour se log sur la WebUI OpenNebula**

- pour ça, il faut se log en tant que l'utilisateur `oneadmin` sur le serveur
```bash
sudo su oneadmin
```

```bash
nano /var/lib/one/.one/one_auth
```
> toto:password

🌞 **Démarrer les services OpenNebula**

- démarrez les services `opennebula`, `opennebula-sunstone`
- activez-les aussi au démarrage de la machine

```bash
[vagrant@frontend]$ sudo systemctl enable opennebula
[vagrant@frontend]$ sudo systemctl start opennebula
[vagrant@frontend]$ sudo systemctl enable opennebula-sunstone
[vagrant@frontend]$ sudo systemctl start opennebula-sunstone
```

## C. Conf système

🌞 **Ouverture firewall**

```bash
sudo firewall-cmd --permanent --add-port=9869/tcp
sudo firewall-cmd --permanent --add-port=22/tcp
sudo firewall-cmd --permanent --add-port=2633/tcp
sudo firewall-cmd --permanent --add-port=4124/tcp
sudo firewall-cmd --permanent --add-port=4124/udp
sudo firewall-cmd --permanent --add-port=29876/tcp
```

- ouvrez les ports suivants, avec des commandes `firewall-cmd` :

| Port  | Proto      | Why ?                        |
|-------|------------|------------------------------|
| 9869  | TCP        | WebUI (Sunstone)             |
| 22    | TCP        | SSH                          |
| 2633  | TCP        | Daemon `oned` et API XML RPC |
| 4124  | TPC et UDP | Monitoring                   |
| 29876 | TCP        | NoVNC proxy                  |

## D. Test

A ce stade, vous devriez déjà pouvoir **visiter la WebUI de OpenNebula.**

**RDV sur `http://10.3.1.11:9869`** avec un navigateur ! Vous pouvez vous log avec les identifiants définis dans la partie B.
