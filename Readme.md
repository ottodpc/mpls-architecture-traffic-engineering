PART 2:
Question 2.3
Donnez les commandes utilisées pour ajouter cette nouvelle instance, et décrivez de façon macroscopique le rôle de chacune d'entre elles.

Prouvez que votre configuration est valide en utilisant les commandes ping / traceroute.
CREATION VPN SUR PE1:
RP/0/RP0/CPU0:PE1(config-vrf)#vrf ORANGE
RP/0/RP0/CPU0:PE1(config-vrf)#rd 172.30.0.1:3
RP/0/RP0/CPU0:PE1(config-vrf)#address-family ipv4 unicast
RP/0/RP0/CPU0:PE1(config-vrf-af)#import route-target 65000:128
RP/0/RP0/CPU0:PE1(config-vrf-af)#export route-target 65000:128
RP/0/RP0/CPU0:PE1(config-vrf-af)#!
RP/0/RP0/CPU0:PE1(config-vrf-af)#commit
Sat Nov  9 14:30:43.141 UTC
RP/0/RP0/CPU0:PE1(config-vrf-af)#
