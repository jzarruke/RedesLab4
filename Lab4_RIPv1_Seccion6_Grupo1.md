# LABORATORIO 4 — RIPv1
## Sección 6 · Grupo 1 (X = 6, Y = 1)
### Actividad 6.1 · Topología 1 — IPv4 (Guía 3)

---

## 1. TOPOLOGÍA

### Mapa de la topología

```
   [PC-1]                                                          [PC-2]
192.168.30.16                                                  192.168.40.16
      |                                                               |
  [Switch0]                                                       [Switch1]
      | 192.168.30.0/24                               192.168.40.0/24 |
      |                                                               |
 G0/1 [R2] G0/0 ---- 192.168.10.0/24 ---- G0/0 [R1] G0/1 ---- 192.168.20.0/24 ---- G0/0 [R3] G0/1
      |                                                                                |
  Loopback2: 172.16.0.0/16                                               Loopback3: 172.17.0.0/16
```

R1 es el router central y no tiene LAN propia. R2 y R3 tienen cada uno una LAN y una loopback.

### Redes de la topología

| Red | Máscara | Dónde está |
|-----|---------|------------|
| 192.168.10.0 | /24 | Enlace R1 — R2 |
| 192.168.20.0 | /24 | Enlace R1 — R3 |
| 192.168.30.0 | /24 | LAN de R2 (PC-1) |
| 192.168.40.0 | /24 | LAN de R3 (PC-2) |
| 172.16.0.0 | /16 | Loopback 2 de R2 |
| 172.17.0.0 | /16 | Loopback 3 de R3 |

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red |
|---|----------|---------------|-----|
| 1 | R1 **G0/0** → R2 **G0/0** | Copper Cross-Over | 192.168.10.0/24 |
| 2 | R1 **G0/1** → R3 **G0/0** | Copper Cross-Over | 192.168.20.0/24 |
| 3 | R2 **G0/1** → Switch0 **Fa0/1** | Copper Straight-Through | 192.168.30.0/24 |
| 4 | Switch0 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 192.168.30.0/24 |
| 5 | R3 **G0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 192.168.40.0/24 |
| 6 | Switch1 **Fa0/2** → PC-2 **Fa0** | Copper Straight-Through | 192.168.40.0/24 |
| 7 | R2 **Loopback2** → (virtual) | — | 172.16.0.0/16 |
| 8 | R3 **Loopback3** → (virtual) | — | 172.17.0.0/16 |

> Los puertos del switch (Fa0/1, Fa0/2) pueden ser cualquier FastEthernet libre; los números son de ejemplo.

**Equipos requeridos en Packet Tracer:**
- 3 × Router Cisco 2911
- 2 × Switch Cisco 2960-24TT
- 1 × PC (PC-1) y 1 × Laptop (PC-2), como en la imagen de la hoja de ruta
- Cables: **Crossover** entre routers, **Straight-through** de router a switch y de switch a PC

---

## 2. TABLA DE DIRECCIONAMIENTO (X=6, Y=1)

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|--------------|---------|---------|
| R1 | G0/0 | **192.168.10.1** | 255.255.255.0 | N/A |
| R1 | G0/1 | **192.168.20.1** | 255.255.255.0 | N/A |
| R2 | G0/0 | **192.168.10.2** | 255.255.255.0 | N/A |
| R2 | G0/1 | **192.168.30.1** | 255.255.255.0 | N/A |
| R2 | Loopback2 | **172.16.6.1** | 255.255.0.0 | N/A |
| R3 | G0/0 | **192.168.20.2** | 255.255.255.0 | N/A |
| R3 | G0/1 | **192.168.40.1** | 255.255.255.0 | N/A |
| R3 | Loopback3 | **172.17.6.1** | 255.255.0.0 | N/A |
| PC-1 | NIC | **192.168.30.16** | 255.255.255.0 | 192.168.30.1 |
| PC-2 | NIC | **192.168.40.16** | 255.255.255.0 | 192.168.40.1 |

### Cálculo paso a paso

| Fórmula | Sustitución | Resultado |
|---------|-------------|-----------|
| 192.168.10.**Y** | Y = 1 | 192.168.10.**1** |
| 192.168.10.**Y+1** | 1+1 = 2 | 192.168.10.**2** |
| 192.168.20.**Y** | Y = 1 | 192.168.20.**1** |
| 192.168.20.**Y+1** | 1+1 = 2 | 192.168.20.**2** |
| 172.16.**X.Y** | X=6, Y=1 | 172.16.**6.1** |
| 172.17.**X.Y** | X=6, Y=1 | 172.17.**6.1** |
| 192.168.30.**YX** | "1" seguido de "6" (no es multiplicación) | 192.168.30.**16** |
| 192.168.40.**YX** | "1" seguido de "6" | 192.168.40.**16** |

---

## 3. CONCEPTO RIPv1

| Aspecto | Detalle |
|---------|---------|
| Tipo | Vector distancia (IGP) |
| Métrica | Número de saltos (máximo 15) |
| Actualizaciones | Tabla completa cada 30 s, por broadcast 255.255.255.255 |
| Transporte | UDP puerto 520 |
| Classful/Classless | Classful: no envía la máscara en las actualizaciones |
| Rutas en tabla | Marcadas con **R** |
| Distancia administrativa | 120 |

> RIPv1 es **classful**: los comandos `network` usan la red de clase sin máscara. Por eso R2 y R3 anuncian sus loopbacks como las redes clase B completas 172.16.0.0 y 172.17.0.0.

---

## 4. CONFIGURACIÓN COMPLETA RIPv1

### PASO 1 — Construir la topología en Packet Tracer

| Desde | Puerto | Hacia | Puerto |
|-------|--------|-------|--------|
| R1 | G0/0 | R2 | G0/0 |
| R1 | G0/1 | R3 | G0/0 |
| R2 | G0/1 | Switch0 | cualquier FastEthernet |
| R3 | G0/1 | Switch1 | cualquier FastEthernet |
| Switch0 | cualquier FastEthernet | PC-1 | NIC |
| Switch1 | cualquier FastEthernet | PC-2 | NIC |

---

### PASO 2 — Configuración de R1

```
enable
configure terminal

hostname R1
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

router rip
 version 1
 network 192.168.10.0
 network 192.168.20.0

end
write memory
```

---

### PASO 3 — Configuración de R2

```
enable
configure terminal

hostname R2
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface Loopback2
 ip address 172.16.6.1 255.255.0.0

router rip
 version 1
 network 192.168.10.0
 network 192.168.30.0
 network 172.16.0.0

end
write memory
```

---

### PASO 4 — Configuración de R3

```
enable
configure terminal

hostname R3
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.20.2 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 no shutdown

interface Loopback3
 ip address 172.17.6.1 255.255.0.0

router rip
 version 1
 network 192.168.20.0
 network 192.168.40.0
 network 172.17.0.0

end
write memory
```

> Las interfaces loopback siempre están activas; no necesitan `no shutdown`.

---

### PASO 5 — Configuración de PCs

*(En Packet Tracer: click en el PC → Desktop → IP Configuration → Static)*

**PC-1:**
```
IPv4 Address    : 192.168.30.16
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.30.1
```

**PC-2:**
```
IPv4 Address    : 192.168.40.16
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.40.1
```

---

## 5. VALIDACIÓN

### PASO 6 — Conectividad local ANTES de configurar RIP

Configurar primero solo las interfaces (sin `router rip`) y verificar:

```
R1# ping 192.168.10.2    ! R2 G0/0
R1# ping 192.168.20.2    ! R3 G0/0
R2# ping 192.168.10.1    ! R1 G0/0
R3# ping 192.168.20.1    ! R1 G0/1
PC-1> ping 192.168.30.1  ! su gateway (R2 G0/1)
PC-2> ping 192.168.40.1  ! su gateway (R3 G0/1)
```

En este momento **PC-1 no debe llegar a PC-2**. `show ip route` en cada router muestra solo redes C y L (conectadas directamente). Ese es el problema que resuelve el protocolo de enrutamiento.

### PASO 7 — Tabla de enrutamiento con RIP (esperar unos 30 segundos)

**R1** — debe ver con R las redes 30, 40, 172.16 y 172.17:
```
R1# show ip route
R    172.16.0.0/16 [120/1] via 192.168.10.2, GigabitEthernet0/0
R    172.17.0.0/16 [120/1] via 192.168.20.2, GigabitEthernet0/1
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/1
L       192.168.20.1/32 is directly connected, GigabitEthernet0/1
R    192.168.30.0/24 [120/1] via 192.168.10.2, GigabitEthernet0/0
R    192.168.40.0/24 [120/1] via 192.168.20.2, GigabitEthernet0/1
```

**R2** — debe ver con R las redes 20, 40 y 172.17:
```
R2# show ip route
     172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.16.0.0/16 is directly connected, Loopback2
L       172.16.6.1/32 is directly connected, Loopback2
R    172.17.0.0/16 [120/2] via 192.168.10.1, GigabitEthernet0/0
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
R    192.168.20.0/24 [120/1] via 192.168.10.1, GigabitEthernet0/0
C    192.168.30.0/24 is directly connected, GigabitEthernet0/1
R    192.168.40.0/24 [120/2] via 192.168.10.1, GigabitEthernet0/0
```

**R3** — debe ver con R las redes 10, 30 y 172.16:
```
R3# show ip route
R    172.16.0.0/16 [120/2] via 192.168.20.1, GigabitEthernet0/0
     172.17.0.0/16 is variably subnetted, 2 subnets, 2 masks
C       172.17.0.0/16 is directly connected, Loopback3
L       172.17.6.1/32 is directly connected, Loopback3
R    192.168.10.0/24 [120/1] via 192.168.20.1, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0
R    192.168.30.0/24 [120/2] via 192.168.20.1, GigabitEthernet0/0
C    192.168.40.0/24 is directly connected, GigabitEthernet0/1
```

> En R2 y R3 se omitieron las líneas L de las interfaces físicas para abreviar. En `[120/2]`, 120 es la distancia administrativa y 2 la métrica (saltos).

### Verificar la configuración RIP

```
R1# show ip protocols
R1# show running-config | section router rip
```

### PASO 8 — Prueba final PC a PC (objetivo principal)

```
PC-1> ping 192.168.40.16   ! PC-2
PC-2> ping 192.168.30.16   ! PC-1
PC-1> ping 172.17.6.1      ! Loopback de R3
PC-2> ping 172.16.6.1      ! Loopback de R2
```

### PASO 9 — Captura de Wireshark

- Filtro para RIP: `rip` o `udp.port == 520`
- Filtro para el ping: `icmp`
- En el informe: mostrar el Echo Request/Reply entre PC-1 y PC-2 y un paquete RIP (Response, versión 1, destino 255.255.255.255, sin campo de máscara).

---

## 6. ERRORES COMUNES Y SOLUCIONES

### Error 1: Interfaces en estado down/down
**Causa:** falta `no shutdown` o el cable está mal conectado.
```
R1# show ip interface brief
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no shutdown
```

### Error 2: RIP no anuncia redes / no hay rutas R
**Causa:** comando `network` mal escrito o interfaz apagada.
```
R2# show running-config | section router rip
R2(config)# router rip
R2(config-router)# network 192.168.10.0
```

### Error 3: La loopback no aparece en los vecinos
**Causa:** falta `network 172.16.0.0` en R2 o `network 172.17.0.0` en R3.
```
R2(config)# router rip
R2(config-router)# network 172.16.0.0
R3(config)# router rip
R3(config-router)# network 172.17.0.0
```

### Error 4: El PC no hace ping a su gateway
**Causas:** IP mal configurada en el PC, interfaz G0/1 del router apagada o cable equivocado.

### Error 5: Los routers se ven pero los PCs no se alcanzan
**Causa:** gateway mal configurado.
```
PC-1: gateway 192.168.30.1
PC-2: gateway 192.168.40.1
```

### Nota: RIPv1 y la loopback
La Loopback2 tiene IP 172.16.6.1 con máscara /16 y RIPv1 la anuncia como la red de clase B completa 172.16.0.0/16. Esto es correcto y esperado.

### Comandos de depuración
```
R1# debug ip rip
R1# undebug all
```

---

## 7. TABLA DE PINGS — REFERENCIA RÁPIDA

### Conectividad directa (antes y después de RIP)
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | R2 G0/0 | `192.168.10.2` | Siempre |
| R1 | R3 G0/0 | `192.168.20.2` | Siempre |
| R2 | R1 G0/0 | `192.168.10.1` | Siempre |
| R3 | R1 G0/1 | `192.168.20.1` | Siempre |
| PC-1 | Gateway R2 | `192.168.30.1` | Siempre |
| PC-2 | Gateway R3 | `192.168.40.1` | Siempre |

### Conectividad remota (solo con RIP convergido)
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | PC-1 | `192.168.30.16` | Con RIP |
| R1 | PC-2 | `192.168.40.16` | Con RIP |
| R1 | Loopback R2 | `172.16.6.1` | Con RIP |
| R1 | Loopback R3 | `172.17.6.1` | Con RIP |
| R2 | PC-2 | `192.168.40.16` | Con RIP |
| R2 | Loopback R3 | `172.17.6.1` | Con RIP |
| R3 | PC-1 | `192.168.30.16` | Con RIP |
| R3 | Loopback R2 | `172.16.6.1` | Con RIP |
| **PC-1** | **PC-2** | **`192.168.40.16`** | **Objetivo final** |
| **PC-2** | **PC-1** | **`192.168.30.16`** | **Objetivo final** |

---

## 8. CHECKLIST RIPv1

- [ ] Topología construida (3 routers, 2 switches, 2 PCs) y mapa de la topología hecho
- [ ] Hostnames R1, R2, R3 (como en la topología)
- [ ] R1: IPs en G0/0 y G0/1, `no shutdown`, RIPv1
- [ ] R2: IPs en G0/0, G0/1 y Loopback2, `no shutdown`, RIPv1
- [ ] R3: IPs en G0/0, G0/1 y Loopback3, `no shutdown`, RIPv1
- [ ] PC-1: 192.168.30.16/24, gateway 192.168.30.1
- [ ] PC-2: 192.168.40.16/24, gateway 192.168.40.1
- [ ] Pings locales OK y `show ip route` sin RIP solo con redes C/L
- [ ] `show ip route` en R1 muestra 2 redes C + 4 redes R
- [ ] `show ip route` en R2 y en R3 muestra 3 redes C + 3 redes R
- [ ] PC-1 hace ping a PC-2 y PC-2 a PC-1
- [ ] Captura de Wireshark: ICMP exitoso + paquetes RIP (UDP 520)

---

## 9. CONFIGURACIÓN ALTERNATIVA — FastEthernet

> Usa esta sección si los routers del laboratorio tienen interfaces **FastEthernet** (Fa0/0, Fa0/1) en lugar de GigabitEthernet (por ejemplo Cisco 1841 o 2811). Las IPs y la lógica son las mismas; solo cambia el nombre de la interfaz.

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
 ip address 192.168.20.1 255.255.255.0
 no shutdown
router rip
 version 1
 network 192.168.10.0
 network 192.168.20.0
end
write memory
```

### R2 — FastEthernet
```
enable
configure terminal
hostname R2
no ip domain-lookup
interface FastEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown
interface Loopback2
 ip address 172.16.6.1 255.255.0.0
router rip
 version 1
 network 192.168.10.0
 network 192.168.30.0
 network 172.16.0.0
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
 ip address 192.168.20.2 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 no shutdown
interface Loopback3
 ip address 172.17.6.1 255.255.0.0
router rip
 version 1
 network 192.168.20.0
 network 192.168.40.0
 network 172.17.0.0
end
write memory
```

---

## 10. BORRAR TODA LA CONFIGURACIÓN

Para empezar de cero en un router:
```
enable
write erase
reload
```
> Cuando pregunte `Proceed with reload? [confirm]`, presionar **Enter**. Si pregunta si se guarda la configuración, responder **no**.

Para borrar solo el proceso RIP:
```
enable
configure terminal
no router rip
end
write memory
```
