# II. 3. Setup réseau

**Ici on va préparer le réseau virtuel VXLAN que les VMs vont utiliser pour communiquer entre elles.**

Grâce à VXLAN, des VMs qui se situent sur des hyperviseurs différents pourront communiquer comme si elles étaient dans le même réseau local.

## Sommaire

- [II. 3. Setup réseau](#ii-3-setup-réseau)
  - [Sommaire](#sommaire)
  - [A. Intro](#a-intro)
  - [B. Création du Virtual Network](#b-création-du-virtual-network)
  - [C. Préparer le bridge réseau](#c-préparer-le-bridge-réseau)

## A. Intro

Pour le réseau, la plupart des étapes sont gérées par OpenNebula, notamment la création des endpoints VXLAN.

Il restera quelques étapes manuelles à effectuer, afin de monter un setup minimaliste avec des fonctionnalités attendues pour de la virtu :

- pouvoir contacter les VMs de l'extérieur, pour pouvoir s'y co en SSH par exemple
- avoir un réseau local entre les VMs pour qu'elles se joignent entre elles
- proposer un accès internet aux VMs

## B. Création du Virtual Network

➜ **RDV de nouveau sur la WebUI de OpenNebula, et naviguez dans `Network > Virtual Networks`**

➜ **Créer un nouveau Virtual Network, et renseignez :**

- un nom de réseau (ce que vous voulez)
- le mode VXLAN
- Onglet `Conf`
  - spécifiez `eth1` en interface réseau physique (l'interface qui a une IP statique sur votre machine)
  - définissez `vxlan_bridge` en nom de bridge (attention aux commandes plus bas si vous choisissez un autre nom)
- Onglet `Addresses`
  - spécifiez une IP de départ dans `First IPv4 address`, par exemple `10.220.220.1/24`
  - indiquez aussi ainsi qu'un nombre de machines possibles dans ce réseau avec `Size`
- Onglet `Context`
  - spécifier une adresse de réseau et un masque dans les champs dédiés, par exemple `10.220.220.0` et `255.255.255.0`

## C. Préparer le bridge réseau

➜ **Ces étapes sont à effectuer uniquement sur `kvm1.one`** dans un premier temps

- dans la partie IV du TP, quand vous mettrez en place `kvm2.one`, il faudra aussi refaire ça dessus

🌞 **Créer et configurer le bridge Linux**, j'vous file tout, suivez le guide :

```bash
# création du bridge
ip link add name vxlan_bridge type bridge

# on allume le bridge
ip link set dev vxlan_bridge up 

# on définit une IP sur cette interface bridge
ip addr add 10.220.220.201/24 dev vxlan_bridge

# ajout de l'interface bridge à la zone public de firewalld
firewall-cmd --add-interface=vxlan_bridge --zone=public --permanent

# activation du masquerading NAT dans cette zone
firewall-cmd --add-masquerade --permanent

# on reload le firewall pour que les deux commandes précédentes prennent effet
firewall-cmd --reload
```

```bash
[vagrant@kvm1 ~]$ sudo ip link add name vxlan_bridge type bridge
RTNETLINK answers: File exists
[vagrant@kvm1 ~]$ sudo ip link set dev vxlan_bridge up 
[vagrant@kvm1 ~]$ sudo ip addr add 10.220.220.201/24 dev vxlan_bridge
[vagrant@kvm1 ~]$ sudo firewall-cmd --add-interface=vxlan_bridge --zone=public --permanent
Warning: ALREADY_ENABLED: vxlan_bridge
success
[vagrant@kvm1 ~]$ sudo firewall-cmd --add-masquerade --permanent
Warning: ALREADY_ENABLED: masquerade
success
[vagrant@kvm1 ~]$ sudo firewall-cmd --reload
success