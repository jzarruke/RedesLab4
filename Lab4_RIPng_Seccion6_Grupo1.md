# LABORATORIO 4 — RIPng
## Sección 6 · Grupo 1 (X = 6, Y = 1)
### Actividad 6.3 · Topología 2 — IPv6 (Guía 4)

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

### Redes de la topología

| Red (prefijo) | Dónde está |
|---------------|------------|
| 2001:ABCD:1435:A1::/64 | Enlace R1 — R2 |
| 2002:ABCD:1435:B1::/64 | Enlace R1 — R3 |
| 2002:ABCD:1435:C1::/64 | LAN de R2 (PC-1) |
| 2002:ABCD:1435:D1::/64 | LAN de R3 (PC-2) |

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red IPv6 |
|---|----------|---------------|----------|
| 1 | R1 **G0/0** → R2 **G0/0** | Copper Cross-Over | 2001:ABCD:1435:A1::/64 |
| 2 | R1 **G0/1** → R3 **G0/0** | Copper Cross-Over | 2002:ABCD:1435:B1::/64 |
| 3 | R2 **G0/1** → S1 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 4 | S1 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 5 | R3 **G0/1** → S3 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |
| 6 | S3 **Fa0/2** → PC-2 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |

> Los puertos del switch pueden ser cualquier FastEthernet libre. S1 y S3 son los nombres de los switches en la imagen de la hoja de ruta.

**Equipos:** 3 × Router 2911, 2 × Switch 2960-24TT, 2 × PC.

---

## 2. TABLA DE DIRECCIONAMIENTO (X=6, Y=1)

| Dispositivo | Interfaz | Dirección IPv6 / Prefijo | Gateway |
|-------------|----------|--------------------------|---------|
| R1 | G0/0 | **2001:ABCD:1435:A1::1/64** | N/A |
| R1 | G0/1 | **2002:ABCD:1435:B1::1/64** | N/A |
| R2 | G0/0 | **2001:ABCD:1435:A1::2/64** | N/A |
| R2 | G0/1 | **2002:ABCD:1435:C1::1/64** | N/A |
| R3 | G0/0 | **2002:ABCD:1435:B1::2/64** | N/A |
| R3 | G0/1 | **2002:ABCD:1435:D1::1/64** | N/A |
| PC-1 | NIC | **2002:ABCD:1435:C1::16/64** | 2002:ABCD:1435:C1::1 |
| PC-2 | NIC | **2002:ABCD:1435:D1::16/64** | 2002:ABCD:1435:D1::1 |

### Cálculo de valores

| Fórmula | Sustitución | Resultado |
|---------|-------------|-----------|
| ::Y | Y = 1 | **::1** → R1 en G0/0 y G0/1 |
| ::Y+1 | 1+1 = 2 | **::2** → R2 G0/0 y R3 G0/0 |
| ::YX | "1" seguido de "6" | **::16** → PCs |

> En IPv6 el "16" de `::16` se escribe tal cual (es la concatenación YX como en la Topología 1, aunque en hexadecimal equivale a 22 decimal). Así lo muestra el ejemplo de la hoja de ruta (Sección 5, Grupo 1 → `::15`).

---

## 3. DIFERENCIAS CLAVE: RIPv1 vs RIPng

| Aspecto | RIPv1 (IPv4) | RIPng (IPv6) |
|---------|-------------|--------------|
| Protocolo IP | IPv4 | IPv6 |
| Configuración | Global (`router rip` + `network`) | Por interfaz (`ipv6 rip <nombre> enable`) |
| Requisito previo | Ninguno | `ipv6 unicast-routing` |
| Envío de actualizaciones | Broadcast 255.255.255.255 | Multicast FF02::9 |
| Puerto | UDP 520 | UDP 521 |
| Classful/Classless | Classful | Classless (envía la longitud de prefijo) |
| Siguiente salto | IP del vecino | Dirección link-local (FE80::) del vecino |
| Distancia administrativa / máx. saltos | 120 / 15 | 120 / 15 |

> **Obligatorio antes de todo:** `ipv6 unicast-routing` en cada router. Sin esto el router no enruta IPv6 y RIPng no arranca.

---

## 4. CONFIGURACIÓN COMPLETA RIPng

El nombre del proceso (`RIP1`) es local al router; se usa el mismo en los tres por orden.

### PASO 1 — Configuración de R1

```
enable
configure terminal

hostname R1
no ip domain-lookup

ipv6 unicast-routing

ipv6 router rip RIP1

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::1/64
 ipv6 rip RIP1 enable
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::1/64
 ipv6 rip RIP1 enable
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

ipv6 router rip RIP1

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::2/64
 ipv6 rip RIP1 enable
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
 ipv6 rip RIP1 enable
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

ipv6 router rip RIP1

interface GigabitEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::2/64
 ipv6 rip RIP1 enable
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 rip RIP1 enable
 no shutdown

end
write memory
```

---

### PASO 4 — Configuración de PCs (IPv6)

> En Packet Tracer: PC → **Desktop** → **IP Configuration** → sección **IPv6 Configuration** → **Static**.

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

> El gateway va en el campo **IPv6 Default Gateway**, no en el de IPv4.

---

## 5. VALIDACIÓN

### PASO 5a — Interfaces IPv6

```
R1# show ipv6 interface brief
GigabitEthernet0/0     [up/up]
    FE80::...                    <- link-local (automática)
    2001:ABCD:1435:A1::1
GigabitEthernet0/1     [up/up]
    FE80::...
    2002:ABCD:1435:B1::1
```

### PASO 5b — Conectividad local (antes de RIPng)

```
R1# ping 2001:ABCD:1435:A1::2   ! R2 G0/0
R1# ping 2002:ABCD:1435:B1::2   ! R3 G0/0
PC-1> ping 2002:ABCD:1435:C1::1 ! gateway
PC-2> ping 2002:ABCD:1435:D1::1 ! gateway
```
Sin RIPng, `show ipv6 route` muestra solo rutas C y L, y PC-1 no llega a PC-2.

### PASO 5c — Tabla de enrutamiento IPv6 (esperar unos 30 segundos)

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
R   2002:ABCD:1435:C1::/64 [120/2]
     via FE80::..., GigabitEthernet0/0
R   2002:ABCD:1435:D1::/64 [120/2]
     via FE80::..., GigabitEthernet0/1
L   FF00::/8 [0/0]
     via Null0, receive
```

```
R2# show ipv6 route
R   2002:ABCD:1435:B1::/64 [120/2]
     via FE80::..., GigabitEthernet0/0
R   2002:ABCD:1435:D1::/64 [120/3]
     via FE80::..., GigabitEthernet0/0
```

```
R3# show ipv6 route
R   2001:ABCD:1435:A1::/64 [120/2]
     via FE80::..., GigabitEthernet0/0
R   2002:ABCD:1435:C1::/64 [120/3]
     via FE80::..., GigabitEthernet0/0
```

> En R2 y R3 solo se muestran las rutas R. En RIPng de Cisco el router suma 1 al recibir la ruta, por eso una red a un salto aparece con métrica 2. El siguiente salto es la dirección link-local del vecino.

### PASO 5d — Proceso RIPng

```
R1# show ipv6 protocols
R1# show ipv6 rip
RIP process "RIP1", port 521, multicast-group FF02::9, pid ...
     Administrative distance is 120. Maximum paths is 16
     Updates every 30 seconds, expire after 180
     ...
R1# show ipv6 rip database
```

### PASO 5e — Prueba final PC a PC

```
PC-1> ping 2002:ABCD:1435:D1::16   ! PC-2
PC-2> ping 2002:ABCD:1435:C1::16   ! PC-1
```

### PASO 5f — Captura de Wireshark

- Filtro para RIPng: `ripng` o `udp.port == 521`
- Filtro para el ping: `icmpv6.type == 128 || icmpv6.type == 129`
- En el informe: ICMPv6 Echo Request/Reply entre PC-1 y PC-2 y un paquete RIPng Response (origen link-local FE80::, destino FF02::9, prefijos con su longitud).

---

## 6. ERRORES COMUNES RIPng

### Error 1: `% IPv6 routing not enabled`
**Causa:** falta `ipv6 unicast-routing`.
```
R1(config)# ipv6 unicast-routing
```

### Error 2: Interfaz sin dirección o down
```
R1# show ipv6 interface brief
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ipv6 address 2001:ABCD:1435:A1::1/64
R1(config-if)# no shutdown
```

### Error 3: No aparecen rutas R
**Causa:** falta `ipv6 rip RIP1 enable` en alguna interfaz (también en la LAN, para que se anuncie).
```
R1# show ipv6 rip
R2# show ipv6 rip
R3# show ipv6 rip
R2# show running-config | include ipv6 rip
```

### Error 4: El PC no llega a su gateway
**Causas:** gateway puesto en el campo IPv4, prefijo distinto de 64 o dirección mal escrita.
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

### Conectividad remota (con RIPng convergido)
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | R2 G0/1 | `2002:ABCD:1435:C1::1` | Con RIPng |
| R1 | R3 G0/1 | `2002:ABCD:1435:D1::1` | Con RIPng |
| R2 | R3 G0/0 | `2002:ABCD:1435:B1::2` | Con RIPng |
| R3 | R2 G0/1 | `2002:ABCD:1435:C1::1` | Con RIPng |
| **PC-1** | **PC-2** | **`2002:ABCD:1435:D1::16`** | **Objetivo final** |
| **PC-2** | **PC-1** | **`2002:ABCD:1435:C1::16`** | **Objetivo final** |

---

## 8. CHECKLIST RIPng

- [ ] Topología construida y mapa de la topología hecho
- [ ] Hostnames R1, R2, R3
- [ ] `ipv6 unicast-routing` en R1, R2 y R3
- [ ] `ipv6 router rip RIP1` en los tres routers
- [ ] `ipv6 rip RIP1 enable` en G0/0 y G0/1 de cada router
- [ ] PC-1: 2002:ABCD:1435:C1::16/64, gateway 2002:ABCD:1435:C1::1
- [ ] PC-2: 2002:ABCD:1435:D1::16/64, gateway 2002:ABCD:1435:D1::1
- [ ] `show ipv6 route` en R1: 2 rutas C + 2 rutas R
- [ ] PC-1 hace ping a PC-2 y PC-2 a PC-1
- [ ] Captura de Wireshark: ICMPv6 exitoso + paquetes RIPng (UDP 521)

---

## 9. CONFIGURACIÓN ALTERNATIVA — FastEthernet

> Para routers con FastEthernet (Cisco 1841, 2811). Las direcciones y los comandos son los mismos.

### R1 — FastEthernet
```
enable
configure terminal
hostname R1
no ip domain-lookup
ipv6 unicast-routing
ipv6 router rip RIP1
interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::1/64
 ipv6 rip RIP1 enable
 no shutdown
interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::1/64
 ipv6 rip RIP1 enable
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
ipv6 router rip RIP1
interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::2/64
 ipv6 rip RIP1 enable
 no shutdown
interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
 ipv6 rip RIP1 enable
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
ipv6 router rip RIP1
interface FastEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::2/64
 ipv6 rip RIP1 enable
 no shutdown
interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 rip RIP1 enable
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

Para borrar solo RIPng:
```
enable
configure terminal
no ipv6 router rip RIP1
end
write memory
```
