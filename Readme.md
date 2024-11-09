# PARTIE 2: Etude des VPN et mise en place d'une nouvelle instance
## QUESTION 2.1:

Effectuez une capture de trame entre P2 et P4 en utilisant la méthode décrite dans les annexes, lancez le trafic du VPN Green. Décrivez le contenu d'un des paquets associé au trafic client dans le VPN (Couches OSI, Labels, etc...).

Commande utilisée :
```bash
sudo bash utilis/traffic.sh start GREEN
```

---

### Analyse des trames : Trame 278 et Trame 279 (Trafic VPN Green)

Pour analyser le trafic VPN Green entre P2 et P4, deux trames consécutives, la Trame 278 et la Trame 279, ont été capturées.
On utilisant la commande:

<img width="600" alt="capturetrafficbetweenp2andp4indirectionofPE1toPE2" src="https://github.com/user-attachments/assets/95b74ec0-2069-4393-a173-3c9e2714fe32">

Et la résultat été:

<img width="474" alt="mplslabels" src="https://github.com/user-attachments/assets/9f08fb1f-2b38-40d0-90f7-e172e1a94e96">


Les deux trames appartiennent à la même session TCP, comme indiqué par les adresses MAC et IP cohérentes. Les couches suivantes ont été analysées selon le modèle OSI, avec un accent sur les labels MPLS et leur usage.

---

#### 1. **Couches Physique et Liaison de données (Couches 1 et 2)**
   - **Longueur de la trame** : Les deux trames mesurent 74 octets.
   - **Adresses MAC** :
     - **Source** : `aa:c1:ab:3d:be:22` (adresse administrée localement).
     - **Destination** : `aa:c1:ab:b9:0a:86` (adresse administrée localement).
   - **Type d'encapsulation** : Ethernet II, typique pour les réseaux TCP/IP.
   - **Ethertype** : Paquet commuté par label MPLS (`0x8847`), indiquant une encapsulation MPLS au niveau de la couche Liaison de données.

#### 2. **Couche Réseau (Couche 3) - En-têtes MPLS et IP**
   - **En-têtes MPLS** : Les deux trames contiennent deux en-têtes MPLS.
     - **Premier en-tête MPLS** :
       - **Label** : 24006 – Ce label est utilisé à des fins de **transport**, facilitant les décisions de routage dans le réseau MPLS.
       - **Bits expérimentaux (Exp)** : 0 – Pas de priorité définie.
       - **Bottom of Stack (S)** : 0 – Indique qu'il y a des labels supplémentaires dans la pile.
       - **Time to Live (TTL)** : 62 – Durée de vie initiale pour l'encapsulation MPLS.
     - **Deuxième en-tête MPLS** :
       - **Label** : 24012 – Ce label est associé à la **différenciation de service** pour le trafic VPN Green, distinguant le trafic VPN des autres services.
       - **Bits expérimentaux (Exp)** : 0.
       - **Bottom of Stack (S)** : 1 – Dernier label dans la pile MPLS.
       - **Time to Live (TTL)** : 63.
   - **En-tête IP** :
     - **IP Source** : `10.0.2.10`
     - **IP Destination** : `10.0.1.10`
     - **Flags** : Le drapeau "Don't Fragment" est activé, indiquant que la fragmentation n'est pas autorisée.
     - **Time to Live (TTL)** : 63, suggérant un nombre minimal de sauts entre les routeurs.

#### 3. **Couche Transport (Couche 4) - TCP**
   - **Port source** : 5201
   - **Port destination** : 55227
   - **Numéros de séquence et d'acquittement** :
     - **Trame 278** : Numéro d'ACK `279425`.
     - **Trame 279** : Numéro d'ACK `282305`, indiquant l'acquittement du segment précédent.
   - **Taille de fenêtre** : 271 pour les deux trames.
   - **Drapeaux** : Le drapeau ACK est activé, sans données de charge utile dans ces paquets.

---

### Résumé et Observations
- Les deux trames analysées font partie de la même session TCP, avec des adresses source et destination cohérentes aux niveaux MAC et IP.
- **Labels MPLS** :
  - Le **premier label MPLS (24006)** est utilisé pour le **transport** au sein du réseau MPLS.
  - Le **deuxième label MPLS (24012)** est désigné pour la **différenciation de service**, en particulier pour le trafic VPN Green.
- **Informations IP et TCP** : Les deux paquets utilisent les mêmes adresses IP et ports, avec un léger incrément dans les numéros d’acquittement, indiquant la continuité de la session.

Cette analyse de paquet confirme la présence de labels MPLS distinguant les couches de transport et de service pour le VPN Green, permettant un flux sans interruption de PE1 vers PE2. Cette structure assure une gestion et une segmentation efficaces du trafic VPN dans le réseau MPLS.
## QUESTION 2.2:
## QUESTION 2.3:
### Ajout d'une nouvelle instance VPN sur l'infrastructure

Dans cette section, nous allons configurer un nouveau VPN entre PE1 et PE2, en créant une instance VPN "ORANGE". Vous pouvez vous référer aux VPNs existants (GREEN et RED) pour adapter les commandes nécessaires. Nous allons procéder à la configuration, vérifier la validité de l'instance via des commandes `ping` et `traceroute`, puis tester la connectivité avec le générateur de trafic.

## 1. Configuration de l'instance VPN "ORANGE"

### Sur le routeur **PE1**:

#### 1.1. Création de l'instance VRF "ORANGE"

Tout d'abord, nous créons la nouvelle instance VRF sur **PE1**. La VRF permet de séparer les tables de routage pour différents VPNs.

# Création de la VRF "ORANGE" sur PE1
```bash
RP/0/RP0/CPU0:PE1(config-vrf)#vrf ORANGE
RP/0/RP0/CPU0:PE1(config-vrf)#rd 172.30.0.1:3  # Route Distinguisher, identifie cette VRF
RP/0/RP0/CPU0:PE1(config-vrf)#address-family ipv4 unicast  # Spécification de l'adresse IPv4 pour la VRF
RP/0/RP0/CPU0:PE1(config-vrf-af)#import route-target 65000:128  # Importation des routes depuis un autre VRF
RP/0/RP0/CPU0:PE1(config-vrf-af)#export route-target 65000:128  # Exportation des routes vers un autre VRF
RP/0/RP0/CPU0:PE1(config-vrf-af)#commit  # Validation de la configuration
```

#### 1.2. Configuration de l'interface GigabitEthernet0/0/0/3.30 et assignation de la VRF "ORANGE"
Ensuite, nous configurons une interface physique ou sous-interface pour le VPN, et nous y assignons la VRF "ORANGE".
# Configuration de l'interface GigabitEthernet0/0/0/3.30 sur PE1
```bash
RP/0/RP0/CPU0:PE1(config)#interface GigabitEthernet0/0/0/3.30
RP/0/RP0/CPU0:PE1(config-subif)#vrf ORANGE  # Attribution de la VRF à l'interface
RP/0/RP0/CPU0:PE1(config-subif)#ip address 30.0.1.1 255.255.255.0  # Attribution de l'adresse IP
RP/0/RP0/CPU0:PE1(config-subif)#commit  # Validation de la configuration
```
#### 1.3. Configuration du BGP pour la VRF "ORANGE"
Enfin, nous configurons le protocole BGP pour échanger des routes entre les VRFs.
# Configuration BGP pour la VRF ORANGE sur PE1
```bash
RP/0/RP0/CPU0:PE1(config)#router bgp 65000  # Configuration du BGP avec le numéro de système autonome
RP/0/RP0/CPU0:PE1(config-router)#address-family vpnv4 unicast  # Configuration du BGP VPNv4
RP/0/RP0/CPU0:PE1(config-router-af)#address-family ipv4 unicast  # Configuration du BGP IPv4
RP/0/RP0/CPU0:PE1(config-router-af)#redistribute connected  # Redistribution des routes connectées
RP/0/RP0/CPU0:PE1(config-router-af)#redistribute static  # Redistribution des routes statiques
RP/0/RP0/CPU0:PE1(config-router-af)#redistribute rip  # Redistribution des routes RIP
RP/0/RP0/CPU0:PE1(config-router-af)#commit  # Validation de la configuration
```
### Sur le routeur **PE2**:
### 2.1. Création de l'instance VRF "ORANGE"
La même configuration VRF est appliquée sur PE2.
# Création de la VRF "ORANGE" sur PE2
```bash
RP/0/RP0/CPU0:PE2(config-vrf)#vrf ORANGE
RP/0/RP0/CPU0:PE2(config-vrf)#rd 172.30.0.6:3  # Route Distinguisher pour PE2
RP/0/RP0/CPU0:PE2(config-vrf)#address-family ipv4 unicast  # Spécification de l'adresse IPv4
RP/0/RP0/CPU0:PE2(config-vrf-af)#import route-target 65000:128  # Importation des routes depuis un autre VRF
RP/0/RP0/CPU0:PE2(config-vrf-af)#export route-target 65000:128  # Exportation des routes vers un autre VRF
RP/0/RP0/CPU0:PE2(config-vrf-af)#commit  # Validation de la configuration
```
### 2.2. Configuration de l'interface GigabitEthernet0/0/0/3.30 et assignation de la VRF "ORANGE"
Sur PE2, nous configurons également l'interface et y assignons la VRF "ORANGE".
# Configuration de l'interface GigabitEthernet0/0/0/3.30 sur PE2
```bash
RP/0/RP0/CPU0:PE2(config)#interface GigabitEthernet0/0/0/3.30
RP/0/RP0/CPU0:PE2(config-subif)#vrf ORANGE  # Attribution de la VRF à l'interface
RP/0/RP0/CPU0:PE2(config-subif)#ip address 30.0.2.1 255.255.255.0  # Attribution de l'adresse IP
RP/0/RP0/CPU0:PE2(config-subif)#commit  # Validation de la configuration
```
### 2.3. Configuration du BGP pour la VRF "ORANGE"
Nous appliquons la même configuration BGP sur PE2.
```
# Configuration BGP pour la VRF ORANGE sur PE2
RP/0/RP0/CPU0:PE2(config)#router bgp 65000
RP/0/RP0/CPU0:PE2(config-router)#address-family vpnv4 unicast
RP/0/RP0/CPU0:PE2(config-router-af)#address-family ipv4 unicast
RP/0/RP0/CPU0:PE2(config-router-af)#redistribute connected
RP/0/RP0/CPU0:PE2(config-router-af)#redistribute static
RP/0/RP0/CPU0:PE2(config-router-af)#redistribute rip
RP/0/RP0/CPU0:PE2(config-router-af)#commit
```
# 2. Vérification de la configuration
### 2.1. Vérification de la table BGP sur PE1
Lancez la commande suivante sur PE1 pour vérifier que les routes sont bien échangées via BGP :
```show bgp vpnv4 unicast vrf ORANGE```
La sortie donne:
```
Sat Nov  9 18:15:23.773 UTC
BGP router identifier 172.30.0.1, local AS number 65000
BGP generic scan interval 60 secs
Non-stop routing is enabled
BGP table state: Active
Table ID: 0x0
BGP main routing table version 18
BGP NSR Initial initsync version 13 (Reached)
BGP NSR/ISSU Sync-Group versions 0/0
BGP scan interval 60 secs

Status codes: s suppressed, d damped, h history, * valid, > best
              i - internal, r RIB-failure, S stale, N Nexthop-discard
Origin codes: i - IGP, e - EGP, ? - incomplete
   Network            Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 172.30.0.1:3 (default for vrf ORANGE)
Route Distinguisher Version: 18
*> 30.0.1.0/24        0.0.0.0                  0         32768 ?
*>i30.0.2.0/24        172.30.0.6               0    100      0 ?

Processed 2 prefixes, 2 paths
```
### 2.2. Vérification de la table de routage sur PE1
Vérifiez que les routes sont correctement installées dans la table de routage de la VRF "ORANGE" avec la commande :
```show ip route vrf ORANGE```
La sortie est :
```
RP/0/RP0/CPU0:PE1#show ip route vrf ORANGE
Sat Nov  9 18:16:04.175 UTC

Codes: C - connected, S - static, R - RIP, B - BGP, (>) - Diversion path
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - ISIS, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, su - IS-IS summary null, * - candidate default
       U - per-user static route, o - ODR, L - local, G  - DAGR, l - LISP
       A - access/subscriber, a - Application route
       M - mobile route, r - RPL, t - Traffic Engineering, (!) - FRR Backup path

Gateway of last resort is not set

C    30.0.1.0/24 is directly connected, 01:41:03, GigabitEthernet0/0/0/3.30
L    30.0.1.1/32 is directly connected, 01:41:03, GigabitEthernet0/0/0/3.30
B    30.0.2.0/24 [200/0] via 172.30.0.6 (nexthop in vrf default), 00:26:42
```

## 2.3. Vérification de la configuration VRF sur PE1 et PE2
### Sur PE1, vérifiez la configuration de la VRF "ORANGE" :
```show running-config vrf ORANGE```
### Sur PE2, vérifiez la configuration de la VRF "ORANGE" :
```show running-config vrf ORANGE```

Résultat des deux donne ça:

![image](https://github.com/user-attachments/assets/a5d4facf-c883-4bb9-bcd9-61afc919f260)

Les configurations sont similaires.
## 2.4. Test de connectivité avec ping:
Lancer la commande:
```sudo docker exec -it Client ping 30.0.2.10```

Résultat:

![image](https://github.com/user-attachments/assets/3724af06-a445-4c40-a6e8-7989611a1512)

## 2.5. Lancer le générateur de trafic
Une fois le ping fonctionnel, vous pouvez lancer le générateur de trafic pour tester la connectivité du réseau. 
avec cette commande: ```sudo bash utilis/traffic.sh start ORANGE``` qui donne effictivement cette résultat:

![image](https://github.com/user-attachments/assets/f81ff202-ef44-4884-bea3-618c1d3176bc)

# Conclusion
La configuration de l'instance VPN "ORANGE" a été réalisée avec succès sur les routeurs PE1 et PE2. Les tests de connectivité, incluant les commandes ping et traceroute, ont confirmé que le VPN est opérationnel et que les routes sont correctement échangées entre les deux points. 
