# Lab 4 — Protocolos de Enrutamiento Dinámico IPv4 e IPv6
**Universidad de los Andes · Infraestructura de Comunicaciones**
**Sección 6 · Grupo 1 (X = 6, Y = 1)**

---

## Guías del laboratorio

| Actividad | Protocolo | Topología | Guía del curso | Archivo |
|-----------|-----------|-----------|----------------|---------|
| 6.1 | RIPv1 | Topología 1 — IPv4 | Guía 3 | [Lab4_RIPv1](Lab4_RIPv1_Seccion6_Grupo1.md) |
| 6.2 | OSPFv2 | Topología 1 — IPv4 | Guía 3 | [Lab4_OSPFv2](Lab4_OSPFv2_Seccion6_Grupo1.md) |
| 6.3 | RIPng | Topología 2 — IPv6 | Guía 4 | [Lab4_RIPng](Lab4_RIPng_Seccion6_Grupo1.md) |
| 6.4 | OSPFv3 | Topología 2 — IPv6 | Guía 4 | [Lab4_OSPFv3](Lab4_OSPFv3_Seccion6_Grupo1.md) |
| 6.5 | BGP | Topología 3 — IPv4 entre AS | Guía 5 | [Lab4_BGP](Lab4_BGP_Seccion6_Grupo1.md) |

Los enunciados y guías originales están en la carpeta `docs/`.

---

## Resumen de direcciones (X=6, Y=1)

### Topología 1 — RIPv1 y OSPF
| Dispositivo | Interfaz | IP | Máscara | Gateway |
|-------------|----------|----|---------|---------|
| R1 | G0/0 | 192.168.10.1 | /24 | — |
| R1 | G0/1 | 192.168.20.1 | /24 | — |
| R2 | G0/0 | 192.168.10.2 | /24 | — |
| R2 | G0/1 | 192.168.30.1 | /24 | — |
| R2 | Loopback2 | 172.16.6.1 | /16 | — |
| R3 | G0/0 | 192.168.20.2 | /24 | — |
| R3 | G0/1 | 192.168.40.1 | /24 | — |
| R3 | Loopback3 | 172.17.6.1 | /16 | — |
| PC-1 | NIC | 192.168.30.16 | /24 | 192.168.30.1 |
| PC-2 | NIC | 192.168.40.16 | /24 | 192.168.40.1 |

### Topología 2 — RIPng y OSPFv3
| Dispositivo | Interfaz | IPv6 | Gateway |
|-------------|----------|------|---------|
| R1 | G0/0 | 2001:ABCD:1435:A1::1/64 | — |
| R1 | G0/1 | 2002:ABCD:1435:B1::1/64 | — |
| R2 | G0/0 | 2001:ABCD:1435:A1::2/64 | — |
| R2 | G0/1 | 2002:ABCD:1435:C1::1/64 | — |
| R3 | G0/0 | 2002:ABCD:1435:B1::2/64 | — |
| R3 | G0/1 | 2002:ABCD:1435:D1::1/64 | — |
| PC-1 | NIC | 2002:ABCD:1435:C1::16/64 | 2002:ABCD:1435:C1::1 |
| PC-2 | NIC | 2002:ABCD:1435:D1::16/64 | 2002:ABCD:1435:D1::1 |

### Topología 3 — BGP
| Dispositivo | Interfaz | IP | Gateway | AS |
|-------------|----------|----|---------|----|
| R1 | G0/0 | 192.168.10.1/24 | — | 100 |
| R1 | G0/1 | 192.168.30.7/24 | — | 100 |
| R2 | G0/0 | 192.168.30.16/24 | — | 200 |
| R2 | G0/1 | 192.168.40.16/24 | — | 200 |
| R2 | G0/2 | 192.168.60.1/24 | — | 200 |
| R3 | G0/0 | 192.168.40.7/24 | — | 300 |
| R3 | G0/1 | 192.168.50.1/24 | — | 300 |
| PC-0 | NIC | 192.168.10.7/24 | 192.168.10.1 | 100 |
| PC-1 | NIC | 192.168.60.7/24 | 192.168.60.1 | 200 |
| PC-2 | NIC | 192.168.50.7/24 | 192.168.50.1 | 300 |

Reglas: X = sección, Y = grupo. "YX" es concatenación (grupo seguido de sección = 16); "Y+1", "X+1" y "X+Y" son sumas.

---

## Entregables

1. **Pre-entrega:** el `.pkt` de la topología, revisado con pings entre estaciones antes de acceder a los equipos.
2. **Entrega intermedia (en clase):** capturas de Wireshark con ping exitoso de 6.1, 6.2 y una actividad a elección entre 6.3, 6.4 y 6.5. Sin captura, ese numeral vale 0.
3. **Entrega final (PDF):** mapa de cada topología, foto de todos los integrantes en el laboratorio y análisis de las capturas (paquetes ICMP y del protocolo de enrutamiento).

| Entregable | Valor |
|------------|-------|
| 6.1 RIPv1 | 5% |
| 6.2 OSPF | 5% |
| 6.3 RIPng | 9% |
| 6.4 OSPFv3 | 9% |
| 6.5 BGP | 12% |
| Calidad del informe | 10% |
| Sustentación | 50% |

Al terminar hay que desmontar las topologías y guardar los cables.

---

## Filtros de Wireshark

| Protocolo | Filtro |
|-----------|--------|
| RIPv1 | `rip` o `udp.port == 520` |
| OSPFv2 / OSPFv3 | `ospf` (IPv4: `ip.proto == 89`) |
| RIPng | `ripng` o `udp.port == 521` |
| BGP | `bgp` o `tcp.port == 179` |
| Ping IPv4 | `icmp` |
| Ping IPv6 | `icmpv6.type == 128 \|\| icmpv6.type == 129` |

---

## Cheat sheet de comandos

### Configuración base (todos los routers)
```
enable
configure terminal
hostname R1
no ip domain-lookup
interface GigabitEthernet0/0
 ip address [IP] [MÁSCARA]
 no shutdown
end
write memory
```

### RIPv1
```
router rip
 version 1
 network [red-de-clase]           ! sin máscara, ej: network 192.168.10.0
```
```
show ip route                     ! rutas con R
show ip protocols
debug ip rip
```

### OSPFv2
```
router ospf 1
 network [red] [wildcard] area 0  ! ej: network 192.168.10.0 0.0.0.255 area 0
```
```
show ip ospf neighbor             ! estado FULL
show ip route                     ! rutas con O (las loopbacks salen como /32)
show ip ospf interface brief
show ip ospf database
```

### RIPng
```
ipv6 unicast-routing              ! obligatorio, va primero
ipv6 router rip RIP1
interface GigabitEthernet0/0
 ipv6 address [IPv6]/64
 ipv6 rip RIP1 enable
```
```
show ipv6 route                   ! rutas con R
show ipv6 rip
show ipv6 rip database
show ipv6 interface brief
```

### OSPFv3
```
ipv6 unicast-routing              ! obligatorio, va primero
ipv6 router ospf 1
 router-id 1.1.1.1                ! obligatorio y único (2.2.2.2 en R2, 3.3.3.3 en R3)
interface GigabitEthernet0/0
 ipv6 address [IPv6]/64
 ipv6 ospf 1 area 0
```
```
show ipv6 ospf neighbor           ! estado FULL
show ipv6 route                   ! rutas con O
show ipv6 ospf interface brief
show ipv6 ospf database
```

### BGP
```
router bgp [AS-propio]
 neighbor [IP-vecino] remote-as [AS-vecino]
 network [red] mask [máscara]     ! todas las redes conectadas del router
```
```
show ip bgp summary               ! número en la última columna = Established
show ip bgp                       ! *> = mejor ruta; columna Path = AS_PATH
show ip route                     ! rutas con B [20/0]
```

### Comandos útiles generales
```
show ip interface brief           ! estado de interfaces IPv4
show ipv6 interface brief         ! estado de interfaces IPv6
show running-config
no router rip                     ! eliminar RIPv1
no router ospf 1                  ! eliminar OSPFv2
no ipv6 router rip RIP1           ! eliminar RIPng
no ipv6 router ospf 1             ! eliminar OSPFv3
no router bgp [AS]                ! eliminar BGP
clear ip ospf process             ! reiniciar OSPF (confirmar con yes)
clear ipv6 ospf process           ! reiniciar OSPFv3
clear ip bgp *                    ! reiniciar sesiones BGP
write erase + reload              ! borrar todo y reiniciar
```
