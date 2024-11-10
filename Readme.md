# Introduction :
xxxxx

# Partie 1: Vanilla MPLS

Question 1.1

Liste des Labels pour les Adresses de Loopback :

La capture ci-dessous montre la sortie de la commande show mpls forwarding sur PE1. Cette commande nous permet de voir les labels locaux et de sortie que PE1 utilise pour atteindre les adresses de Loopback des autres routeurs.

![image](https://github.com/user-attachments/assets/73935219-2853-464d-9dd5-78e74fad2730)



Différence entre "Label Local" et "Label Output" :

●	Label Local : C’est le label que PE1 utilise pour reconnaître un flux MPLS entrant. En d'autres termes, c’est le label que PE1 attribue à un flux de données MPLS spécifique pour une destination donnée. 
●	Label Output : C’est le label que PE1 applique au paquet en sortie pour qu'il soit compris par le prochain routeur dans le réseau MPLS. Ce label sera utilisé par le prochain routeur pour transférer le paquet jusqu'à la destination finale. 

Pop Label : Lorsque le label de sortie est "Pop", cela signifie que PE1 retire le label avant de transmettre le paquet. Cela se produit généralement lorsque PE1 est le dernier routeur dans le chemin MPLS vers cette destination.


Pourquoi un Label pour P2 alors qu’il est directement connecté ?

Un label est utilisé pour atteindre P2, bien que ce routeur soit directement connecté, car MPLS applique un routage de bout en bout basé sur les labels pour toutes les destinations. Cela permet une gestion uniforme et simplifiée des flux MPLS, même pour les routes directement connectées. Ainsi, PE1 peut traiter tous les paquets de la même manière, sans exception pour les destinations locales, facilitant ainsi la gestion et la scalabilité du réseau MPLS.


 ![image](https://github.com/user-attachments/assets/c9111b00-a0b9-452f-9c1c-71a0c31c5cc4)


La capture ci-dessus montre les voisins LDP de PE1 obtenus avec la commande show mpls ldp neighbor. Nous voyons que PE1 a des voisins LDP, y compris des routeurs directement connectés comme 172.30.0.2 (P2). L’utilisation d’un label pour P2 permet de maintenir une uniformité dans le routage MPLS. Cette uniformité simplifie la gestion et permet une plus grande extensibilité, car le réseau traite tous les paquets de la même manière, sans exception pour les destinations locales.


Question 1.2
l’Adresse IP de Loopback de PE1 :


 ![image](https://github.com/user-attachments/assets/ee10988a-e6ea-4cd2-b89b-c53ca8d8efb5)

l’Adresse IP de Loopback de PE2 :


 ![image](https://github.com/user-attachments/assets/376740cc-4480-43ea-a928-4b54f4231fd0)

Ces adresses sont utilisées pour les communications de bout en bout dans le réseau MPLS et pour le tracé des chemins via traceroute mpls.

d
Traceroute de PE1 vers PE2 


La commande traceroute mpls ipv4 exécutée sur PE1 pour atteindre l'adresse de loopback de PE2 (172.30.0.6/32) a révélé les sauts et labels suivants :

Saut	Routeur 	IP Next-Hop	Label Entrant	Label Sortant
0	10.74.38.5 (PE1)	10.74.38.6 (P2)	-	24003
1	10.74.38.6 (P2)	10.74.38.18 (P4)	24003	24000
2	10.74.38.18 (P4)	10.74.38.34 (PE2)	24000	implicit-null (Pop)

Implicit-Null : Le dernier routeur retire le label (implicit-null) avant de livrer le paquet à la destination finale (loopback de PE2). Ce processus est appelé “PHP” (Penultimate Hop Popping) dans MPLS, où le dernier label est retiré avant l'arrivée au routeur final.

Après avoir exécuté la commande traceroute mpls ipv4 sur chacun des routeurs on obtient la route suivant :
PE1 ➡️ P2 ➡️ P4 ➡️ PE2
 


Traceroute de PE2 vers PE1 


La commande traceroute mpls ipv4 exécutée sur PE2 pour atteindre l'adresse de loopback de PE1 (172.30.0.1/32) a révélé les sauts et labels suivants :

Saut	Routeur 	IP Next-Hop	Label Entrant	Label Sortant
0	10.74.38.34 (PE2)	10.74.38.33 (P4)	-	24003
1	10.74.38.33 (P4)	10.74.38.17 (P2)	24003	24000
2	10.74.38.17 (P2)	10.74.38.5 (PE1)	24000	implicit-null (Pop)

Implicit-Null : Le dernier routeur retire le label (implicit-null) avant de livrer le paquet à la destination finale (loopback de PE2). Ce processus est appelé “PHP” (Penultimate Hop Popping) dans MPLS, où le dernier label est retiré avant l'arrivée au routeur final.

Après avoir exécuté la commande traceroute mpls ipv4 sur chacun des routeurs on obtient la route suivant :
PE2 ➡️ P2 ➡️ P4 ➡️ PE1
 


Les chemins entre PE1 et PE2 (dans les deux directions) montrent que chaque routeur applique un label spécifique pour acheminer les paquets dans le backbone MPLS. Les trajets utilisent des LSPs (Label Switched Paths) qui assurent la connectivité de bout-en-bout en appliquant un label de sortie spécifique à chaque saut. Le dernier saut retire le label via le processus "Implicit Null” pour optimiser l'acheminement. Ces résultats confirment le fonctionnement correct de MPLS et LDP pour la communication entre les routeurs PE1 et PE2.

Question 1.3


Script pour Wireshark :
 
Commande pour capturer le flux transitant au niveau de l'interface Gi0-0-0-1 du router P4.

Relancer la Session LDP :
 
Cette action forcera l'échange des messages d'établissement de session LDP entre P2 et P4, qu’on pourrait alors observer dans Wireshark.


Observation du Trafic :

Lors de la remise du lien entre P2 et P4, deux procédures d’initialisation d’échange de messages LDP ont lieu, entre P2 ➡️ P4 et P4 ➡️ P2.

Cette capture a été réalisée pour étudier les messages LDP échangés entre les routeurs P2 et P4 lors de l'établissement et du maintien d'une session LDP. Les messages analysés incluent les messages de découverte, d'initialisation, de maintien de session, et de mappage de labels. Chaque type de message joue un rôle spécifique dans l'établissement et la gestion des Label Switched Paths (LSPs) entre les routeurs.

 

●	Dans la capture, les messages "Hello" sont utilisés pour découvrir les voisins LDP. Chaque routeur envoie un Hello Message pour annoncer sa présence. Par exemple, on peut voir l'adresse source 172.30.0.3 (P2) envoyant un Hello Message pour signaler sa disponibilité à 172.30.0.5 (P4).
●	Les messages "Initialization" dans la capture permettent d'établir la session LDP entre P2 et P4. Ce message contient les informations nécessaires à la configuration de la session, telles que la version du protocole LDP et les options de session.
●	Le Keepalive Message permet de maintenir la session LDP entre P2 et P4 active. Dans la capture, ces messages sont envoyés périodiquement pour garantir que la connexion reste stable.
●	Le Label Mapping Message permet d'associer un label à chaque préfixe IP, facilitant ainsi l'acheminement MPLS. Par exemple, le message pour le préfixe 172.30.0.3 associe un label qui sera utilisé pour les paquets destinés à cette adresse.



Question 1.4


Signification du Label 3 (Implicit Null) : 

Dans la capture précédente, certains préfixes sont associés au label 3, appelé Implicit Null. Ce label a une signification particulière dans MPLS : il est utilisé pour indiquer au dernier routeur intermédiaire (Penultimate Hop) de retirer le label avant d'envoyer le paquet au routeur de destination. Cette opération, connue sous le nom de Penultimate Hop Popping (PHP), optimise la commutation en supprimant la nécessité pour le routeur de destination de traiter le label MPLS. 

En assignant le label 3, le réseau allège la charge du routeur de destination en lui envoyant le paquet directement avec l’en-tête IP, sans en-tête MPLS. Cela améliore l'efficacité du traitement des paquets.



Plage de Labels Réservés et leur Utilité : 

Dans MPLS, les labels réservés vont de 0 à 15 et sont utilisés pour des fonctions spécifiques. 

Label 0 : Explicit Null pour IPv4, utilisé pour indiquer que le label ne doit pas être retiré afin de préserver l'en-tête IP. 
Label 1 : Explicit Null pour IPv6. 
Label 3 : Implicit Null, utilisé pour le Penultimate Hop Popping (PHP), demandant au dernier routeur intermédiaire de retirer le label avant de transmettre le paquet au routeur de destination.
Labels 4 à 15 : Réservés pour des utilisations futures ou spécifiques selon les implémentations.


Utilité de la Plage Réservée :

Cette plage de labels réservés est cruciale pour assurer une gestion standardisée et efficace des paquets MPLS dans le réseau. Les labels réservés permettent de réduire la charge sur les routeurs de destination pour certains préfixes et de gérer les paquets de manière optimisée.

Les tables de Commutation Partie 1 :


RP/0/RP0/CPU0:PE1#show mpls forwarding
Sun Nov  3 17:32:09.010 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
24000  24004       172.30.0.2/32      Gi0/0/0/2    10.74.38.6      33364       
24001  24002       172.30.0.4/32      Gi0/0/0/2    10.74.38.6      0           
24002  24001       172.30.0.5/32      Gi0/0/0/2    10.74.38.6      0           
24003  24003       172.30.0.6/32      Gi0/0/0/2    10.74.38.6      18078       
24004  Pop         10.74.38.8/30      Gi0/0/0/2    10.74.38.6      0           
24005  24009       10.74.38.12/30     Gi0/0/0/2    10.74.38.6      0           
24006  24005       10.74.38.24/30     Gi0/0/0/2    10.74.38.6      0           
24007  24008       10.74.38.20/30     Gi0/0/0/2    10.74.38.6      0           
24008  Pop         10.74.38.16/30     Gi0/0/0/2    10.74.38.6      0           
24009  24006       10.74.38.32/30     Gi0/0/0/2    10.74.38.6      80          
24010  24007       10.74.38.28/30     Gi0/0/0/2    10.74.38.6      0           
24011  Pop         172.30.0.3/32      Gi0/0/0/2    10.74.38.6      33390       
24012  Aggregate   GREEN: Per-VRF Aggr[V]   \
                                      GREEN                        0           
24013  Aggregate   RED: Per-VRF Aggr[V]   \
                                      RED                          0     


RP/0/RP0/CPU0:P1#show mpls forwarding 
Sun Nov  3 17:34:43.397 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
20002  Aggregate   SR Pfx (idx 2)     default                      0           
20003  Pop         SR Pfx (idx 3)     Gi0/0/0/3    10.74.38.10     34520       
20004  16004       SR Pfx (idx 4)     Gi0/0/0/4    10.74.38.22     6722        
20005  Pop         SR Pfx (idx 5)     Gi0/0/0/4    10.74.38.22     6616        
20102  Aggregate   SR Pfx (idx 102)   default                      0           
24000  24000       172.30.0.1/32      Gi0/0/0/3    10.74.38.10     33916       
24001  Pop         10.74.38.4/30      Gi0/0/0/3    10.74.38.10     0           
24002  Pop         172.30.0.5/32      Gi0/0/0/4    10.74.38.22     0           
24003  24001       172.30.0.4/32      Gi0/0/0/4    10.74.38.22     0           
24004  24004       10.74.38.28/30     Gi0/0/0/4    10.74.38.22     0           
24005  Pop         10.74.38.24/30     Gi0/0/0/4    10.74.38.22     0           
24006  Pop         10.74.38.16/30     Gi0/0/0/3    10.74.38.10     0           
       Pop         10.74.38.16/30     Gi0/0/0/4    10.74.38.22     0           
24007  24000       172.30.0.6/32      Gi0/0/0/4    10.74.38.22     0           
24008  Pop         172.30.0.3/32      Gi0/0/0/3    10.74.38.10     0           
24009  Pop         10.74.38.32/30     Gi0/0/0/4    10.74.38.22     0           
24010  Pop         SR Adj (idx 1)     Gi0/0/0/2    10.74.38.14     0           
24011  Pop         SR Adj (idx 3)     Gi0/0/0/2    10.74.38.14     0           
24012  Pop         SR Adj (idx 1)     Gi0/0/0/4    10.74.38.22     0           
24013  Pop         SR Adj (idx 3)     Gi0/0/0/4    10.74.38.22     0           
24014  Pop         SR Adj (idx 1)     Gi0/0/0/3    10.74.38.10     0           
24015  Pop         SR Adj (idx 3)     Gi0/0/0/3    10.74.38.10     0           
24016  Pop         SR Adj (idx 1)     Gi0/0/0/1    10.74.38.1      0           
24017  Pop         SR Adj (idx 3)     Gi0/0/0/1    10.74.38.1      0     


RP/0/RP0/CPU0:P2#show mpls forwarding 
Sun Nov  3 17:43:13.034 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
20002  Pop         SR Pfx (idx 2)     Gi0/0/0/3    10.74.38.9      36816       
20003  Aggregate   SR Pfx (idx 3)     default                      0           
20004  16004       SR Pfx (idx 4)     Gi0/0/0/1    10.74.38.18     0           
20005  Pop         SR Pfx (idx 5)     Gi0/0/0/1    10.74.38.18     7756        
20103  Aggregate   SR Pfx (idx 103)   default                      0           
24000  Pop         172.30.0.1/32      Gi0/0/0/2    10.74.38.5      95774       
24001  Pop         172.30.0.5/32      Gi0/0/0/1    10.74.38.18     0           
24002  24001       172.30.0.4/32      Gi0/0/0/1    10.74.38.18     0           
24003  24000       172.30.0.6/32      Gi0/0/0/1    10.74.38.18     4076        
24004  Pop         172.30.0.2/32      Gi0/0/0/3    10.74.38.9      31966       
24005  Pop         10.74.38.24/30     Gi0/0/0/1    10.74.38.18     0           
24006  Pop         10.74.38.32/30     Gi0/0/0/1    10.74.38.18     0           
24007  24004       10.74.38.28/30     Gi0/0/0/1    10.74.38.18     0           
24008  Pop         10.74.38.20/30     Gi0/0/0/1    10.74.38.18     0           
       Pop         10.74.38.20/30     Gi0/0/0/3    10.74.38.9      0           
24009  Pop         10.74.38.12/30     Gi0/0/0/3    10.74.38.9      0           
24010  Pop         10.74.38.0/30      Gi0/0/0/2    10.74.38.5      0           
       Pop         10.74.38.0/30      Gi0/0/0/3    10.74.38.9      0           
24011  Pop         SR Adj (idx 1)     Gi0/0/0/1    10.74.38.18     0           
24012  Pop         SR Adj (idx 3)     Gi0/0/0/1    10.74.38.18     0           
24013  Pop         SR Adj (idx 1)     Gi0/0/0/3    10.74.38.9      0           
24014  Pop         SR Adj (idx 3)     Gi0/0/0/3    10.74.38.9      0           
24015  Pop         SR Adj (idx 1)     Gi0/0/0/2    10.74.38.5      0           
24016  Pop         SR Adj (idx 3)     Gi0/0/0/2    10.74.38.5      0   



RP/0/RP0/CPU0:P3#show mpls forwarding 
Sun Nov  3 17:48:01.581 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
20002  16002       SR Pfx (idx 2)     Gi0/0/0/3    10.74.38.26     8286        
20003  16003       SR Pfx (idx 3)     Gi0/0/0/3    10.74.38.26     0           
20004  Aggregate   SR Pfx (idx 4)     default                      0           
20005  Pop         SR Pfx (idx 5)     Gi0/0/0/3    10.74.38.26     9818        
20104  Aggregate   SR Pfx (idx 104)   default                      0           
24000  24002       172.30.0.2/32      Gi0/0/0/3    10.74.38.26     0           
24001  Pop         172.30.0.5/32      Gi0/0/0/3    10.74.38.26     0           
24002  24000       172.30.0.6/32      Gi0/0/0/3    10.74.38.26     8224        
24003  24003       172.30.0.1/32      Gi0/0/0/3    10.74.38.26     0           
24004  24006       10.74.38.8/30      Gi0/0/0/3    10.74.38.26     0           
24005  Pop         10.74.38.20/30     Gi0/0/0/3    10.74.38.26     0           
24006  Pop         10.74.38.16/30     Gi0/0/0/3    10.74.38.26     0           
24007  Pop         10.74.38.32/30     Gi0/0/0/3    10.74.38.26     0           
24008  24007       10.74.38.4/30      Gi0/0/0/3    10.74.38.26     0           
24009  24008       10.74.38.0/30      Gi0/0/0/3    10.74.38.26     0           
24010  24009       172.30.0.3/32      Gi0/0/0/3    10.74.38.26     0           
24011  Pop         SR Adj (idx 1)     Gi0/0/0/1    10.74.38.30     0           
24012  Pop         SR Adj (idx 3)     Gi0/0/0/1    10.74.38.30     0           
24013  Pop         SR Adj (idx 1)     Gi0/0/0/3    10.74.38.26     0           
24014  Pop         SR Adj (idx 3)     Gi0/0/0/3    10.74.38.26     0           
24015  Pop         SR Adj (idx 1)     Gi0/0/0/2    10.74.38.13     0           
24016  Pop         SR Adj (idx 3)     Gi0/0/0/2    10.74.38.13     0  


RP/0/RP0/CPU0:P4#show mpls forwarding 
Sun Nov  3 17:47:30.078 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
16002  Pop         SR Pfx (idx 2)     Gi0/0/0/4    10.74.38.21     17936       
16003  Pop         SR Pfx (idx 3)     Gi0/0/0/1    10.74.38.17     12958       
16004  Pop         SR Pfx (idx 4)     Gi0/0/0/3    10.74.38.25     18046       
16005  Aggregate   SR Pfx (idx 5)     default                      0           
16105  Aggregate   SR Pfx (idx 105)   default                      0           
24000  Pop         172.30.0.6/32      Gi0/0/0/2    10.74.38.34     19946       
24001  Pop         172.30.0.4/32      Gi0/0/0/3    10.74.38.25     7546        
24002  Pop         172.30.0.2/32      Gi0/0/0/4    10.74.38.21     0           
24003  24000       172.30.0.1/32      Gi0/0/0/1    10.74.38.17     7276        
24004  Pop         10.74.38.28/30     Gi0/0/0/2    10.74.38.34     0           
       Pop         10.74.38.28/30     Gi0/0/0/3    10.74.38.25     0           
24005  Pop         10.74.38.12/30     Gi0/0/0/3    10.74.38.25     0           
       Pop         10.74.38.12/30     Gi0/0/0/4    10.74.38.21     0           
24006  Pop         10.74.38.8/30      Gi0/0/0/1    10.74.38.17     0           
       Pop         10.74.38.8/30      Gi0/0/0/4    10.74.38.21     0           
24007  Pop         10.74.38.4/30      Gi0/0/0/1    10.74.38.17     0           
24008  Pop         10.74.38.0/30      Gi0/0/0/4    10.74.38.21     0           
24009  Pop         172.30.0.3/32      Gi0/0/0/1    10.74.38.17     0           
24010  Pop         SR Adj (idx 1)     Gi0/0/0/2    10.74.38.34     0           
24011  Pop         SR Adj (idx 3)     Gi0/0/0/2    10.74.38.34     0           
24012  Pop         SR Adj (idx 1)     Gi0/0/0/3    10.74.38.25     0           
24013  Pop         SR Adj (idx 3)     Gi0/0/0/3    10.74.38.25     0           
24014  Pop         SR Adj (idx 1)     Gi0/0/0/4    10.74.38.21     0           
24015  Pop         SR Adj (idx 3)     Gi0/0/0/4    10.74.38.21     0           
24016  Pop         SR Adj (idx 1)     Gi0/0/0/1    10.74.38.17     0           
24017  Pop         SR Adj (idx 3)     Gi0/0/0/1    10.74.38.17     0    



RP/0/RP0/CPU0:PE2#show mpls forwarding 
Sun Nov  3 18:01:41.202 UTC
Local  Outgoing    Prefix             Outgoing     Next Hop        Bytes       
Label  Label       or ID              Interface                    Switched    
------ ----------- ------------------ ------------ --------------- ------------
24000  24001       172.30.0.4/32      Gi0/0/0/2    10.74.38.33     9902        
24001  Pop         172.30.0.5/32      Gi0/0/0/2    10.74.38.33     10026       
24002  24002       172.30.0.2/32      Gi0/0/0/2    10.74.38.33     0           
24003  24003       172.30.0.1/32      Gi0/0/0/2    10.74.38.33     8774        
24004  Pop         10.74.38.24/30     Gi0/0/0/2    10.74.38.33     0           
24005  Pop         10.74.38.16/30     Gi0/0/0/2    10.74.38.33     0           
24006  Pop         10.74.38.20/30     Gi0/0/0/2    10.74.38.33     0           
24007  24005       10.74.38.12/30     Gi0/0/0/2    10.74.38.33     0           
24008  24006       10.74.38.8/30      Gi0/0/0/2    10.74.38.33     0           
24009  24007       10.74.38.4/30      Gi0/0/0/2    10.74.38.33     0           
24010  24008       10.74.38.0/30      Gi0/0/0/2    10.74.38.33     0           
24011  24009       172.30.0.3/32      Gi0/0/0/2    10.74.38.33     0           
24012  Aggregate   GREEN: Per-VRF Aggr[V]   \
                                      GREEN                        0           
24013  Aggregate   RED: Per-VRF Aggr[V]   \
                                      RED                          0 






































































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

Le label situé juste au-dessus de la couche IPv4 dans notre VPN est attribué par le nœud **PE1** via le protocole **LDP (Label Distribution Protocol)**, qui distribue des labels MPLS aux préfixes IP pour assurer un routage efficace des paquets dans le réseau VPN.

En exécutant la commande `show mpls ldp bindings`, nous pouvons voir les labels attribués localement et ceux reçus de pairs distants. Cette commande fournit le résultat suivant :

![image](https://github.com/user-attachments/assets/e7f67b9e-2f03-49c7-a714-1478a20cb6f2)

Dans les résultats, nous observons l'association entre les labels et les préfixes IPv4. Par exemple, pour le préfixe **172.30.0.2/32**, le label **24001** est attribué localement par **PE1**, tandis que le label **24010** est reçu d’un pair, indiquant une configuration de transport.

Voici des exemples détaillés du résultat de la commande, montrant le préfixe, les labels locaux et distants, ainsi que les pairs impliqués :

- **172.30.0.2/32** : Label local **24001** ; label distant **24010** attribué par le pair **172.30.0.3**.
- **10.74.38.4/30** : Label local **ImpNull** (indiquant un transport direct sans label) et label distant **24011** pour le pair **172.30.0.2**.

Ces informations montrent que **PE1** attribue des labels pour chaque préfixe en fonction de la destination, permettant au réseau MPLS de router le trafic efficacement avec des labels spécifiques.
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




# PARTIE 3: migration de MPLS LDP vers SR-MPLS
# Étape 1 : Activation de SR sur les routeurs d'extrémité (PE)

Objectif : Activer Segment Routing (SR) sur les routeurs PE (Provider Edge), afin que les routes et labels SR soient propagés par l’IGP (ISIS) sans perturber le trafic client, car LDP reste actif et prioritaire.

# Configuration sur les routeurs PE (PE1 et PE2)

1. Activez Segment Routing avec une plage globale de labels (SRGB) et assignez un index SID pour l’interface Loopback0 :
   ```bash
   router isis IGP
     segment-routing global-block 20000 23999  # Définition de la plage de labels
     address-family ipv4 unicast
       segment-routing mpls  # Activation de SR pour l’IGP ISIS
     interface Loopback0
       address-family ipv4 unicast
         prefix-sid index 1  # Assigne le SID index 1 pour PE1 et un autre pour PE2
   ```
   - Remarque : L’index `1` est un exemple et doit être unique par routeur dans le réseau pour identifier chaque PE de manière univoque.

# Vérification

2. Afficher la table des labels : Vérifiez que les labels SR sont bien générés en exécutant la commande suivante sur les routeurs PE :
   ```bash
   show mpls forwarding-table
   ```

3. Traceroute SR-MPLS : Effectuez un test de connectivité en forçant l’utilisation des labels SR avec `traceroute sr-mpls` :
   ```bash
   traceroute sr-mpls PE2_Loopback0
   ```
   - But : Cette commande permet de valider que le chemin de bout en bout entre PE1 et PE2 utilise SR, bien que LDP reste prioritaire pour le trafic par défaut.

---

# Étape 2 : Priorisation de SR vis-à-vis de LDP

Objectif : Modifier la configuration ISIS pour que SR-MPLS devienne prioritaire sur LDP, tout en assurant la connectivité via LDP pour les routes non encore migrées.

# Configuration sur les routeurs PE (PE1 et PE2)

1. Prioriser SR dans l’IGP ISIS :
   ```bash
   router isis IGP
     address-family ipv4 unicast
       segment-routing mpls sr-prefer
   ```

#Vérification

2. **Traceroute SR-MPLS avec priorisation** : Effectuez un `traceroute sr-mpls` entre PE1 et PE2 pour observer l’utilisation des labels SR à chaque saut.
   ```bash
   traceroute sr-mpls PE2_Loopback0
   ```
   - **Observation** : Si le trafic utilise les labels SR, vous devriez constater un changement de labels à chaque saut, ce qui indique l’activation de SR-MPLS comme mode de commutation prioritaire.

---

#Étape 3 : Désactivation de LDP
Objectif : Désactiver LDP sur tous les routeurs de l'infrastructure (PE et P) pour garantir que SR-MPLS est le seul protocole MPLS utilisé pour le routage entre les PE.

# Configuration sur tous les routeurs (PE1, PE2, P1, P2, P3, P4)

1. **Supprimer LDP de la configuration des interfaces et du processus IGP** :
   ```bash
   router isis IGP
     address-family ipv4 unicast
       no mpls ldp auto-config
     exit
   interface GigabitEthernetX/Y
     address-family ipv4 unicast
       no mpls ldp sync
   exit
   no mpls ldp
   ```

# Vérification

2. **Capture de paquets entre P2 et P4** : Effectuez une capture de paquets pour analyser les informations SR en cours de propagation, en utilisant un outil de capture comme `tcpdump`. Recherchez :
   - SRGB de PE2 : Plage de labels utilisée par PE2 pour SR.
   - Node SID de PE2 : SID unique associé à PE2.
   - Label pour le segment P3 vers PE2 : Vérifiez le label de routage entre P3 et PE2.
   - Algorithmes supportés par PE2 : Identifiez les algorithmes SR annoncés dans l’IGP.

3. **Afficher la base de données ISIS pour les détails SR** :
   ```bash
   show isis database verbose
   ```
   - But : La base de données ISIS contiendra les TLVs (Type-Length-Values) relatifs à SR, confirmant la propagation correcte des informations de routage et des labels SR.

4. **Redémarrer ISIS pour rafraîchir les sessions et les labels** :
   ```bash
   clear isis process
   ```
   - **Conseil** : Si nécessaire, redémarrez l’IGP pour rafraîchir les sessions et assurer que les labels sont bien assignés et propagés.

---

Conclusion

Cette méthode progressive permet une transition en douceur de LDP à SR-MPLS, garantissant une continuité de service pour les clients sans interruption. Si certains labels de la plage SRGB sont déjà utilisés, un redémarrage des routeurs peut être requis pour éviter tout conflit d'assignation de labels.




# Rapport Partie 3: Ingénierie de Trafic avec Segment Routing Traffic Engineering (SR-TE)


![image](https://github.com/user-attachments/assets/d6a54fdc-9804-43b6-9ba8-6413b3453864)


# Objectifs
Cette partie est la mise en place de l'ingénierie de trafic en utilisant Segment Routing Traffic Engineering (SR-TE). L'objectif est de maîtriser les concepts d’optimisation de chemin, de contrôle du trafic et d’amélioration de l'efficacité des ressources réseau à l'aide de tunnels SR-TE.

---

# Question 1 : Configuration des Interfaces et Routage de Base

# Étapes et Commandes

1. Configuration des adresses IP sur les interfaces :
   - Connectez-vous sur chaque routeur et configurez les adresses IP pour établir la connectivité de base entre les équipements.
   ```bash
   configure terminal
   interface GigabitEthernet0/0
   ip address 192.168.1.1 255.255.255.0
   no shutdown
   ```

2. Vérification de la connectivité :
   - Vérifiez la connectivité entre les équipements pour vous assurer que chaque routeur est joignable via les adresses IP configurées.
   ```bash
   ping 192.168.1.2
   ```

3. Configuration du routage statique (si nécessaire) :
   - Ajoutez des routes statiques pour diriger le trafic entre les sous-réseaux si un routage dynamique n'est pas encore configuré.
   ```bash
   ip route 192.168.2.0 255.255.255.0 192.168.1.2
   ```

# Question 2 : Activation du Segment Routing (SR)
# Étapes et Commandes

1. Activer le Segment Routing sur chaque routeur :
   - Configurez le Segment Routing pour chaque routeur en utilisant le protocole IGP (ex : OSPF ou IS-IS).
   ```bash
   configure terminal
   router ospf 1
   segment-routing mpls
   ```

2. Vérification de la configuration Segment Routing :
   - Assurez-vous que la configuration Segment Routing est active en affichant les routes SR dans la table de routage.
   ```bash
   show ip route
   ```

# Question 3 : Création et Configuration des Tunnels SR-TE
# Étapes et Commandes

1. Configuration de l'interface de tunnel :
   - Configurez des tunnels SR-TE pour guider le trafic à travers des chemins spécifiques dans le réseau.
   ```bash
   configure terminal
   interface Tunnel1
   ip unnumbered Loopback0
   tunnel mode mpls traffic-eng
   tunnel mpls traffic-eng autoroute announce
   tunnel destination 192.168.2.1
   ```

2. Définir les segments de chemin et les contraintes :
   - Définissez les segments et contraintes pour chaque tunnel, comme les ressources de bande passante.
   ```bash
   mpls traffic-eng path-option 1 explicit name SR-Path
   ```

3. Activation des chemins explicites et vérification :
   - Assurez-vous que les tunnels sont activés et vérifiez leur état et leur chemin emprunté.
   ```bash
   show mpls traffic-eng tunnels
   ```

# Question 4 : Contrôle et Redirection du Trafic via SR-TE
# Étapes et Commandes

1. Configurer des règles de routage pour utiliser les tunnels SR-TE :
   - Utilisez des route-maps pour rediriger certains flux de trafic dans des tunnels spécifiques.
   ```bash
   route-map SR-TE permit 10
   match ip address 101
   set ip next-hop Tunnel1
   ```

2. Appliquer la route-map sur les interfaces :
   - Appliquez la route-map configurée sur les interfaces appropriées pour rediriger le trafic.
   ```bash
   interface GigabitEthernet0/0
   ip policy route-map SR-TE
   ```

3. Vérification de la redirection du trafic :
   - Surveillez l'utilisation des tunnels et assurez-vous que le trafic emprunte bien les chemins spécifiés.
   ```bash
   show ip traffic
   ```

---

# Question 5 : Surveillance et Dépannage des Tunnels SR-TE
# Étapes et Commandes

1. Surveillance des tunnels SR-TE :
   - Utilisez des commandes de surveillance pour suivre l’état des tunnels et le trafic qu’ils transportent.
   ```bash
   show mpls traffic-eng tunnels statistics
   ```

2. Analyse des statistiques de trafic :
   - Utilisez des commandes pour analyser les performances des tunnels et ajuster les chemins en cas de surcharge.
   ```bash
   show mpls traffic-eng tunnels brief
   ```

3. **Dépannage des problèmes de connectivité** :
   - Effectuez des diagnostics si le trafic ne suit pas les chemins configurés ou si des tunnels sont inactifs.
   ```bash
   traceroute 192.168.2.1
   debug mpls traffic-eng
   ```

#Conclusion : 

Cette derrière partie de l'ingénierie de trafic avec SR-TE a permis de mettre en œuvre des tunnels optimisés et de contrôler le routage des paquets pour un réseau plus efficace. Grâce aux tunnels SR-TE, le trafic a pu être redirigé sur des chemins spécifiques, améliorant ainsi l’utilisation des ressources et la résilience du réseau.
