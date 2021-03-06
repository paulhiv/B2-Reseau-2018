# Procédures

Vous trouverez ici quelques mini-procédures pour réaliser certaines opérations récurrentes. Ce sera évidemment principalement utilisé pour notre cours de réseau, mais peut-être serez-vous amenés à le réutiliser plus tard.  

**Elles sont écrites pour un système Cisco**.  

C'est quoi un routeur Cisco ? C'est comme votre PC, un serveur, ou une machine virtuelle CentOS : un "ordinateur". Juste, le matériel et l'OS sont optimisés pour faire du routage. 

## Sommaire

* Tous les équipements
  * [Présentation du terminal Cisco](#présentation-du-terminal-cisco)
  * [Garder ses changements après reboot](#garder-les-changements-après-reboot)
  * [Changer son nom de domaine](#changer-son-nom-de-domaine)
  * [Gérer sa table ARP](#gérer-sa-table-arp)
* Routeurs
  * [Définir une IP statique](#définir-une-ip-statique)
  * [Ajouter une route statique](#ajouter-une-route-statique)
  * [OSPF](#ospf)
  * [NAT](#nat)
* Switches
  * [VLAN](#vlan)

---

## Tous les équipements

### Présentation du terminal Cisco

#### Les modes

Le terminal Cisco possède plusieurs modes :

Mode | Commande | What ? | Why ?
--- | --- | --- | ---
`user EXEC` | X | C'est le mode par défaut : il permet essentiellement de visualiser des choses, mais peu d'actions à réaliser | Pour visualiser les routes ou les IPs par ex
`privileged EXEC` | enable | Mode privilégié : permet de réalisé des actions privilégiées sur la machine | Peu utilisé dans notre cours au début
`global conf` | conf t | Configuration de la machine | Permet de configurer les interface et le routage 

L'idée globale c'est que pour **faire des choses** on passera en `global conf`, et on restera en **user EXEC** pour **voir** des choses.

#### Abbréviation/Complétion/Help

* Quand vous tapez une commande, vous pouvez tapez `?` pour avoir une aide sur la complétion de la commande (à n'importe quel moment).  
* L'auto-complétion avec TAB est toujours recommandée
* Toutes les commandes peuvent être abrégées quand il n'y a qu'un seul choix possible
  * mieux avec un exemple
  * `show ip interface brief` devient `sh ip int br` par exemple
 
#### Fonctionnalités

Il fait pas grand chose non plus le shell Cisco mais il est pas là pour ça. Cela dit :
* `ping` et `traceroute` évidemment !
* y'a `telnet` donc possible de tester des connexions arbitraires sur des ports
* y'a le `|` et y'a un équivalent de `grep` et ça c'est le :fire:
  * `show running config | s address`

#### Annuler un changement

Pour annuler un changement sur une machine Cisco, il suffit de taper la même commande et de la précéder par `no`. 
* mettre une IP
  * `ip address 10.1.1.10 255.255.255.0`
* enlever l'IP
  * `no ip address 10.1.1.10 255.255.255.0`

#### La commande `show`

La commande `show` permet de voir toute la configuration actuelle de la machine, par exemple : 
* toute la configuration actuelle `show running-config`
* toute la configuration "froide", utilisée lors du boot `show startup-config`
* la configuration IP des interfaces (sur un routeur)
  * `show ip interface brief`
  * `show ip interface`
* la configuration IP des interfaces (sur un switch)
  * `show interface switchport`
* `show ?` pour plus d'infoooos

### Garder les changements après reboot
Les équipements Cisco possèdent deux configurations :
* la `running-config`
  * c'est la conf actuelle
  * elle contient toutes vos modifications
  * `# show running-config` pour la voir
* la `startup-config`
  * c'est la conf qui est chargée au démarrage de la machine
  * elle ne contient aucune de vos modifications
  * `show startup-config`  
  
Comment garder vos changements à travers les reboots ? Il faut copier la `running-config` sur la `startup-config` :
```
# copy running-config startup-config
```

---

### Changer son nom de domaine
**1. Passer en mode configuration**
```
# conf t
```

**2. Changer le hostname**
```
(config)# hostname <HOSTNAME>
```

---

### Gérer sa table ARP

* voir sa table ARP sur un routeur
```
# show arp
```

* voir sa table CAM sur un switch
```
# show mac address-table
```

---

## Routeurs

### Définir une IP statique

**Routeur uniquement**  

**1. Repérer le nom de l'interface dont on veut changer l'IP**
```
# show ip interface brief
OU
# show ip int br
```
**2. Passer en mode configuration d'interface**
```
# conf t
(config)# interface ethernet <NUMERO>
```
**3. Définir une IP**
```
(config-if)# ip address <IP> <MASK>
Exemple :
(config-if)# ip address 10.5.1.254 255.255.255.0
```
**4. Allumer l'interface**
```
(config-if)# no shut
```
**5. Vérifier l'IP**
```
(config-if)# exit
(config)# exit
# show ip int br
```
---

### Ajouter une route statique

**Routeur uniquement**  

**1. Passer en mode configuration**
```
# conf t
```

**2.1. Ajouter une route vers un réseau**
```
(config)# ip route <REMOTE_NETWORK_ADDRESS> <MASK> <GATEWAY_IP> 
Exemple, pour ajouter une route vers le réseau 10.1.0.0/24 en passant par la passerelle 10.2.0.254
(config)# ip route 10.1.0.0 255.255.255.0 10.2.0.254 
```

**2.2. Ajouter la route par défaut**
```
(config)# ip route 0.0.0.0 0.0.0.0 10.2.0.254 
```

**3. Vérifier la route**
```
(config)# exit
# show ip route
```

### [OSPF](./3.md)

**Routeur uniquement**  

**1. Passer en mode configuration**
```
# conf t
```

**2. Activer OSPF**
```
(config)# router ospf 1
```
* le `1` correspond à l'ID de ce processus OSPF
  * ui on peut en faire tourner plusieurs
* **nous utiliserons toujours `1` pendant nos cours**

**3. Définir un `router-id`**
```
# dans nos TP, le router-id sera toujours le numéro du routeur répété 4 fois
# donc 1.1.1.1 pour router1
(config-router)# router-id 1.1.1.1
```

**4. Partager une ou plusieurs routes**
```
(config-router)# network 10.6.100.0 0.0.0.3 area 0
```
* cette commande partage le réseau `10.6.100.0/30` avec les voisins OSPF, et indique que ce réseau est dans l'`area 0` (l'aire "backbone")
* l'utilisation de cette commande est un peu particulière concernant le masque de sous-réseau
  * **vous devez écrire l'inverse de d'habitude, en binaire**
  * c'est à dire `0.0.0.3` au lieu de `255.255.255.252` par exemple
  * c'est un "wildcard mask" au lieu de juste un "mask"

**Vérifier l'état d'OSPF** :
```
# show ip protocols
# show ip ospf interface
# show ip ospf neigh
# show ip ospf border-routers
```

---

### [NAT](./2.md#concept-et-utilisation-du-nat)

**Routeur uniquement**  

**1. Passer en mode configuration**
```
# conf t
```

**2. Définir les interfaces internes et externes pour le NAT** :
* l'interface externe est celle qui pointe vers le WAN
* les interfaces internes sont celles qui pointe vers les LANs qui doivent accéder à internet
```
(config)# interface fastEthernet 0/0
(config-if)# ip nat outside
(config-if)# exit

(config)# interface fastEthernet 1/0
(config-if)# ip nat inside
(config-if)# exit
```

**3. Activation du NAT** et création d'une *access-list* très permissive qui autorise tout le trafic
```
(config)# ip nat inside source list 1 interface fastEthernet0/0 overload
(config)# access-list 1 permit any
```

**4. Vérification**
```
# ping 8.8.8.8
```

---
## Switches

### VLAN

[Le cours possède un passage détaillé à ce sujet.](./3.md#vlan)

**Switches uniquement**  

**0. Pour voir les VLANs actuels du switch**
```
# show vlan
# show vlan br
```

**1. Passer en mode configuration**
```
# conf t
```

**2. Créer le VLAN**
  * le nom est arbitraire, essayez d'utiliser des noms clairs
```
(config)# vlan 10
(config-vlan)# name client-network
(config-vlan)# exit
```

**3. Assigner une interface pour donner accès à un VLAN**. C'est le port où sera branché le client. **C'est le mode *access*.**
```
(config)# interface Ethernet 0/0
(config-if)# switchport mode access
(config-if)# switchport access vlan 10
```

**4. Configurer une interface entre deux switches. C'est le mode *trunk*.**
```
(config)# interface Ethernet 0/0
(config-if)# switchport trunk encapsulation dot1q
(config-if)# switchport mode trunk
```
