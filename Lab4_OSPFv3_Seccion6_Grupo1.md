# LABORATORIO 4 — OSPFv3
## Sección 6 · Grupo 1 (X = 6, Y = 1)
### Actividad 6.4 · Topología 2 — IPv6 (Guía 4)

---

## 1. TOPOLOGÍA

### Mapa de la topología

```
                                   [R1]
                              G0/0 /    \ G0/1
                                  /      \
              2001:ABCD:1435:A1::/64    2002:ABCD:1435:B1::/64
                                /          \
                          G0/0 /            \ G0/0
                            [R2]            [R3]
                         G0/1 |               | G0/1
       2002:ABCD:1435:C1::/64 |               | 2002:ABCD:1435:D1::/64
                            [S1]            [S3]
                              |               |
                           [PC-1]          [PC-2]
                C1::16 (gw C1::1)          D1::16 (gw D1::1)
```

Es la misma topología de RIPng: solo cambia el protocolo.

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red IPv6 |
|---|----------|---------------|----------|
| 1 | R1 **G0/0** → R2 **G0/0** | Copper Cross-Over | 2001:ABCD:1435:A1::/64 |
| 2 | R1 **G0/1** → R3 **G0/0** | Copper Cross-Over | 2002:ABCD:1435:B1::/64 |
| 3 | R2 **G0/1** → S1 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 4 | S1 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 5 | R3 **G0/1** → S3 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |
| 6 | S3 **Fa0/2** → PC-2 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |

---

## 2. TABLA DE DIRECCIONAMIENTO (X=6, Y=1)

| Dispositivo | Interfaz | Dirección IPv6 / Prefijo | Gateway |
|-------------|----------|--------------------------|---------|
| R1 | G0/0 | 2001:ABCD:1435:A1::1/64 | N/A |
| R1 | G0/1 | 2002:ABCD:1435:B1::1/64 | N/A |
| R2 | G0/0 | 2001:ABCD:1435:A1::2/64 | N/A |
| R2 | G0/1 | 2002:ABCD:1435:C1::1/64 | N/A |
| R3 | G0/0 | 2002:ABCD:1435:B1::2/64 | N/A |
| R3 | G0/1 | 2002:ABCD:1435:D1::1/64 | N/A |
| PC-1 | NIC | 2002:ABCD:1435:C1::16/64 | 2002:ABCD:1435:C1::1 |
| PC-2 | NIC | 2002:ABCD:1435:D1::16/64 | 2002:ABCD:1435:D1::1 |

> El cálculo de cada dirección está en `Lab4_RIPng_Seccion6_Grupo1.md`.

### Router-ID de cada router

| Router | Router-ID |
|--------|-----------|
| R1 | 1.1.1.1 |
| R2 | 2.2.2.2 |
| R3 | 3.3.3.3 |

> Como no hay direcciones IPv4 en los routers, el Router-ID (32 bits, formato de IPv4) se debe asignar a mano. La Guía 4 sugiere el formato 1.1.1.1, 2.2.2.2, 3.3.3.3. Debe ser único en cada router.

---

## 3. DIFERENCIAS CLAVE: OSPFv2 vs OSPFv3

| Aspecto | OSPFv2 (IPv4) | OSPFv3 (IPv6) |
|---------|--------------|---------------|
| Redes | IPv4 | IPv6 |
| Habilitar en interfaces | Global (`network` + wildcard) | Por interfaz (`ipv6 ospf 1 area 0`) |
| Router-ID | Automático (IP más alta o loopback) | Manual si no hay IPv4 (`router-id`) |
| Multicast de vecinos | 224.0.0.5 / 224.0.0.6 | FF02::5 / FF02::6 |
| Transporte | IP protocolo 89 | IPv6, siguiente encabezado 89 |
| Siguiente salto en la tabla | IP del vecino | Link-local FE80:: del vecino |
| Métrica / AD | Costo / 110 | Costo / 110 |

---

## 4. CONFIGURACIÓN COMPLETA OSPFv3

> Si vienen de RIPng en los mismos routers, primero `no ipv6 router rip RIP1`. El orden importa: crear el proceso con su `router-id` **antes** de poner `ipv6 ospf 1 area 0` en las interfaces.

### PASO 1 — Configuración de R1

```
enable
configure terminal

hostname R1
no ip domain-lookup

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 1.1.1.1
 exit

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::1/64
 ipv6 ospf 1 area 0
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::1/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

---

### PASO 2 — Configuración de R2

```
enable
configure terminal

hostname R2
no ip domain-lookup

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 2.2.2.2
 exit

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::2/64
 ipv6 ospf 1 area 0
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

---

### PASO 3 — Configuración de R3

```
enable
configure terminal

hostname R3
no ip domain-lookup

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 3.3.3.3
 exit

interface GigabitEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::2/64
 ipv6 ospf 1 area 0
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

---

### PASO 4 — Configuración de PCs (IPv6)

**PC-1:**
```
IPv6 Address         : 2002:ABCD:1435:C1::16
Prefix Length        : 64
IPv6 Default Gateway : 2002:ABCD:1435:C1::1
```

**PC-2:**
```
IPv6 Address         : 2002:ABCD:1435:D1::16
Prefix Length        : 64
IPv6 Default Gateway : 2002:ABCD:1435:D1::1
```

---

## 5. VALIDACIÓN

### PASO 5a — Router-ID

```
R1# show ipv6 ospf
 Routing Process "ospfv3 1" with ID 1.1.1.1
```

### PASO 5b — Vecinos OSPFv3

```
R1# show ipv6 ospf neighbor
Neighbor ID     Pri   State      Dead Time   Interface ID    Interface
2.2.2.2           1   FULL/DR    00:00:35    1               GigabitEthernet0/0
3.3.3.3           1   FULL/DR    00:00:35    1               GigabitEthernet0/1
```

```
R2# show ipv6 ospf neighbor
Neighbor ID     Pri   State      Dead Time   Interface ID    Interface
1.1.1.1           1   FULL/BDR   00:00:35    1               GigabitEthernet0/0
```

```
R3# show ipv6 ospf neighbor
Neighbor ID     Pri   State      Dead Time   Interface ID    Interface
1.1.1.1           1   FULL/BDR   00:00:35    2               GigabitEthernet0/0
```

> Lo obligatorio es **FULL**. Si arrancan al tiempo, el DR de cada enlace es el de Router-ID más alto (R2 y R3); si uno arrancó antes puede quedar al revés, y eso no es un error.

### PASO 5c — Tabla de enrutamiento IPv6

```
R1# show ipv6 route
C   2001:ABCD:1435:A1::/64 [0/0]
     via GigabitEthernet0/0, directly connected
L   2001:ABCD:1435:A1::1/128 [0/0]
     via GigabitEthernet0/0, receive
C   2002:ABCD:1435:B1::/64 [0/0]
     via GigabitEthernet0/1, directly connected
L   2002:ABCD:1435:B1::1/128 [0/0]
     via GigabitEthernet0/1, receive
O   2002:ABCD:1435:C1::/64 [110/2]
     via FE80::..., GigabitEthernet0/0
O   2002:ABCD:1435:D1::/64 [110/2]
     via FE80::..., GigabitEthernet0/1
L   FF00::/8 [0/0]
     via Null0, receive
```

```
R2# show ipv6 route ospf
O   2002:ABCD:1435:B1::/64 [110/2]
     via FE80::..., GigabitEthernet0/0
O   2002:ABCD:1435:D1::/64 [110/3]
     via FE80::..., GigabitEthernet0/0
```

```
R3# show ipv6 route ospf
O   2001:ABCD:1435:A1::/64 [110/2]
     via FE80::..., GigabitEthernet0/0
O   2002:ABCD:1435:C1::/64 [110/3]
     via FE80::..., GigabitEthernet0/0
```

### PASO 5d — Diagnóstico adicional

```
R1# show ipv6 protocols
R1# show ipv6 ospf interface brief
R1# show ipv6 ospf database
```

### PASO 5e — Prueba final

```
R1# ping 2002:ABCD:1435:C1::1      ! R2 G0/1
R1# ping 2002:ABCD:1435:D1::1      ! R3 G0/1
PC-1> ping 2002:ABCD:1435:D1::16   ! PC-2
PC-2> ping 2002:ABCD:1435:C1::16   ! PC-1
```

### PASO 5f — Captura de Wireshark

- Filtro para OSPFv3: `ospf` (Wireshark distingue la versión 3 en el encabezado)
- Filtro para el ping: `icmpv6.type == 128 || icmpv6.type == 129`
- En el informe: ICMPv6 Echo Request/Reply entre PC-1 y PC-2 y paquetes OSPFv3 Hello (origen FE80::, destino FF02::5, Router-ID 1.1.1.1 / 2.2.2.2 / 3.3.3.3, área 0).

---

## 6. ERRORES COMUNES OSPFv3

### Error 1: `%OSPFv3: Router process 1 is not running, please configure a router-id`
**Causa:** no se configuró `router-id` y no hay IPv4 en el router.
```
R1(config)# ipv6 router ospf 1
R1(config-rtr)# router-id 1.1.1.1
R1# clear ipv6 ospf process
```

### Error 2: No aparecen vecinos o quedan en INIT/EXSTART
**Causas:** falta `ipv6 unicast-routing`, falta `ipv6 ospf 1 area 0` en la interfaz, o dos routers con el mismo Router-ID.
```
R1# show running-config | include ipv6 unicast-routing
R1# show ipv6 ospf interface brief
R1# show ipv6 ospf | include ID
```

### Error 3: Una LAN no aparece con O
**Causa:** falta `ipv6 ospf 1 area 0` en la interfaz de la LAN.
```
R2(config)# interface GigabitEthernet0/1
R2(config-if)# ipv6 ospf 1 area 0
```

### Error 4: El PC no llega a su gateway
**Causas:** gateway en el campo IPv4 o prefijo distinto de 64.
```
R2# show ipv6 interface GigabitEthernet0/1
```

---

## 7. TABLA DE PINGS — REFERENCIA RÁPIDA

### Conectividad directa
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | R2 G0/0 | `2001:ABCD:1435:A1::2` | Siempre |
| R1 | R3 G0/0 | `2002:ABCD:1435:B1::2` | Siempre |
| R2 | R1 G0/0 | `2001:ABCD:1435:A1::1` | Siempre |
| R3 | R1 G0/1 | `2002:ABCD:1435:B1::1` | Siempre |
| PC-1 | Gateway R2 | `2002:ABCD:1435:C1::1` | Siempre |
| PC-2 | Gateway R3 | `2002:ABCD:1435:D1::1` | Siempre |

### Conectividad remota (con OSPFv3 en FULL)
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | R2 G0/1 | `2002:ABCD:1435:C1::1` | Con OSPFv3 |
| R1 | R3 G0/1 | `2002:ABCD:1435:D1::1` | Con OSPFv3 |
| R2 | R3 G0/0 | `2002:ABCD:1435:B1::2` | Con OSPFv3 |
| R3 | R2 G0/1 | `2002:ABCD:1435:C1::1` | Con OSPFv3 |
| **PC-1** | **PC-2** | **`2002:ABCD:1435:D1::16`** | **Objetivo final** |
| **PC-2** | **PC-1** | **`2002:ABCD:1435:C1::16`** | **Objetivo final** |

---

## 8. CHECKLIST OSPFv3

- [ ] Topología construida y mapa de la topología hecho
- [ ] Hostnames R1, R2, R3
- [ ] `ipv6 unicast-routing` en R1, R2 y R3
- [ ] R1: `ipv6 router ospf 1` con `router-id 1.1.1.1`
- [ ] R2: `ipv6 router ospf 1` con `router-id 2.2.2.2`
- [ ] R3: `ipv6 router ospf 1` con `router-id 3.3.3.3`
- [ ] `ipv6 ospf 1 area 0` en G0/0 y G0/1 de cada router
- [ ] PC-1: 2002:ABCD:1435:C1::16/64, gateway 2002:ABCD:1435:C1::1
- [ ] PC-2: 2002:ABCD:1435:D1::16/64, gateway 2002:ABCD:1435:D1::1
- [ ] `show ipv6 ospf neighbor` en R1: 2 vecinos en **FULL**
- [ ] `show ipv6 route` en R1: 2 rutas C + 2 rutas O
- [ ] PC-1 hace ping a PC-2 y PC-2 a PC-1
- [ ] Captura de Wireshark: ICMPv6 exitoso + paquetes OSPFv3

---

## 9. CONFIGURACIÓN ALTERNATIVA — FastEthernet

> Para routers con FastEthernet (Cisco 1841, 2811). Las direcciones, los Router-ID y los comandos son los mismos.

### R1 — FastEthernet
```
enable
configure terminal
hostname R1
no ip domain-lookup
ipv6 unicast-routing
ipv6 router ospf 1
 router-id 1.1.1.1
 exit
interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::1/64
 ipv6 ospf 1 area 0
 no shutdown
interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::1/64
 ipv6 ospf 1 area 0
 no shutdown
end
write memory
```

### R2 — FastEthernet
```
enable
configure terminal
hostname R2
no ip domain-lookup
ipv6 unicast-routing
ipv6 router ospf 1
 router-id 2.2.2.2
 exit
interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::2/64
 ipv6 ospf 1 area 0
 no shutdown
interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
 ipv6 ospf 1 area 0
 no shutdown
end
write memory
```

### R3 — FastEthernet
```
enable
configure terminal
hostname R3
no ip domain-lookup
ipv6 unicast-routing
ipv6 router ospf 1
 router-id 3.3.3.3
 exit
interface FastEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::2/64
 ipv6 ospf 1 area 0
 no shutdown
interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 ospf 1 area 0
 no shutdown
end
write memory
```

---

## 10. BORRAR TODA LA CONFIGURACIÓN

```
enable
write erase
reload
```

Para borrar solo OSPFv3:
```
enable
configure terminal
no ipv6 router ospf 1
end
write memory
```
