# Packet Tracer : Configuration de base d'un routeur (mode physique)

**Objectifs** : câbler la topologie, configurer R1 (sécurité, SSH, interfaces IPv4/IPv6), vérifier la connectivité, puis afficher les informations du routeur avec des commandes `show`.

https://github.com/richmondklab/configuration_basique_Routeur/blob/95ecdedf391584609edd8201779b92ecc4d57398/table%20d'addressage.png





---

## Partie 1 : Topologie

Câblage en cuivre droit entre R1, le commutateur 2960, le PC-A et le serveur. Câble console du PC-A vers R1.

**Capture : topologie**




---

## Partie 2 : Configuration

### Ordinateurs

PC-A et Serveur : Desktop > IP Configuration.

**Capture : IP du PC-A**




**Capture : IP du serveur**




### Routeur R1

```
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# ip domain-name ccna-lab.com
R1(config)# service password-encryption
R1(config)# security passwords min-length 12
R1(config)# username SSHadmin secret 55Hadm!n2020
R1(config)# crypto key generate rsa modulus 1024
R1(config)# enable secret $cisco!PRIV*
```

Console :

```
R1(config)# line console 0
R1(config-line)# password $cisco!!CON*
R1(config-line)# exec-timeout 4 0
R1(config-line)# login
R1(config-line)# exit
```

VTY (SSH uniquement) :

```
R1(config)# line vty 0 15
R1(config-line)# password $cisco!!VTY*
R1(config-line)# transport input ssh
R1(config-line)# exec-timeout 4 0
R1(config-line)# login local
R1(config-line)# exit
```

Bannière, IPv6 et blocage des tentatives :

```
R1(config)# banner motd #Acces non autorise strictement interdit.#
R1(config)# ipv6 unicast-routing
R1(config)# login block-for 120 attempts 3 within 60
```

Interfaces :

```
R1(config)# interface g0/0/0
R1(config-if)# description Lien vers Serveur
R1(config-if)# ip address 192.168.0.1 255.255.255.0
R1(config-if)# ipv6 address 2001:db8:acad::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface g0/0/1
R1(config-if)# description Lien vers PC-A
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# ipv6 address 2001:db8:acad:1::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface loopback 0
R1(config-if)# description Interface de bouclage
R1(config-if)# ip address 10.0.0.1 255.255.255.0
R1(config-if)# ipv6 address 2001:db8:acad:2::1/64
R1(config-if)# ipv6 address fe80::1 link-local
R1(config-if)# no shutdown
R1(config-if)# end
```

Horloge et sauvegarde :

```
R1# clock set <hh:mm:ss> <jour> <mois> <année>
R1# copy running-config startup-config
```

**Capture : configuration de R1**




**Redémarrage sans sauvegarde** : la running-config (RAM) est perdue ; R1 redémarre avec la startup-config, ou une configuration vide s'il n'y en a pas.

### Connectivité

Depuis le PC-A :

```
C:\> ping 192.168.0.10
C:\> ping 2001:db8:acad::10
C:\> ssh -l SSHadmin 10.0.0.1
C:\> ssh -l SSHadmin 2001:db8:acad:2::1
```

Mot de passe SSH : `55Hadm!n2020`

**Capture : pings**




**Capture : SSH IPv4**




**Capture : SSH IPv6**




**Réponses** : ping : oui / non ; SSH IPv4 : oui / non ; SSH IPv6 : oui / non *(à compléter)*.

**Risque de Telnet** : les identifiants et les données circulent en clair, donc interceptables ; SSH chiffre la session.

---

## Partie 3 : Informations du routeur

Connexion SSH à R1 (`2001:db8:acad:2::1`), utilisateur `SSHadmin`.

**Capture : connexion SSH**




### show version

```
R1# show version
R1# show version | include register
```

**Capture : show version**




**Capture : show version | include register**




- **Image IOS** : ligne `System image file is "flash:..."` *(à relever)*
- **NVRAM** : ligne `non-volatile configuration memory` *(à relever)*
- **Flash** : ligne `flash memory` *(à relever)*
- **Registre `0x2142`** : la startup-config est ignorée, R1 démarre avec une configuration vide (récupération de mot de passe).

### Configuration

```
R1# show startup-config
R1# show running-config | section vty
```

**Capture : show startup-config**




**Capture : section vty**




- **Mots de passe** : chiffrés ou hachés (type 7 pour les lignes, type 5/8/9 pour les `secret`).
- **Section vty** : affiche uniquement la configuration des lignes VTY.

### Table de routage

```
R1# show ip route
```

**Capture : show ip route**




- **Code d'un réseau connecté** : `C`
- **Nombre d'entrées `C`** : 3 (`192.168.0.0/24`, `192.168.1.0/24`, `10.0.0.0/24`), à confirmer sur ta capture

### Interfaces

```
R1# show ip interface brief
R1# show ipv6 interface brief
```

**Capture : show ip interface brief**




**Capture : show ipv6 interface brief**




- **Commande qui active les ports** : `no shutdown`
- **`[up/up]`** : état de la couche 1 (physique) / état de la couche 2 (protocole).

### Serveur en IPv6 automatique

Desktop > IP Configuration > IPv6 : **Automatic**, puis :

```
C:\> ipconfig
C:\> ping fe80::1
C:\> ping 2001:db8:acad::1
```

**Capture : ipconfig du serveur**




**Capture : pings IPv6**




- **Adresse IPv6 du serveur** : préfixe `2001:db8:acad::/64` + identifiant EUI-64 dérivé de la MAC *(à relever sur la capture)*
- **Passerelle par défaut** : `FE80::1` (link-local de R1)
- **Ping vers la passerelle** : oui / non *(à compléter ; l'énoncé cite un PC-B absent de la topologie)*
- **Ping vers `2001:db8:acad::1`** : oui / non *(à compléter)*

---

## Questions de réflexion

1. **Interface non activée** : `show ip interface brief` (Status = *administratively down*).
2. **Masque incorrect** : `show ip interface` ou `show running-config` (`show ip interface brief` n'affiche pas le masque).
