# Packet Tracer : Configuration de base d'un routeur (mode physique)

## Objectifs

- **Partie 1** : Configuration de la topologie et initialisation des appareils
- **Partie 2** : Configuration des périphériques et vérification de la connectivité
- **Partie 3** : Affichage des informations du routeur

## Contexte / scénario

Activité complète en mode physique de Packet Tracer (PTPM) pour réviser les commandes IOS. Dans les parties 1 et 2, le matériel est câblé et le routeur reçoit une configuration de base (sécurité, SSH, interfaces). Dans la partie 3, une session SSH sert à récupérer des informations du routeur avec des commandes `show`.

> **Table d'adressage** 

| Appareil | Interface | Adresse IPv4 | Masque | Adresse IPv6 / préfixe | Passerelle par défaut |
|----------|-----------|--------------|--------|------------------------|-----------------------|
| R1 | G0/0/0 | `192.168.0.1 /24` | `N/A` | `192.168.0.1 /24` |  	
fe80::1|
| R1 | G0/0/1 | `<IPv4>` | `<masque>` | `<IPv6>/64` | N/A |
| R1 | Loopback0 | `<IPv4>` | `<masque>` | `<IPv6>/64` | N/A |
| PC-A | NIC | `<IPv4>` | `<masque>` | `<IPv6>/64` | `<passerelle>` |
| Serveur | NIC | `<IPv4>` | `<masque>` | `<IPv6>/64` | `<passerelle>` |

---

## Partie 1 : Configurer la topologie et initialiser les périphériques

### Étape 1 : Câbler le réseau

1. Glisser le **Cisco 4321 ISR**, le **commutateur 2960** et le **serveur** de l'étagère vers le rack.
2. Glisser le **PC** de l'étagère vers le tableau.
3. Câbler les appareils selon la topologie avec des **câbles directs en cuivre**.
4. Relier le PC au Cisco 4321 ISR avec un **câble de console**.
5. Mettre sous tension le routeur, le PC-A et le serveur (le commutateur 2960 s'allume automatiquement).

**Capture : topologie câblée**




---

## Partie 2 : Configurer les périphériques et vérifier la connectivité

### Étape 1 : Configurer les interfaces des ordinateurs

**PC-A** : Desktop > IP Configuration (adresse IP, masque, passerelle par défaut).

**Capture : configuration IP du PC-A**




**Serveur** : Desktop > IP Configuration (adresse IP, masque, passerelle par défaut).

**Capture : configuration IP du serveur**




### Étape 2 : Configurer le routeur

Accès via le terminal (Desktop > Terminal) depuis le PC-A.

#### a. Mode d'exécution privilégié

```
Router> enable
```

#### b. Mode de configuration

```
Router# configure terminal
```

#### c. Nom du périphérique

```
Router(config)# hostname R1
```

#### d. Nom de domaine

```
R1(config)# ip domain-name ccna-lab.com
```

#### e. Chiffrement des mots de passe en clair

```
R1(config)# service password-encryption
```

#### f. Longueur minimale des mots de passe (12 caractères)

```
R1(config)# security passwords min-length 12
```

#### g. Utilisateur SSH avec mot de passe chiffré

```
R1(config)# username SSHadmin secret 55Hadm!n2020
```

#### h. Clés de chiffrement RSA (module de 1024 bits)

```
R1(config)# crypto key generate rsa modulus 1024
R1(config)# ip ssh version 2
```

#### i. Mot de passe du mode privilégié

```
R1(config)# enable secret $cisco!PRIV*
```

#### j. Console : mot de passe, déconnexion après 4 min, connexion activée

```
R1(config)# line console 0
R1(config-line)# password $cisco!!CON*
R1(config-line)# exec-timeout 4 0
R1(config-line)# login
R1(config-line)# exit
```

#### k. Lignes VTY : SSH uniquement, déconnexion après 4 min, base de données locale

```
R1(config)# line vty 0 15
R1(config-line)# password $cisco!!VTY*
R1(config-line)# transport input ssh
R1(config-line)# exec-timeout 4 0
R1(config-line)# login local
R1(config-line)# exit
```

#### l. Bannière d'avertissement

```
R1(config)# banner motd #
Acces non autorise strictement interdit. Toute tentative sera poursuivie. #
```

#### m. Routage IPv6

```
R1(config)# ipv6 unicast-routing
```

#### n. Interfaces (IPv4, IPv6, descriptions, activation)

```
R1(config)# interface g0/0/0
R1(config-if)# description Lien vers le commutateur / PC-A
R1(config-if)# ip address <IPv4> <masque>
R1(config-if)# ipv6 address <IPv6>/64
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface g0/0/1
R1(config-if)# description Lien vers le serveur
R1(config-if)# ip address <IPv4> <masque>
R1(config-if)# ipv6 address <IPv6>/64
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface loopback 0
R1(config-if)# description Interface de bouclage
R1(config-if)# ip address <IPv4> <masque>
R1(config-if)# ipv6 address <IPv6>/64
R1(config-if)# no shutdown
R1(config-if)# exit
```

**Blocage des connexions VTY** : 2 minutes après 3 échecs en 1 minute.

```
R1(config)# login block-for 120 attempts 3 within 60
```

#### o. Réglage de l'horloge

```
R1(config)# end
R1# clock set <hh:mm:ss> <jour> <mois> <année>
```

Exemple : `clock set 14:30:00 3 October 2026`

#### p. Sauvegarde de la configuration

```
R1# copy running-config startup-config
```

**Capture : configuration du routeur**




**Question : quel serait le résultat du redémarrage du routeur avant l'exécution de `copy running-config startup-config` ?**

> **Réponse :** La configuration en cours (running-config, en RAM) est perdue. Le routeur redémarre avec la configuration initiale (startup-config en NVRAM), ou avec une configuration vide s'il n'en existe aucune. Toutes les modifications non sauvegardées disparaissent.

### Étape 3 : Vérifier la connectivité du réseau

#### a. Ping depuis le PC-A vers le serveur (IPv4 et IPv6)

```
C:\> ping <IPv4 du serveur>
C:\> ping <IPv6 du serveur>
```

**Capture : résultats des ping**




**Question : les requêtes ping ont-elles abouti ?**

> **Réponse :** Oui / Non (à compléter selon ta capture).

#### b. Accès SSH à l'adresse IPv4 de la loopback de R1

```
C:\> ssh -l SSHadmin <IPv4 loopback R1>
Password: 55Hadm!n2020
```

**Capture : session SSH IPv4**




**Question : l'accès distant a-t-il abouti ?**

> **Réponse :** Oui / Non (à compléter).

#### c. Accès SSH à l'adresse IPv6 de la loopback de R1

```
C:\> ssh -l SSHadmin <IPv6 loopback R1>
Password: 55Hadm!n2020
```

**Capture : session SSH IPv6**




**Question : l'accès distant a-t-il abouti ?**

> **Réponse :** Oui / Non (à compléter).

**Question : pourquoi Telnet est-il considéré comme un risque de sécurité ?**

> **Réponse :** Telnet transmet toutes les données (identifiants et mots de passe compris) **en clair**, sans chiffrement. Un attaquant qui capture le trafic (sniffing, attaque de type man-in-the-middle) peut lire les identifiants et prendre le contrôle de l'appareil. SSH chiffre la session.

---

## Partie 3 : Afficher les informations du routeur

### Étape 1 : Session SSH vers R1

```
C:\> ssh -l SSHadmin <IPv6 loopback R1>
Password: 55Hadm!n2020
```

**Capture : connexion SSH établie**




### Étape 2 : Informations matérielles et logicielles

#### a. `show version`

```
R1# show version
```

**Capture : sortie de `show version`**




**Question : quel est le nom de l'image IOS exécutée par le routeur ?**

> **Réponse :** (à relever dans la ligne `System image file is "flash:..."`)

**Question : quelle quantité de NVRAM le routeur possède-t-il ?**

> **Réponse :** (à relever dans la ligne `... bytes of non-volatile configuration memory`)

**Question : quelle quantité de mémoire Flash le routeur possède-t-il ?**

> **Réponse :** (à relever dans la ligne `... bytes of physical memory` / `... bytes of flash memory`)

#### b. Filtrage de la sortie

```
R1# show version | include register
```

**Capture : sortie filtrée**




**Question : quel serait le processus de démarrage lors du prochain rechargement si le registre de configuration était `0x2142` ?**

> **Réponse :** Avec `0x2142`, le routeur **ignore la startup-config** en NVRAM. Il charge l'IOS puis démarre avec une configuration vide (assistant de configuration initiale). Cette valeur sert notamment à la récupération de mot de passe.

### Étape 3 : Configuration initiale

#### a. `show startup-config`

```
R1# show startup-config
```

**Capture : sortie de `show startup-config`**




**Question : comment les mots de passe sont-ils présentés dans les résultats ?**

> **Réponse :** Ils apparaissent **chiffrés / hachés** (et non en clair) : `enable secret` et `username ... secret` sont hachés (type 5, 8 ou 9), les mots de passe de ligne sont chiffrés (type 7) grâce à `service password-encryption`.

#### b. Section VTY

```
R1# show running-config | section vty
```

**Capture : sortie de la commande**




**Question : quel est le résultat de l'exécution de cette commande ?**

> **Réponse :** Elle n'affiche que la partie de la configuration relative aux lignes VTY (`line vty 0 15` avec mot de passe chiffré, `exec-timeout 4 0`, `login local`, `transport input ssh`).

### Étape 4 : Table de routage

```
R1# show ip route
```

**Capture : table de routage**




**Question : quel code indique un réseau connecté directement ?**

> **Réponse :** **C**

**Question : combien d'entrées sont codées avec le code C ?**

> **Réponse :** (à compter sur ta capture : une entrée `C` par réseau directement connecté, soit 3 si les trois interfaces sont actives)

### Étape 5 : Récapitulatif des interfaces

#### a. `show ip interface brief`

```
R1# show ip interface brief
```

**Capture : sortie de la commande**




**Question : quelle commande a fait passer les ports Gigabit Ethernet de *administratively down* à *up* ?**

> **Réponse :** `no shutdown`

#### b. `show ipv6 interface brief`

```
R1# show ipv6 interface brief
```

**Capture : sortie de la commande**




**Question : quelle est la signification de `[up/up]` ?**

> **Réponse :** Le premier `up` est l'état de la **couche 1** (statut physique de l'interface). Le second `up` est l'état de la **couche 2** (protocole de liaison de données). Les deux `up` signifient que l'interface est opérationnelle.

#### c. Configuration IPv6 automatique du serveur

Sur le serveur : Desktop > IP Configuration > IPv6 Configuration > **Automatic** (suppression de l'adresse statique), puis :

```
C:\> ipconfig
```

**Capture : sortie de `ipconfig` sur le serveur**




**Question : quelle est l'adresse IPv6 affectée au serveur ?**

> **Réponse :** (à relever : adresse construite avec le préfixe annoncé par R1 et l'identifiant d'interface EUI-64)

**Question : quelle est la passerelle par défaut attribuée au serveur ?**

> **Réponse :** (à relever : adresse **link-local** `FE80::...` de l'interface de R1)

**Question : depuis le PC-B, le ping vers l'adresse link-local de la passerelle R1 a-t-il abouti ?**

> **Réponse :** (à compléter). *Remarque : l'énoncé mentionne un PC-B, alors que la topologie ne contient que PC-A et le serveur.*

```
C:\> ping <FE80::... de R1>
```

**Capture : ping vers la passerelle**




**Question : depuis le serveur, le ping vers R1 à l'adresse `2001:db8:acad::1` a-t-il abouti ?**

```
C:\> ping 2001:db8:acad::1
```

> **Réponse :** (à compléter)

**Capture : ping vers 2001:db8:acad::1**




---

## Questions de réflexion

**1. Quelle commande `show` permet de vérifier si une interface n'a pas été activée ?**

> **Réponse :** `show ip interface brief` (colonne Status : *administratively down* si l'interface est désactivée). On peut aussi utiliser `show interfaces`.

**2. Quelle commande `show` permet de vérifier un masque de sous-réseau incorrect sur une interface ?**

> **Réponse :** `show ip interface` ou `show running-config` (section de l'interface) ; `show interfaces` affiche aussi l'adresse avec son préfixe. `show ip interface brief` n'affiche **pas** le masque.

---

## Récapitulatif des commandes utilisées

| Commande | Rôle |
|----------|------|
| `enable` / `configure terminal` | Accès aux modes privilégié et de configuration |
| `hostname` | Nom du routeur |
| `ip domain-name` | Nom de domaine (requis pour les clés RSA) |
| `service password-encryption` | Chiffre les mots de passe en clair |
| `security passwords min-length` | Longueur minimale des mots de passe |
| `username ... secret` | Utilisateur local avec mot de passe haché |
| `crypto key generate rsa modulus 1024` | Génère les clés RSA pour SSH |
| `enable secret` | Mot de passe du mode privilégié |
| `line console 0` / `line vty 0 15` | Configuration des accès console et distants |
| `transport input ssh` | Autorise uniquement SSH |
| `exec-timeout` | Délai d'inactivité avant déconnexion |
| `login` / `login local` | Active l'authentification |
| `banner motd` | Bannière d'avertissement |
| `ipv6 unicast-routing` | Active le routage IPv6 |
| `ip address` / `ipv6 address` | Adressage des interfaces |
| `description` / `no shutdown` | Description et activation d'interface |
| `login block-for 120 attempts 3 within 60` | Bloque les connexions après des échecs répétés |
| `clock set` | Réglage de l'horloge |
| `copy running-config startup-config` | Sauvegarde de la configuration |
| `show version` | Informations matérielles et logicielles |
| `show startup-config` / `show running-config` | Affichage des configurations |
| `show ip route` | Table de routage |
| `show ip interface brief` / `show ipv6 interface brief` | État des interfaces |
