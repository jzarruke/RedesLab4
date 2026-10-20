# LABORATORIO 4 — BGP
## Sección 6 · Grupo 1 (X = 6, Y = 1)
### Actividad 6.5 · Topología 3 — IPv4 entre sistemas autónomos (Guía 5)

---

## 1. TOPOLOGÍA

### Mapa de la topología

```
        AS 100                         AS 200                          AS 300

 [PC0]--[Switch2]--G0/0 [R1] G0/1 ---------- G0/0 [R2] G0/1 ---------- G0/0 [R3] G0/1--[Switch1]--[PC2]
   192.168.10.0/24          192.168.30.0/24     |     192.168.40.0/24           192.168.50.0/24
                                                | G0/2
                                            [Switch0]
                                                |
                                             [PC1]
                                         192.168.60.0/24
```

### Redes de la topología

| Red | Dónde está | AS |
|-----|-----------|----|
| 192.168.10.0/24 | LAN de R1 (PC-0) | 100 |
| 192.168.30.0/24 | Enlace R1 — R2 (eBGP) | 100 / 200 |
| 192.168.40.0/24 | Enlace R2 — R3 (eBGP) | 200 / 300 |
| 192.168.60.0/24 | LAN de R2 (PC-1) | 200 |
| 192.168.50.0/24 | LAN de R3 (PC-2) | 300 |

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red | AS |
|---|----------|---------------|-----|-----|
| 1 | R1 **G0/0** → Switch2 **Fa0/1** | Copper Straight-Through | 192.168.10.0/24 | 100 |
| 2 | Switch2 **Fa0/2** → PC-0 **Fa0** | Copper Straight-Through | 192.168.10.0/24 | 100 |
| 3 | R1 **G0/1** → R2 **G0/0** | Copper Cross-Over | 192.168.30.0/24 | 100 / 200 |
| 4 | R2 **G0/1** → R3 **G0/0** | Copper Cross-Over | 192.168.40.0/24 | 200 / 300 |
| 5 | R2 **G0/2** → Switch0 **Fa0/1** | Copper Straight-Through | 192.168.60.0/24 | 200 |
| 6 | Switch0 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 192.168.60.0/24 | 200 |
| 7 | R3 **G0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 192.168.50.0/24 | 300 |
| 8 | Switch1 **Fa0/2** → PC-2 **Fa0** | Copper Straight-Through | 192.168.50.0/24 | 300 |

**Equipos requeridos:**
- 3 × Router Cisco 2911 (R2 necesita 3 interfaces GigabitEthernet; el 2911 las trae)
- 3 × Switch Cisco 2960-24TT
- 3 × PC

---

## 2. TABLA DE DIRECCIONAMIENTO (X=6, Y=1)

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway | AS |
|-------------|----------|--------------|---------|---------|-----|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | N/A | 100 |
| R1 | G0/1 | 192.168.30.**7** | 255.255.255.0 | N/A | 100 |
| R2 | G0/0 | 192.168.30.**16** | 255.255.255.0 | N/A | 200 |
| R2 | G0/1 | 192.168.40.**16** | 255.255.255.0 | N/A | 200 |
| R2 | G0/2 | 192.168.60.1 | 255.255.255.0 | N/A | 200 |
| R3 | G0/0 | 192.168.40.**7** | 255.255.255.0 | N/A | 300 |
| R3 | G0/1 | 192.168.50.1 | 255.255.255.0 | N/A | 300 |
| PC-0 | NIC | 192.168.10.**7** | 255.255.255.0 | 192.168.10.1 | 100 |
| PC-1 | NIC | 192.168.60.**7** | 255.255.255.0 | 192.168.60.1 | 200 |
| PC-2 | NIC | 192.168.50.**7** | 255.255.255.0 | 192.168.50.1 | 300 |

### Cálculo de valores

| Fórmula | Sustitución | Resultado |
|---------|-------------|-----------|
| X+1 | 6+1 (suma) | **7** → R1 G0/1 y R3 G0/0 |
| YX | "1" seguido de "6" (concatenación) | **16** → R2 G0/0 y R2 G0/1 |
| X+Y | 6+1 (suma) | **7** → PC-0, PC-1 y PC-2 |

> Coincidencia: X+1 y X+Y dan 7 para este grupo. No hay conflicto porque están en redes distintas (192.168.30.7 y 192.168.10.7, por ejemplo).

---

## 3. CONCEPTO BGP

BGP (Border Gateway Protocol) es el único EGP (Exterior Gateway Protocol) en uso: enruta **entre** sistemas autónomos y es el protocolo de enrutamiento de Internet. Es un protocolo de **vector de rutas**: cada ruta lleva el atributo AS_PATH con la lista de sistemas autónomos que atraviesa.

| Aspecto | RIP | OSPF | BGP |
|---------|-----|------|-----|
| Tipo | Vector distancia | Estado de enlace | Vector de rutas |
| Ámbito | Dentro de un AS (IGP) | Dentro de un AS (IGP) | Entre AS (EGP) |
| Criterio de selección | Saltos | Costo | Atributos y políticas (AS_PATH) |
| Transporte | UDP 520 | IP protocolo 89 | TCP puerto 179 |
| Vecinos | Automáticos | Mensajes Hello | Manuales (`neighbor`) |
| Distancia administrativa | 120 | 110 | 20 (eBGP) / 200 (iBGP) |

**Mensajes BGP:** OPEN (abre la sesión), UPDATE (anuncia o retira rutas), KEEPALIVE (confirma que el vecino sigue activo), NOTIFICATION (error y cierre).

**Estados de la sesión:** Idle → Connect → Active → OpenSent → OpenConfirm → **Established**. Solo en Established se intercambian rutas, y el IOS muestra `%BGP-5-ADJCHANGE: neighbor ... Up`.

En esta topología hay **3 sistemas autónomos:**
- **AS 100** → R1 + PC-0
- **AS 200** → R2 + PC-1 (AS de tránsito: conecta a los otros dos)
- **AS 300** → R3 + PC-2

Todas las sesiones son **eBGP** porque cada vecino está en un AS distinto.

---

## 4. CONFIGURACIÓN COMPLETA BGP

Cada router anuncia **todas sus redes conectadas directamente** (su LAN y sus enlaces), como en el ejemplo de R3 de la Guía 5. El comando `network ... mask` debe coincidir exactamente con una red de la tabla de enrutamiento.

### PASO 1 — Configuración de R1 (AS 100)

```
enable
configure terminal

hostname R1
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.30.7 255.255.255.0
 no shutdown

router bgp 100
 neighbor 192.168.30.16 remote-as 200
 network 192.168.10.0 mask 255.255.255.0
 network 192.168.30.0 mask 255.255.255.0

end
write memory
```

---

### PASO 2 — Configuración de R2 (AS 200)

```
enable
configure terminal

hostname R2
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.30.16 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.40.16 255.255.255.0
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.60.1 255.255.255.0
 no shutdown

router bgp 200
 neighbor 192.168.30.7 remote-as 100
 neighbor 192.168.40.7 remote-as 300
 network 192.168.30.0 mask 255.255.255.0
 network 192.168.40.0 mask 255.255.255.0
 network 192.168.60.0 mask 255.255.255.0

end
write memory
```

---

### PASO 3 — Configuración de R3 (AS 300)

```
enable
configure terminal

hostname R3
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.40.7 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.50.1 255.255.255.0
 no shutdown

router bgp 300
 neighbor 192.168.40.16 remote-as 200
 network 192.168.40.0 mask 255.255.255.0
 network 192.168.50.0 mask 255.255.255.0

end
write memory
```

---

### PASO 4 — Configuración de PCs

**PC-0:**
```
IP Address  : 192.168.10.7
Subnet Mask : 255.255.255.0
Gateway     : 192.168.10.1
```

**PC-1:**
```
IP Address  : 192.168.60.7
Subnet Mask : 255.255.255.0
Gateway     : 192.168.60.1
```

**PC-2:**
```
IP Address  : 192.168.50.7
Subnet Mask : 255.255.255.0
Gateway     : 192.168.50.1
```

---

## 5. VALIDACIÓN

### PASO 5a — Conectividad local antes de BGP

```
R1# ping 192.168.30.16   ! R2 G0/0
R2# ping 192.168.40.7    ! R3 G0/0
PC-0> ping 192.168.10.1  ! gateway
PC-1> ping 192.168.60.1  ! gateway
PC-2> ping 192.168.50.1  ! gateway
```
Si los routers vecinos no se hacen ping, la sesión BGP no va a subir: revisar esto primero.

### PASO 5b — Sesiones BGP (esperar unos 30 a 60 segundos)

```
R1# show ip bgp summary
BGP router identifier 192.168.30.7, local AS number 100
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.30.16   4   200      10      10        7    0    0 00:01:30        4
```

```
R2# show ip bgp summary
BGP router identifier 192.168.60.1, local AS number 200
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.30.7    4   100      10      10        7    0    0 00:01:30        2
192.168.40.7    4   300      10      10        7    0    0 00:01:30        2
```

```
R3# show ip bgp summary
BGP router identifier 192.168.50.1, local AS number 300
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.40.16   4   200      10      10        7    0    0 00:01:30        4
```

> La última columna debe ser un **número** (prefijos recibidos). Si dice `Idle` o `Active`, la sesión no se estableció. El Router-ID es la IP más alta del router. Los contadores de mensajes y el número exacto de prefijos pueden variar un poco.

### PASO 5c — Tabla BGP

```
R1# show ip bgp
   Network          Next Hop            Metric LocPrf Weight Path
*> 192.168.10.0     0.0.0.0                  0         32768 i
*  192.168.30.0     192.168.30.16            0             0 200 i
*>                  0.0.0.0                  0         32768 i
*> 192.168.40.0     192.168.30.16            0             0 200 i
*> 192.168.50.0     192.168.30.16                          0 200 300 i
*> 192.168.60.0     192.168.30.16            0             0 200 i
```

> `*>` indica la mejor ruta. La red 192.168.30.0 aparece dos veces porque la anuncian R1 y R2; R1 prefiere la suya (Weight 32768). La columna Path muestra el AS_PATH: para llegar a 192.168.50.0 (LAN de R3) se atraviesa el AS 200 y luego el AS 300, como dice la Guía 5.

```
R2# show ip bgp
R3# show ip bgp
```
En R3, la red 192.168.10.0 debe aparecer con Path `200 100`.

### PASO 5d — Tabla de enrutamiento

```
R1# show ip route
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
C    192.168.30.0/24 is directly connected, GigabitEthernet0/1
B    192.168.40.0/24 [20/0] via 192.168.30.16, 00:01:30
B    192.168.50.0/24 [20/0] via 192.168.30.16, 00:01:30
B    192.168.60.0/24 [20/0] via 192.168.30.16, 00:01:30
```

> Las rutas BGP aparecen con **B** y distancia administrativa **20** (eBGP). Se omitieron las líneas L.

### PASO 5e — Prueba final PC a PC

```
PC-0> ping 192.168.60.7   ! PC-1
PC-0> ping 192.168.50.7   ! PC-2
PC-1> ping 192.168.10.7   ! PC-0
PC-1> ping 192.168.50.7   ! PC-2
PC-2> ping 192.168.10.7   ! PC-0
PC-2> ping 192.168.60.7   ! PC-1
```

### PASO 5f — Captura de Wireshark

- Filtro para BGP: `bgp` o `tcp.port == 179`
- Filtro para el ping: `icmp`
- En el informe: el ICMP entre dos PCs de distinto AS, el three-way handshake TCP hacia el puerto 179 y los mensajes OPEN (con el número de AS), UPDATE (con los prefijos y el AS_PATH) y KEEPALIVE. Para ver OPEN y UPDATE hay que capturar mientras sube la sesión (o forzarla con `clear ip bgp *`).

---

## 6. ERRORES COMUNES BGP

### Error 1: La sesión queda en Idle o Active
**Causas:** la IP del `neighbor` no es la de la interfaz del vecino, el `remote-as` no coincide con el AS del vecino o la interfaz está apagada.
```
R1# ping 192.168.30.16
R1# show running-config | section router bgp
R1# show ip bgp summary
```
Para corregir un vecino mal escrito:
```
R1(config)# router bgp 100
R1(config-router)# no neighbor 192.168.30.61 remote-as 200
R1(config-router)# neighbor 192.168.30.16 remote-as 200
```

### Error 2: Una red no se anuncia
**Causa:** el `network ... mask` no coincide exactamente con la red en la tabla de enrutamiento (por ejemplo, máscara equivocada o la IP de la interfaz en lugar de la red).
```
R3(config)# router bgp 300
R3(config-router)# network 192.168.50.0 mask 255.255.255.0
```

### Error 3: Las sesiones están establecidas pero el ping falla
**Causa:** falta el `network` de alguna LAN en un router, así que el paquete llega pero la respuesta no tiene ruta de regreso. Revisar `show ip bgp` en los tres routers.

### Error 4: El AS del router está mal
`router bgp` solo admite un proceso por router. Si se creó con el número equivocado:
```
R1(config)# no router bgp 200
R1(config)# router bgp 100
```

### Reiniciar las sesiones
```
R2# clear ip bgp *
```

---

## 7. TABLA DE PINGS — REFERENCIA RÁPIDA

### Conectividad directa (antes de BGP)
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | R2 G0/0 | `192.168.30.16` | Siempre |
| R2 | R1 G0/1 | `192.168.30.7` | Siempre |
| R2 | R3 G0/0 | `192.168.40.7` | Siempre |
| R3 | R2 G0/1 | `192.168.40.16` | Siempre |
| PC-0 | Gateway R1 | `192.168.10.1` | Siempre |
| PC-1 | Gateway R2 | `192.168.60.1` | Siempre |
| PC-2 | Gateway R3 | `192.168.50.1` | Siempre |

### Conectividad remota (con BGP Established)
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | R3 G0/0 | `192.168.40.7` | Con BGP |
| R3 | R1 G0/1 | `192.168.30.7` | Con BGP |
| **PC-0** | **PC-1** | **`192.168.60.7`** | **Objetivo final** |
| **PC-0** | **PC-2** | **`192.168.50.7`** | **Objetivo final** |
| **PC-1** | **PC-2** | **`192.168.50.7`** | **Objetivo final** |
| **PC-2** | **PC-0** | **`192.168.10.7`** | **Objetivo final** |

---

## 8. CHECKLIST BGP

- [ ] Topología construida y mapa de la topología hecho (con los AS marcados)
- [ ] Hostnames R1, R2, R3
- [ ] R1: `router bgp 100`, neighbor 192.168.30.16 remote-as 200, network 192.168.10.0 y 192.168.30.0
- [ ] R2: `router bgp 200`, neighbors 192.168.30.7 (AS 100) y 192.168.40.7 (AS 300), network 192.168.30.0, 192.168.40.0 y 192.168.60.0
- [ ] R3: `router bgp 300`, neighbor 192.168.40.16 remote-as 200, network 192.168.40.0 y 192.168.50.0
- [ ] PC-0: 192.168.10.7/24, gateway 192.168.10.1
- [ ] PC-1: 192.168.60.7/24, gateway 192.168.60.1
- [ ] PC-2: 192.168.50.7/24, gateway 192.168.50.1
- [ ] `show ip bgp summary` con números (no Idle/Active) en los tres routers
- [ ] `show ip bgp` en R1 muestra 192.168.50.0 con Path `200 300`
- [ ] `show ip route` muestra rutas B con [20/0]
- [ ] Pings entre los tres PCs exitosos
- [ ] Captura de Wireshark: ICMP exitoso + paquetes BGP (TCP 179)

---

## 9. CONFIGURACIÓN ALTERNATIVA — FastEthernet

> Para routers con FastEthernet (Cisco 1841, 2811). **Ojo:** R2 necesita 3 interfaces y estos modelos traen solo 2 FastEthernet integradas. Hace falta un módulo adicional (por ejemplo, una HWIC-1FE o una tarjeta con puerto Ethernet extra); el nombre de la tercera interfaz depende del módulo (por ejemplo `FastEthernet0/0/0`). Revisar con `show ip interface brief`.

### R1 — FastEthernet
```
enable
configure terminal
hostname R1
no ip domain-lookup
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.30.7 255.255.255.0
 no shutdown
router bgp 100
 neighbor 192.168.30.16 remote-as 200
 network 192.168.10.0 mask 255.255.255.0
 network 192.168.30.0 mask 255.255.255.0
end
write memory
```

### R2 — FastEthernet (tercera interfaz según el módulo)
```
enable
configure terminal
hostname R2
no ip domain-lookup
interface FastEthernet0/0
 ip address 192.168.30.16 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.40.16 255.255.255.0
 no shutdown
interface FastEthernet0/0/0
 ip address 192.168.60.1 255.255.255.0
 no shutdown
router bgp 200
 neighbor 192.168.30.7 remote-as 100
 neighbor 192.168.40.7 remote-as 300
 network 192.168.30.0 mask 255.255.255.0
 network 192.168.40.0 mask 255.255.255.0
 network 192.168.60.0 mask 255.255.255.0
end
write memory
```

### R3 — FastEthernet
```
enable
configure terminal
hostname R3
no ip domain-lookup
interface FastEthernet0/0
 ip address 192.168.40.7 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.50.1 255.255.255.0
 no shutdown
router bgp 300
 neighbor 192.168.40.16 remote-as 200
 network 192.168.40.0 mask 255.255.255.0
 network 192.168.50.0 mask 255.255.255.0
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

Para borrar solo BGP (usar el AS de ese router):
```
enable
configure terminal
no router bgp 100
end
write memory
```
