# Lab 4 — Protocolos de Enrutamiento Dinámico IPv4 e IPv6
**Universidad de los Andes · Infraestructura de Comunicaciones**
**Sección 4 · Grupo 10**

---

## Guías del laboratorio

| Protocolo | Topología | Enlace |
|-----------|-----------|--------|
| RIPv1 | Topología 1 — IPv4 | [Lab4_RIPv1](https://github.com/jzarruke/RedesLab4/blob/main/Lab4_RIPv1_Seccion4_Grupo10.md) |
| OSPFv2 | Topología 1 — IPv4 | [Lab4_OSPFv2](https://github.com/jzarruke/RedesLab4/blob/main/Lab4_OSPFv2_Seccion4_Grupo10.md) |
| BGP | Topología 2 — IPv4 | [Lab4_BGP](https://github.com/jzarruke/RedesLab4/blob/main/Lab4_BGP_Seccion4_Grupo10.md) |
| RIPng | Topología 3 — IPv6 | [Lab4_RIPng](https://github.com/jzarruke/RedesLab4/blob/main/Lab4_RIPng_Seccion4_Grupo10.md) |
| OSPFv3 | Topología 3 — IPv6 | [Lab4_OSPFv3](https://github.com/jzarruke/RedesLab4/blob/main/Lab4_OSPFv3_Seccion4_Grupo10.md) |

---

## Cheat sheet de comandos

### Configuración base (todos los routers, todos los protocolos)
```
enable
configure terminal
hostname RouterX
interface GigabitEthernet0/0
 ip address [IP] [MASK]
 no shutdown
write memory
```

---

### RIPv1
```
router rip
 version 1
 network [red-clase]        ! sin máscara, classful (ej: 192.168.10.0)
```
```
show ip route               ! rutas marcadas con R
show running-config | section router rip
debug ip rip
```

---

### OSPFv2
```
router ospf 1
 network [ip] [wildcard] area 0   ! ej: network 192.168.10.0 0.0.0.255 area 0
```
```
show ip ospf neighbor             ! estado debe ser FULL
show ip route                     ! rutas marcadas con O
show ip ospf interface
show ip ospf database
```

---

### BGP
```
router bgp [ASN]
 neighbor [IP-vecino] remote-as [ASN-vecino]
 network [red] mask [mascara]
```
```
show ip bgp summary               ! estado numérico = sesión activa
show ip bgp                       ! rutas con *> = mejor ruta
show ip route                     ! rutas marcadas con B
```

---

### RIPng (IPv6)
```
ipv6 unicast-routing              ! obligatorio, va primero

interface GigabitEthernet0/0
 ipv6 address [IPv6]/64
 ipv6 rip RIP1 enable

ipv6 router rip RIP1
```
```
show ipv6 route                   ! rutas marcadas con R
show ipv6 rip
show ipv6 rip database
show ipv6 interface brief
```

---

### OSPFv3 (IPv6)
```
ipv6 unicast-routing              ! obligatorio, va primero

ipv6 router ospf 1
 router-id 0.0.0.X               ! obligatorio, debe ser único

interface GigabitEthernet0/0
 ipv6 address [IPv6]/64
 ipv6 ospf 1 area 0
```
```
show ipv6 ospf neighbor           ! estado debe ser FULL
show ipv6 route                   ! rutas marcadas con O
show ipv6 ospf interface
show ipv6 ospf database
```

---

### Comandos útiles generales
```
show ip interface brief           ! estado de todas las interfaces (IPv4)
show ipv6 interface brief         ! estado de todas las interfaces (IPv6)
show running-config               ! configuración activa completa
no router rip                     ! eliminar RIPv1
no router ospf 1                  ! eliminar OSPFv2
no ipv6 router rip RIP1           ! eliminar RIPng
clear ip ospf process             ! reiniciar proceso OSPF (confirmar con yes)
clear ipv6 ospf process           ! reiniciar proceso OSPFv3
```
