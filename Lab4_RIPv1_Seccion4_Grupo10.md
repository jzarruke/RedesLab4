# LABORATORIO 4 — Enrutamiento Dinámico IPv4
## Sección 4 · Grupo 10 (X = 4, Y = 10)
### Guía completa: RIPv1 → OSPFv2 → BGP

---

## 1. INTERPRETACIÓN DE LA TOPOLOGÍA 1

### Diagrama lógico (Topología 1 — RIPv1 y OSPF)

```
                        [R0]
                    G0/0 /  \ G0/1
                        /    \
               192.168.10.0   192.168.20.0
                      /            \
                  [R1]              [R2]
              G0/0  G0/1        G0/0  G0/1
             /         \      /         \
   192.168.10.0   192.168.30.0  192.168.20.0  192.168.40.0
                       |                        |
                   [Switch0]               [Switch1]
                       |                        |
                    [PC-0]                   [PC-1]
               192.168.30.104           192.168.40.104
```

### Descripción de conexiones

| Enlace | Dispositivo A | Interfaz A | Dispositivo B | Interfaz B | Red |
|--------|---------------|------------|---------------|------------|-----|
| R0–R1 | R0 | G0/0 | R1 | G0/0 | 192.168.10.0/24 |
| R0–R2 | R0 | G0/1 | R2 | G0/0 | 192.168.20.0/24 |
| R1–SW0 | R1 | G0/1 | Switch0 | cualquier | 192.168.30.0/24 |
| R2–SW1 | R2 | G0/1 | Switch1 | cualquier | 192.168.40.0/24 |
| Loopback R1 | R1 | Loopback2 | — | — | 172.16.0.0/16 |
| Loopback R2 | R2 | Loopback3 | — | — | 172.17.0.0/16 |

**Equipos requeridos en Packet Tracer:**
- 3 × Router Cisco 2911
- 2 × Switch Cisco 2960-24TT
- 2 × PC (o Laptop)
- Cables: **Crossover** entre routers (o usar Auto-MDI-X), **Straight-through** de router a switch y de switch a PC

---

## 2. TABLA DE DIRECCIONAMIENTO — VALORES FINALES (X=4, Y=10)

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|--------------|---------|---------|
| R0 | G0/0 | **192.168.10.10** | 255.255.255.0 | N/A |
| R0 | G0/1 | **192.168.20.10** | 255.255.255.0 | N/A |
| R1 | G0/0 | **192.168.10.11** | 255.255.255.0 | N/A |
| R1 | G0/1 | **192.168.30.1** | 255.255.255.0 | N/A |
| R1 | Loopback2 | **172.16.4.10** | 255.255.0.0 | N/A |
| R2 | G0/0 | **192.168.20.11** | 255.255.255.0 | N/A |
| R2 | G0/1 | **192.168.40.1** | 255.255.255.0 | N/A |
| R2 | Loopback3 | **172.17.4.10** | 255.255.0.0 | N/A |
| PC-0 | NIC | **192.168.30.104** | 255.255.255.0 | 192.168.30.1 |
| PC-1 | NIC | **192.168.40.104** | 255.255.255.0 | 192.168.40.1 |

### Cálculo paso a paso
| Fórmula | Sustitución | Resultado |
|---------|-------------|-----------|
| 192.168.10.**Y** | Y = 10 | 192.168.10.**10** |
| 192.168.10.**Y+1** | Y+1 = 11 | 192.168.10.**11** |
| 192.168.20.**Y** | Y = 10 | 192.168.20.**10** |
| 192.168.20.**Y+1** | Y+1 = 11 | 192.168.20.**11** |
| 172.16.**X.Y** | X=4, Y=10 | 172.16.**4.10** |
| 172.17.**X.Y** | X=4, Y=10 | 172.17.**4.10** |
| 192.168.30.**YX** | Y=10, X=4 → concat "104" | 192.168.30.**104** |
| 192.168.40.**YX** | Y=10, X=4 → concat "104" | 192.168.40.**104** |

---

## 3. CONFIGURACIÓN COMPLETA — FASE 1: RIPv1

> ⚠️ RIPv1 es **classful**: los comandos `network` usan la dirección de red de clase (sin subnet mask). Esto significa que R2 y R3 anuncian sus loopbacks como redes clase B (172.16.0.0 y 172.17.0.0).

---

### PASO 1 — Construir la topología en Packet Tracer

1. Abrir Packet Tracer → New
2. Agregar: **3 routers Cisco 2911**, **2 switches Cisco 2960-24TT**, **1 PC**, **1 Laptop (como PC-1)**
3. Conexiones (usar **Copper Cross-Over** entre routers, **Copper Straight-Through** para router→switch y switch→PC):

| Desde | Puerto | Hacia | Puerto |
|-------|--------|-------|--------|
| R0 | G0/0 | R1 | G0/0 |
| R0 | G0/1 | R2 | G0/0 |
| R1 | G0/1 | Switch0 | cualquier FastEthernet |
| R2 | G0/1 | Switch1 | cualquier FastEthernet |
| Switch0 | cualquier FastEthernet | PC-0 | NIC |
| Switch1 | cualquier FastEthernet | PC-1 | NIC |

---

### PASO 2 — Configuración de R0

```
! === ROUTER R0 ===
enable
configure terminal

! Nombre del dispositivo
hostname R0

! Interfaz G0/0 → conecta con R1 (red 192.168.10.0/24)
interface GigabitEthernet0/0
 ip address 192.168.10.10 255.255.255.0
 no shutdown

! Interfaz G0/1 → conecta con R2 (red 192.168.20.0/24)
interface GigabitEthernet0/1
 ip address 192.168.20.10 255.255.255.0
 no shutdown

! Configuración RIPv1
router rip
 version 1
 network 192.168.10.0
 network 192.168.20.0

end
write memory
```

---

### PASO 3 — Configuración de R1

```
! === ROUTER R1 ===
enable
configure terminal

! Nombre del dispositivo
hostname R1

! Interfaz G0/0 → conecta con R0 (red 192.168.10.0/24)
interface GigabitEthernet0/0
 ip address 192.168.10.11 255.255.255.0
 no shutdown

! Interfaz G0/1 → conecta con Switch0/PC-0 (red 192.168.30.0/24)
interface GigabitEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown

! Loopback 2 (interfaz virtual siempre activa)
interface Loopback2
 ip address 172.16.4.10 255.255.0.0
 no shutdown

! Configuración RIPv1
! NOTA: 172.16.4.10 pertenece a la clase B → se anuncia como 172.16.0.0
router rip
 version 1
 network 192.168.10.0
 network 192.168.30.0
 network 172.16.0.0

end
write memory
```

---

### PASO 4 — Configuración de R2

```
! === ROUTER R2 ===
enable
configure terminal

! Nombre del dispositivo
hostname R2

! Interfaz G0/0 → conecta con R0 (red 192.168.20.0/24)
interface GigabitEthernet0/0
 ip address 192.168.20.11 255.255.255.0
 no shutdown

! Interfaz G0/1 → conecta con Switch1/PC-1 (red 192.168.40.0/24)
interface GigabitEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 no shutdown

! Loopback 3 (interfaz virtual siempre activa)
interface Loopback3
 ip address 172.17.4.10 255.255.0.0
 no shutdown

! Configuración RIPv1
! NOTA: 172.17.4.10 pertenece a la clase B → se anuncia como 172.17.0.0
router rip
 version 1
 network 192.168.20.0
 network 192.168.40.0
 network 172.17.0.0

end
write memory
```

---

### PASO 5 — Configuración de PCs

#### PC-0
```
IPv4 Address        : 192.168.30.104
Subnet Mask         : 255.255.255.0
Default Gateway     : 192.168.30.1
```
*(En Packet Tracer: click en PC-0 → Desktop → IP Configuration → ingresar valores)*

#### PC-1
```
IPv4 Address        : 192.168.40.104
Subnet Mask         : 255.255.255.0
Default Gateway     : 192.168.40.1
```
*(En Packet Tracer: click en PC-1/Laptop → Desktop → IP Configuration → ingresar valores)*

---

## 4. VALIDACIÓN

### PASO 6a — Verificar conectividad local ANTES de RIP
> Primero confirmar que cada dispositivo llega a su vecino directo (sin enrutamiento dinámico aún activo):

```
! Desde R0:
ping 192.168.10.11    ! → debe llegar a R1 G0/0
ping 192.168.20.11    ! → debe llegar a R2 G0/0

! Desde R1:
ping 192.168.10.10    ! → debe llegar a R0 G0/0
ping 192.168.30.104   ! → debe llegar a PC-0 (si ya tiene IP)

! Desde R2:
ping 192.168.20.10    ! → debe llegar a R0 G0/1
ping 192.168.40.104   ! → debe llegar a PC-1 (si ya tiene IP)

! Desde PC-0 (CMD en Packet Tracer):
ping 192.168.30.1     ! → debe llegar a R1 G0/1 (gateway)

! Desde PC-1 (CMD en Packet Tracer):
ping 192.168.40.1     ! → debe llegar a R2 G0/1 (gateway)
```

### PASO 6b — Verificar tabla de enrutamiento RIP (esperar ~30 segundos)

#### En R0 — debería ver rutas R hacia redes 30, 40, 172.16, 172.17:
```
R0# show ip route
```
**Salida esperada:**
```
Codes: C - connected, S - static, R - RIP ...

     172.16.0.0/16 is variably subnetted
R       172.16.0.0/16 [120/1] via 192.168.10.11, GigabitEthernet0/0
R       172.17.0.0/16 [120/1] via 192.168.20.11, GigabitEthernet0/1
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/1
R    192.168.30.0/24 [120/1] via 192.168.10.11, GigabitEthernet0/0
R    192.168.40.0/24 [120/1] via 192.168.20.11, GigabitEthernet0/1
```

#### En R1 — debería ver rutas R hacia redes 20, 40, 172.17:
```
R1# show ip route
```
**Salida esperada:**
```
R    172.17.0.0/16 [120/2] via 192.168.10.10, GigabitEthernet0/0
C    172.16.0.0/16 is directly connected, Loopback2
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
R    192.168.20.0/24 [120/1] via 192.168.10.10, GigabitEthernet0/0
C    192.168.30.0/24 is directly connected, GigabitEthernet0/1
R    192.168.40.0/24 [120/2] via 192.168.10.10, GigabitEthernet0/0
```

#### En R2 — debería ver rutas R hacia redes 10, 30, 172.16:
```
R2# show ip route
```
**Salida esperada:**
```
R    172.16.0.0/16 [120/2] via 192.168.20.10, GigabitEthernet0/0
C    172.17.0.0/16 is directly connected, Loopback3
R    192.168.10.0/24 [120/1] via 192.168.20.10, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0
R    192.168.30.0/24 [120/2] via 192.168.20.10, GigabitEthernet0/0
C    192.168.40.0/24 is directly connected, GigabitEthernet0/1
```

### PASO 6c — Verificar configuración RIP
```
R0# show running-config
R1# show running-config
R2# show running-config
```
Buscar sección `router rip` con `version 1` y los `network` correspondientes.

### PASO 6d — Prueba final PC a PC (¡el objetivo principal!)
```
! Desde PC-0 (Desktop → Command Prompt):
ping 192.168.40.104    ! → PC-1 (debe funcionar con RIP activo)
ping 172.16.4.10       ! → Loopback R1
ping 172.17.4.10       ! → Loopback R2

! Desde PC-1 (Desktop → Command Prompt):
ping 192.168.30.104    ! → PC-0
```

---

## 5. ERRORES COMUNES Y SOLUCIONES

### Error 1: Interfaces en estado down/down
**Síntoma:** `show ip interface brief` muestra la interfaz como `down/down`
**Causa:** Falta el comando `no shutdown` o cable mal conectado
```
! Solución:
R0# configure terminal
R0(config)# interface GigabitEthernet0/0
R0(config-if)# no shutdown
```
**En Packet Tracer:** Verificar que el cable esté conectado (luz verde en los extremos)

---

### Error 2: RIP no anuncia redes / tabla de rutas vacía
**Síntoma:** `show ip route` no muestra rutas con `R`
**Causa:** Comando `network` mal escrito, o interfaces sin `no shutdown`
```
! Verificar:
R1# show running-config | section router rip
! Debe mostrar:
!  router rip
!   version 1
!   network 192.168.10.0
!   network 192.168.20.0
```
**Solución:** Volver a entrar al proceso RIP y re-agregar las redes
```
R0# configure terminal
R0(config)# router rip
R0(config-router)# network 192.168.10.0
R0(config-router)# network 192.168.20.0
```

---

### Error 3: Loopback no aparece en tabla de rutas de vecinos
**Síntoma:** 172.16.0.0 ó 172.17.0.0 no aparecen en R0 o R2
**Causa:** Se olvidó agregar `network 172.16.0.0` o `network 172.17.0.0`
```
! En R1:
R1(config)# router rip
R1(config-router)# network 172.16.0.0

! En R2:
R2(config)# router rip
R2(config-router)# network 172.17.0.0
```

---

### Error 4: PC no hace ping a su gateway
**Síntoma:** Ping de PC-0 a 192.168.30.1 falla
**Causas posibles:**
1. IP de la PC mal configurada (verificar en Desktop → IP Configuration)
2. Interfaz G0/1 de R1 apagada (`no shutdown`)
3. PC en VLAN equivocada en el switch (raro en PT básico)
```
! Verificar estado de interfaces:
R1# show ip interface brief
! Debe mostrar GigabitEthernet0/1  192.168.30.1  YES manual  up  up
```

---

### Error 5: PC-0 no llega a PC-1 aunque los routers sí se ven entre sí
**Síntoma:** Routers hacen ping entre ellos, pero PCs no se alcanzan
**Causa:** Gateway mal configurado en las PCs
```
PC-0: gateway debe ser 192.168.30.1  (no 192.168.30.104 ni ningún otro)
PC-1: gateway debe ser 192.168.40.1  (no 192.168.40.104 ni ningún otro)
```

---

### Error 6: RIPv1 y las subredes de loopback
**⚠️ Importante:** RIPv1 es classful. La interfaz Loopback2 tiene IP 172.16.4.10 con máscara /16, pero RIPv1 la anuncia como la red de clase B completa: **172.16.0.0/16**. Esto es correcto y esperado. No confundir con un error.

---

### Comandos de debug (usar con cuidado en PT):
```
! Ver actualizaciones RIP en tiempo real:
R1# debug ip rip

! Detener debug:
R1# undebug all
```

---

## 6. OSPFv2 — CONFIGURACIÓN COMPLETA (Topología 1)

> ⚠️ **Prerequisito:** Confirmar que PC-0 hace ping exitoso a PC-1 con RIPv1 antes de continuar. OSPFv2 usa la **misma topología e IPs** — solo cambia el protocolo de enrutamiento.

---

### ¿Qué es OSPFv2 y en qué se diferencia de RIPv1?

| Aspecto | RIPv1 | OSPFv2 |
|---------|-------|--------|
| Tipo | Vector distancia | Link-state |
| Métrica | Conteo de saltos (máx 15) | Costo (basado en ancho de banda) |
| Actualizaciones | Tabla completa cada 30 seg | Solo cuando hay cambios |
| Convergencia | Lenta (~minutos) | Rápida (~segundos) |
| Classful/Classless | Classful (sin máscara) | Classless (usa wildcard mask) |
| Sintaxis `network` | `network [red-clase]` | `network [ip] [wildcard] area [n]` |
| Vecinos | No los establece explícitamente | Forma adyacencias (Hello packets) |
| Router-ID | No tiene | Necesita un Router-ID único |

**Wildcard mask** = inverso de la máscara de subred:
- Máscara /24 (255.255.255.0) → wildcard **0.0.0.255**
- Máscara /16 (255.255.0.0)   → wildcard **0.0.255.255**

---

### PASO 1 — Eliminar RIPv1 en todos los routers

> Hacer esto en los 3 routers antes de configurar OSPF.

```
! En R0:
R0# configure terminal
R0(config)# no router rip
R0(config)# end

! En R1:
R1# configure terminal
R1(config)# no router rip
R1(config)# end

! En R2:
R2# configure terminal
R2(config)# no router rip
R2(config)# end
```

Verificar que las rutas RIP desaparecieron:
```
R0# show ip route
! Ya NO deben aparecer líneas con "R" — solo las "C" (connected)
```

---

### PASO 2 — Configuración de R0 con OSPFv2

```
! === ROUTER R0 — OSPFv2 ===
enable
configure terminal

router ospf 1
 ! R0 no tiene loopback → el Router-ID será la IP más alta activa
 ! (192.168.20.10) — se asigna automáticamente
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0

end
write memory
```

---

### PASO 3 — Configuración de R1 con OSPFv2

```
! === ROUTER R1 — OSPFv2 ===
enable
configure terminal

router ospf 1
 ! R1 tiene Loopback2 (172.16.4.10) → será el Router-ID automáticamente
 ! porque OSPF prefiere la IP de loopback más alta
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.255.255 area 0

end
write memory
```

---

### PASO 4 — Configuración de R2 con OSPFv2

```
! === ROUTER R2 — OSPFv2 ===
enable
configure terminal

router ospf 1
 ! R2 tiene Loopback3 (172.17.4.10) → será el Router-ID automáticamente
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 172.17.0.0 0.0.255.255 area 0

end
write memory
```

---

### PASO 5 — Validación OSPFv2

#### 5a — Verificar vecinos OSPF (esperar ~10 segundos)
```
R0# show ip ospf neighbor
```
**Salida esperada en R0:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
172.16.4.10       1   FULL/DR         00:00:35    192.168.10.11   GigabitEthernet0/0
172.17.4.10       1   FULL/DR         00:00:35    192.168.20.11   GigabitEthernet0/1
```
> El estado debe ser **FULL** — cualquier otro estado (INIT, 2WAY, EXSTART) indica un problema.

```
R1# show ip ospf neighbor
```
**Salida esperada en R1:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.20.10     1   FULL/BDR        00:00:35    192.168.10.10   GigabitEthernet0/0
```

```
R2# show ip ospf neighbor
```
**Salida esperada en R2:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
192.168.20.10     1   FULL/BDR        00:00:35    192.168.20.10   GigabitEthernet0/0
```

---

#### 5b — Verificar tabla de enrutamiento OSPF

```
R0# show ip route
```
**Salida esperada en R0:**
```
Codes: C - connected, O - OSPF ...

     172.16.0.0/16 [110/2] via 192.168.10.11, GigabitEthernet0/0
O    172.17.0.0/16 [110/2] via 192.168.20.11, GigabitEthernet0/1
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/1
O    192.168.30.0/24 [110/2] via 192.168.10.11, GigabitEthernet0/0
O    192.168.40.0/24 [110/2] via 192.168.20.11, GigabitEthernet0/1
```
> Las rutas OSPF aparecen con **O** (no con R). El costo [110/**2**] = distancia administrativa 110, métrica 2.

```
R1# show ip route
```
**Salida esperada en R1:**
```
O    172.17.0.0/16 [110/3] via 192.168.10.10, GigabitEthernet0/0
C    172.16.0.0/16 is directly connected, Loopback2
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
O    192.168.20.0/24 [110/2] via 192.168.10.10, GigabitEthernet0/0
C    192.168.30.0/24 is directly connected, GigabitEthernet0/1
O    192.168.40.0/24 [110/3] via 192.168.10.10, GigabitEthernet0/0
```

```
R2# show ip route
```
**Salida esperada en R2:**
```
O    172.16.0.0/16 [110/3] via 192.168.20.10, GigabitEthernet0/0
C    172.17.0.0/16 is directly connected, Loopback3
O    192.168.10.0/24 [110/2] via 192.168.20.10, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0
O    192.168.30.0/24 [110/3] via 192.168.20.10, GigabitEthernet0/0
C    192.168.40.0/24 is directly connected, GigabitEthernet0/1
```

---

#### 5c — Prueba final PC a PC
```
! Desde PC-0 (Desktop → Command Prompt):
ping 192.168.40.104    ! → PC-1 ✓
ping 172.16.4.10       ! → Loopback R1 ✓
ping 172.17.4.10       ! → Loopback R2 ✓

! Desde PC-1 (Desktop → Command Prompt):
ping 192.168.30.104    ! → PC-0 ✓
```

---

#### 5d — Comandos adicionales de diagnóstico OSPF
```
! Ver detalles de todas las interfaces OSPF:
R0# show ip ospf interface

! Ver la base de datos LSDB (link-state database):
R0# show ip ospf database

! Ver solo rutas aprendidas por OSPF (sin las conectadas):
R0# show ip route ospf

! Ver configuración OSPF activa:
R0# show running-config | section router ospf
```

---

### ERRORES COMUNES OSPFv2

#### Error 1: Vecinos no forman adyacencia (estado INIT o ausente)
**Síntoma:** `show ip ospf neighbor` no muestra vecinos o muestra estado INIT
**Causas:**
- Interfaces apagadas (`no shutdown` faltante)
- IPs mal configuradas (no están en la misma subred)
- Se olvidó incluir la red en el comando `network`
```
! Verificar que las interfaces estén up/up:
R0# show ip interface brief
! Todas las interfaces activas deben mostrar "up   up"
```

---

#### Error 2: Vecino en estado EXSTART o EXCHANGE (nunca llega a FULL)
**Síntoma:** `show ip ospf neighbor` muestra estado EXSTART eternamente
**Causa en Packet Tracer:** Generalmente es un problema de MTU o de Router-ID duplicado
```
! Verificar Router-IDs (deben ser únicos):
R0# show ip ospf | include Router ID
R1# show ip ospf | include Router ID
R2# show ip ospf | include Router ID
```
Si hay IDs duplicados, asignar manualmente:
```
R0(config)# router ospf 1
R0(config-router)# router-id 0.0.0.10
! Luego recargar el proceso OSPF:
R0# clear ip ospf process
! Responder "yes" a la confirmación
```

---

#### Error 3: Red no aparece en tabla de rutas OSPF
**Síntoma:** Una red no aparece con `O` en `show ip route`
**Causa:** El comando `network` en OSPF no cubre esa interfaz
```
! Ejemplo — si 172.16.4.10/16 no se anuncia en R1:
R1(config)# router ospf 1
R1(config-router)# network 172.16.0.0 0.0.255.255 area 0
! La wildcard 0.0.255.255 cubre toda la red 172.16.x.x
```

---

#### Error 4: RIP y OSPF corriendo simultáneamente
**Síntoma:** La tabla de rutas mezcla `R` y `O`, comportamiento impredecible
**Causa:** Se olvidó ejecutar `no router rip` antes de configurar OSPF
```
! Verificar que RIP esté eliminado:
R0# show running-config | section router rip
! NO debe aparecer nada

! Si aparece, eliminarlo:
R0(config)# no router rip
```

---

### CHECKLIST OSPFv2

- [ ] RIPv1 eliminado en R0, R1 y R2 (`no router rip`)
- [ ] R0: `router ospf 1` con `network` para 192.168.10.0 y 192.168.20.0
- [ ] R1: `router ospf 1` con `network` para 192.168.10.0, 192.168.30.0 y 172.16.0.0
- [ ] R2: `router ospf 1` con `network` para 192.168.20.0, 192.168.40.0 y 172.17.0.0
- [ ] `show ip ospf neighbor` en R0 muestra 2 vecinos en estado FULL
- [ ] `show ip route` en R0 muestra 4 rutas con `O`
- [ ] `show ip route` en R1 muestra 3 rutas con `O`
- [ ] `show ip route` en R2 muestra 3 rutas con `O`
- [ ] PC-0 hace ping exitoso a PC-1 (192.168.40.104)
- [ ] PC-1 hace ping exitoso a PC-0 (192.168.30.104)
- [ ] Captura Wireshark del ping exitoso + paquetes OSPF Hello visibles

---

## 7. INTRODUCCIÓN A BGP (PRIORIDAD 2 — Topología 2)

> BGP (Border Gateway Protocol) es el protocolo de enrutamiento **entre sistemas autónomos (AS)**. Es el protocolo del Internet. A diferencia de RIP y OSPF (IGP — dentro de un AS), BGP es un EGP (protocolo de gateway exterior).

### Topología 2 para BGP (con X=4, Y=10):

```
ASN 100              ASN 200              ASN 300
[PC-0]─[Sw2]─[R0]──────[R1]──────[R2]─[Sw1]─[PC-1]
              |    192.168.30.0   192.168.40.0
         192.168.10.0       |
                        [Switch0]
                            |
                          [PC-0]
                       192.168.60.0
```

| Dispositivo | Interfaz | Dirección IP Final |
|-------------|----------|-------------------|
| R0 | G0/0 | 192.168.10.1 |
| R0 | G0/1 | 192.168.30.**5** (X+1 = 4+1) |
| R1 | G0/0 | 192.168.30.**410** ⚠️ |
| R1 | G0/1 | 192.168.40.**410** ⚠️ |
| R1 | G0/2 | 192.168.60.1 |
| R2 | G0/0 | 192.168.40.**5** (X+1 = 4+1) |
| R2 | G0/1 | 192.168.50.1 |
| PC-0 | NIC | 192.168.10.**14** (X+Y = 4+10) |
| PC-0 | NIC | 192.168.60.**14** (X+Y = 4+10) |
| PC-1 | NIC | 192.168.50.**14** (X+Y = 4+10) |

> ⚠️ **NOTA IMPORTANTE:** Para R1 G0/0 y G0/1, la fórmula XY con X=4 y Y=10 genera "410", que excede el rango válido de IPv4 (0-255). **Debes consultar con tu asistente Nathalia** cómo se maneja este caso para el Grupo 10. Una interpretación posible es usar solo los últimos dos dígitos (10 → el cuarto octeto sería 10, y X se ignora o se usa como decenas → 40+10 = 50, etc.). La interpretación más probable para grupos ≥ 10 es que XY = "4" + "10" interpretado como **cuarto octeto = 14** (suma X+Y) o simplemente como **X×10 + Y = 50**. Confirmar con el profesor.

### Estructura base BGP (referencia — no implementar aún):
```
! R0 — pertenece a ASN 100
router bgp 100
 neighbor 192.168.30.5 remote-as 200    ! vecino R1 en ASN 200
 network 192.168.10.0 mask 255.255.255.0

! R1 — pertenece a ASN 200
router bgp 200
 neighbor 192.168.30.5 remote-as 100    ! vecino R0 en ASN 100
 neighbor 192.168.40.5 remote-as 300    ! vecino R2 en ASN 300
 network 192.168.60.0 mask 255.255.255.0

! R2 — pertenece a ASN 300
router bgp 300
 neighbor 192.168.40.5 remote-as 200    ! vecino R1 en ASN 200
 network 192.168.50.0 mask 255.255.255.0
```

---

## RESUMEN RÁPIDO — CHECKLIST RIPv1

- [ ] Topología construida en Packet Tracer (3 routers, 2 switches, 2 PCs)
- [ ] Cables conectados correctamente (crossover R↔R, straight R↔SW↔PC)
- [ ] R0 configurado: hostname, IPs en G0/0 y G0/1, `no shutdown`, RIPv1
- [ ] R1 configurado: hostname, IPs en G0/0, G0/1 y Loopback2, `no shutdown`, RIPv1
- [ ] R2 configurado: hostname, IPs en G0/0, G0/1 y Loopback3, `no shutdown`, RIPv1
- [ ] PC-0: IP 192.168.30.104, máscara /24, gateway 192.168.30.1
- [ ] PC-1: IP 192.168.40.104, máscara /24, gateway 192.168.40.1
- [ ] `show ip route` en R0 muestra 6 redes (2 C + 4 R)
- [ ] `show ip route` en R1 muestra 6 redes (3 C + 3 R)
- [ ] `show ip route` en R2 muestra 6 redes (3 C + 3 R)
- [ ] PC-0 hace ping exitoso a PC-1 (192.168.40.104)
- [ ] PC-1 hace ping exitoso a PC-0 (192.168.30.104)
- [ ] C