#Trusted VPN

 Hemos implementado una trusted VPN entre las distintas sedes de la empresa. Los pasos a seguir son los siguientes:

1) Creamos dos subinterfaces en el enlace entre frontera y cliente.

```bash
interface XX.Y
ip address {IP que queramos usar para tráfico normal} {Máscara}

interface XX.Z
ip address {IP de la conexión BGP para la VRF} {Máscara}
```

2) Configuramos MPLS en las interfaces de los routers del ISP:
```bash
interface XX
mpls ip

#Comandos de comprobación
show mpls interfaces
show mpls ldp neighbor
```

2) Creamos las VRFs en los routers frontera:

```bash
#Repetimos estos comandos en todos los routers frontera que participen en la VPN.
ip vrf {Nombre de la VRF}
rd XXXXX:XX
route-target both XXXXX:XX
```

3) Asignamos las VRFs a las interfaces del router:

```bash
interface XX.Z
ip vrf forwarding {Nombre de la VRF}

#Debemos asignar de nuevo la ip a la interfaz
ip add {IP} {Máscara}
```

4) Configuramos BGP entre los routers frontera.

```bash
router bgp {identificador del AS}

neighbor {IP de la loopback del vecino} remote-as {identificador del AS}
neighbor {IP de la loopback del vecino} update-source loopback 0

address-family vpnv4
neighbor {IP de la loopback del vecino} activate

#Repetimos en el otro vecino
```

5) Configuramos BGP entre los routers frontera y el router del cliente.

```bash
#En el router frontera
router bgp {identificador del AS}

address-family ipv4 vrf Blendugs
neighbor {IP de la subinterfaz VRF del vecino} remote-as {ID del AS del vecino}
neighbor {IP de la subinterfaz VRF del vecino} activate

#En el router del cliente
router bgp {Identificador del AS}

#Se repite tantas veces como redes se quieran compartir
network {Prefijo de la VPN} mask {Máscara del prefjo}

neighbor {IP de la subinterfaz VRF del router frontera} remote-as {ID del AS del ISP}
```

Opcional: Filtrado de las rutas que compartes con cada vecino

```bash

#En el router del cliente

#Rutas que NO queremos enviar, una línea por cada
access-list {1-99} deny {Prefijo que no queremos enviar} {Wildcard de la máscara}
access-list {1-99} deny {Prefijo que no queremos enviar} {Wildcard de la máscara}
access-list {1-99} permit any

router bgp {ID del AS}

#En caso de no haberlo hecho ya añadir todas las rutas que queramos transmitir
network {Prefijo a transmitir} mask {Máscara del prefijo}

#Le asignamos una lista de distribución que filtre las direcciones que se le envían a ese vecino
neighbor {IP del vecino} distribute-list {1-99} out
```

