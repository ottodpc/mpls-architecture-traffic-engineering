Sure! Here's a well-structured and commented version of the configuration for adding a new VPN instance in your infrastructure, ready for your `README.md`. The commands are formatted for easy copy-pasting, and comments explain each step clearly.

```markdown
# Ajout d'une nouvelle instance VPN sur l'infrastructure

Dans cette section, nous allons configurer un nouveau VPN entre PE1 et PE2, en créant une instance VPN "ORANGE". Vous pouvez vous référer aux VPNs existants (GREEN et RED) pour adapter les commandes nécessaires. Nous allons procéder à la configuration, vérifier la validité de l'instance via des commandes `ping` et `traceroute`, puis tester la connectivité avec le générateur de trafic.

## 1. Configuration de l'instance VPN "ORANGE"

### Sur le routeur **PE1**:

#### 1.1. Création de l'instance VRF "ORANGE"

Tout d'abord, nous créons la nouvelle instance VRF sur **PE1**. La VRF permet de séparer les tables de routage pour différents VPNs.

```bash
# Création de la VRF "ORANGE" sur PE1
RP/0/RP0/CPU0:PE1(config-vrf)#vrf ORANGE
RP/0/RP0/CPU0:PE1(config-vrf)#rd 172.30.0.1:3  # Route Distinguisher, identifie cette VRF
RP/0/RP0/CPU0:PE1(config-vrf)#address-family ipv4 unicast  # Spécification de l'adresse IPv4 pour la VRF
RP/0/RP0/CPU0:PE1(config-vrf-af)#import route-target 65000:128  # Importation des routes depuis un autre VRF
RP/0/RP0/CPU0:PE1(config-vrf-af)#export route-target 65000:128  # Exportation des routes vers un autre VRF
RP/0/RP0/CPU0:PE1(config-vrf-af)#commit  # Validation de la configuration
```

#### 1.2. Configuration de l'interface **GigabitEthernet0/0/0/3.30** et assignation de la VRF "ORANGE"

Ensuite, nous configurons une interface physique ou sous-interface pour le VPN, et nous y assignons la VRF "ORANGE".

```bash
# Configuration de l'interface GigabitEthernet0/0/0/3.30 sur PE1
RP/0/RP0/CPU0:PE1(config)#interface GigabitEthernet0/0/0/3.30
RP/0/RP0/CPU0:PE1(config-subif)#vrf ORANGE  # Attribution de la VRF à l'interface
RP/0/RP0/CPU0:PE1(config-subif)#ip address 30.0.1.1 255.255.255.0  # Attribution de l'adresse IP
RP/0/RP0/CPU0:PE1(config-subif)#commit  # Validation de la configuration
```

#### 1.3. Configuration du **BGP** pour la VRF "ORANGE"

Enfin, nous configurons le protocole BGP pour échanger des routes entre les VRFs.

```bash
# Configuration BGP pour la VRF ORANGE sur PE1
RP/0/RP0/CPU0:PE1(config)#router bgp 65000  # Configuration du BGP avec le numéro de système autonome
RP/0/RP0/CPU0:PE1(config-router)#address-family vpnv4 unicast  # Configuration du BGP VPNv4
RP/0/RP0/CPU0:PE1(config-router-af)#address-family ipv4 unicast  # Configuration du BGP IPv4
RP/0/RP0/CPU0:PE1(config-router-af)#redistribute connected  # Redistribution des routes connectées
RP/0/RP0/CPU0:PE1(config-router-af)#redistribute static  # Redistribution des routes statiques
RP/0/RP0/CPU0:PE1(config-router-af)#redistribute rip  # Redistribution des routes RIP
RP/0/RP0/CPU0:PE1(config-router-af)#commit  # Validation de la configuration
```

### Sur le routeur **PE2**:

#### 2.1. Création de l'instance VRF "ORANGE"

La même configuration VRF est appliquée sur **PE2**.

```bash
# Création de la VRF "ORANGE" sur PE2
RP/0/RP0/CPU0:PE2(config-vrf)#vrf ORANGE
RP/0/RP0/CPU0:PE2(config-vrf)#rd 172.30.0.6:3  # Route Distinguisher pour PE2
RP/0/RP0/CPU0:PE2(config-vrf)#address-family ipv4 unicast  # Spécification de l'adresse IPv4
RP/0/RP0/CPU0:PE2(config-vrf-af)#import route-target 65000:128  # Importation des routes depuis un autre VRF
RP/0/RP0/CPU0:PE2(config-vrf-af)#export route-target 65000:128  # Exportation des routes vers un autre VRF
RP/0/RP0/CPU0:PE2(config-vrf-af)#commit  # Validation de la configuration
```

#### 2.2. Configuration de l'interface **GigabitEthernet0/0/0/3.30** et assignation de la VRF "ORANGE"

Sur **PE2**, nous configurons également l'interface et y assignons la VRF "ORANGE".

```bash
# Configuration de l'interface GigabitEthernet0/0/0/3.30 sur PE2
RP/0/RP0/CPU0:PE2(config)#interface GigabitEthernet0/0/0/3.30
RP/0/RP0/CPU0:PE2(config-subif)#vrf ORANGE  # Attribution de la VRF à l'interface
RP/0/RP0/CPU0:PE2(config-subif)#ip address 30.0.2.1 255.255.255.0  # Attribution de l'adresse IP
RP/0/RP0/CPU0:PE2(config-subif)#commit  # Validation de la configuration
```

#### 2.3. Configuration du **BGP** pour la VRF "ORANGE"

Nous appliquons la même configuration BGP sur **PE2**.

```bash
# Configuration BGP pour la VRF ORANGE sur PE2
RP/0/RP0/CPU0:PE2(config)#router bgp 65000
RP/0/RP0/CPU0:PE2(config-router)#address-family vpnv4 unicast
RP/0/RP0/CPU0:PE2(config-router-af)#address-family ipv4 unicast
RP/0/RP0/CPU0:PE2(config-router-af)#redistribute connected
RP/0/RP0/CPU0:PE2(config-router-af)#redistribute static
RP/0/RP0/CPU0:PE2(config-router-af)#redistribute rip
RP/0/RP0/CPU0:PE2(config-router-af)#commit
```

## 2. Vérification de la configuration

### 2.1. Vérification de la table BGP sur PE1

Lancez la commande suivante sur **PE1** pour vérifier que les routes sont bien échangées via BGP :

```bash
show bgp vpnv4 unicast vrf ORANGE
```

La sortie doit ressembler à ceci :

```bash
* 30.0.1.0/24        0.0.0.0                  0         32768 ?
* i30.0.2.0/24        172.30.0.6               0    100      0 ?
```

### 2.2. Vérification de la table de routage sur PE1

Vérifiez que les routes sont correctement installées dans la table de routage de la VRF "ORANGE" avec la commande :

```bash
show ip route vrf ORANGE
```

La sortie attendue est :

```bash
C    30.0.1.0/24 is directly connected, GigabitEthernet0/0/0/3.30
B    30.0.2.0/24 [200/0] via 172.30.0.6 (nexthop in vrf default)
```

### 2.3. Vérification de la configuration VRF sur PE1 et PE2

Sur **PE1**, vérifiez la configuration de la VRF "ORANGE" :

```bash
show running-config vrf ORANGE
```

Sur **PE2**, vérifiez la configuration de la VRF "ORANGE" :

```bash
show running-config vrf ORANGE
```

Les configurations doivent être similaires.

### 2.4. Test de connectivité avec **ping**

Une fois la configuration effectuée, lancez un **ping** depuis le client pour vérifier la connectivité entre les équipements dans le VPN.

```bash
sudo docker exec -it Client ping 30.0.2.10
```

### 2.5. Lancer le générateur de trafic

Une fois le ping fonctionnel, vous pouvez lancer le générateur
