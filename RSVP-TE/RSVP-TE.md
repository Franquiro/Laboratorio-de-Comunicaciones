# RSVP-TE
## CISCO 7200
### Configuraciones
Primero congiguramos los las interfaces de cada router y su protocolo de routeo. en este caso trabajamos con OSPF.
#### Router A
```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#hostname CISCO_A
CISCO_A(config)#int loopback 0
CISCO_A(config-if)#ip address 1.1.1.1 255.255.255.255
CISCO_A(config-if)#exit
CISCO_A(config)#int e2/0
CISCO_A(config-if)#ip add 10.1.2.1 255.255.255.252
CISCO_A(config-if)#no shut
CISCO_A(config-if)#exit
CISCO_A(config)#int e2/3
CISCO_A(config-if)#ip add 10.1.3.1 255.255.255.252
CISCO_A(config-if)#no shut
CISCO_A(config-if)#exit
CISCO_A(config)#
CISCO_A(config)#router ospf 1
CISCO_A(config-router)#network 1.1.1.1 0.0.0.0 area 0
CISCO_A(config-router)#network 10.1.0.0 0.0.255.255 area 0
CISCO_A(config-router)#exit
CISCO_A(config)#
```

#### CISCO_B
```
R2#conf t
R2(config)#hostname CISCO_B
CISCO_B(config)#int loo
CISCO_B(config)#int loopback 0
CISCO_B(config-if)#ip address 2.2.2.2 255.255.255.255
CISCO_B(config-if)#exit
CISCO_B(config)#int e2/0                         
CISCO_B(config-if)#ip add 10.1.2.2 255.255.255.252
CISCO_B(config-if)#no shut
CISCO_B(config-if)#exit
CISCO_B(config)#int e2/1
CISCO_B(config-if)#ip add 10.2.3.1 255.255.255.252
CISCO_B(config-if)#no shut
CISCO_B(config-if)#exit
CISCO_B(config-if)#exit
CISCO_B(config)#router ospf 1
CISCO_B(config-router)#network 2.2.2.2 0.0.0.0 area 0
CISCO_B(config-router)#network 10.0.0.0 0.255.255.255 area 0
CISCO_B(config-router)#
CISCO_B(config-router)#end
CISCO_B#
```
#### CISCO_C
```
R3#conf t
R3(config)#hostname CISCO_C
CISCO_C(config)#int loopback 0
CISCO_C(config-if)#ip add 3.3.3.3 255.255.255.255
CISCO_C(config-if)#exit
CISCO_C(config)#int e2/1 
CISCO_C(config-if)#ip add 10.2.3.2 255.255.255.252
CISCO_C(config-if)#no shut
CISCO_C(config-if)#exit
CISCO_C(config)#int e2/3
CISCO_C(config-if)#ip add 10.1.3.2 255.255.255.252
CISCO_C(config-if)#no shut
CISCO_C(config-if)#exit
CISCO_C(config)#router ospf 1
CISCO_C(config-router)#network 3.3.3.3 0.0.0.0 area 0
CISCO_C(config-router)#network 10.0.0.0 0.255.255.255 area 0
CISCO_C(config-router)#end
CISCO_C#
```
Una vez configuradas las interfaces hay que habilitar la tunelizacion de **MPLS** y la calidad de servicio por **RSVP**

### **En cada Router**
```
CISCO_A#conf t
CISCO_A(config)#mpls
CISCO_A(config)#mpls traffic-eng tunnels
CISCO_A(config)#ip rsvp qos 
CISCO_A(config)#
CISCO_A(config)#router ospf 1
CISCO_A(config-router)#mpls traffic-eng area 0
CISCO_A(config-router)#mpls traffic-eng router-id loopback 0
CISCO_A(config-router)#end
CISCO_A#

```
Luego hay que diseñar el tunel que llevara la información desde el router A hasta el router C

para ello hay que definir una lista de los saltos que conforman la ruta.

```
CISCO_A(config)#ip explicit-path name camino_reservado
CISCO_A(cfg-ip-expl-path)#next-address 10.1.2.2
Explicit Path name camino_reservado:
    1: next-address 10.1.2.2
CISCO_A(cfg-ip-expl-path)#next-address 10.2.3.2
Explicit Path name camino_reservado:
    1: next-address 10.1.2.2
    2: next-address 10.2.3.2
CISCO_A(cfg-ip-expl-path)#exit
```
```
CISCO_A(config)#int tunnel 103
CISCO_A(config-if)#ip unnumbered loopback 0
CISCO_A(config-if)#tunnel mode mpls traffic-eng 
CISCO_A(config-if)#tunnel destination 3.3.3.3
CISCO_A(config-if)#tunnel mpls traffic-eng autoroute announce 
CISCO_A(config-if)#tunnel mpls traffic-eng priority 0 0
CISCO_A(config-if)#tunnel mpls traffic-eng bandwidth 5000
CISCO_A(config-if)#$tunnel mpls traffic-eng path-option 1 explicit name camino_reservado
CISCO_A(config-if)#

```
Luego Configurar el protocolo en cada interfaz
### CISCO_A
```
CISCO_A(config)#int e2/0
CISCO_A(config-if)#mpls ip
CISCO_A(config-if)#mpls traffic-eng tunnels 
CISCO_A(config-if)#ip rsvp bandwidth 5000
CISCO_A(config-if)#exit
CISCO_A(config)#
```
### CISCO_B
```
CISCO_B(config)#int e2/0
CISCO_B(config-if)#mpls traffic-eng tunnels 
CISCO_B(config-if)#ip rsvp bandwidth 5000
CISCO_B(config-if)#mpls ip
CISCO_B(config-if)#exit
CISCO_B(config)#

CISCO_B(config)#int e2/1
CISCO_B(config-if)#mpls traffic-eng tunnels 
CISCO_B(config-if)#ip rsvp bandwidth 5000
CISCO_B(config-if)#mpls ip
CISCO_B(config-if)#end
CISCO_B#
```
### CISCO_C
```
CISCO_C#conf t
CISCO_C(config)#int e2/1
CISCO_C(config-if)#mpls ip
CISCO_C(config-if)#mpls traffic-eng tunnels 
CISCO_C(config-if)#ip rsvp bandwidth 5000
CISCO_C(config-if)#end
CISCO_C#
```
