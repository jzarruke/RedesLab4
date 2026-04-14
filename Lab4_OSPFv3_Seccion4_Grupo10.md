# LABORATORIO 4 — OSPFv3
## Sección 4 · Grupo 10 (X = 4, Y = 10)
### Topología 3 — IPv6

---

## 1. TOPOLOGÍA

> OSPFv3 usa la **misma topología e IPs que RIPng** — solo cambia el protocolo de enrutamiento.

```
                        [R0]
                    G0/0 /  \ G0/1
                        /    \
          2001:ABCD:1435:A1::/64   2002:ABCD:1435:B1::/64
                      /                  \
                  [R1]                    [R2]
              G0/0  G0/1             G0/0   G0/1
                       |                      |
                   [Switch0]             [Switch1]
                       |                      |
                    [PC-0]                 [PC-1]
          2002:ABCD:1435:C1::/64   2002:ABCD:1435:D1::/64
```

---

## 2. TABLA DE DIRECCIONAMIENTO FINAL (X=4, Y=10)

| Dispositivo | Interfaz | Dirección IPv6 / Prefijo | Gateway |
|-------------|----------|--------------------------|---------|
| R0 | G0/0 | **2001:ABCD:1435:A1::10/64** | N/A |
| R0 | G0/1 | **2002:ABCD:1435:B1::10/64** | N/A |
| R1 | G0/0 | **2001:ABCD:1435:A1::11/64** | N/A |
| R1 | G0/1 | **2002:ABCD:1435:C1::1/64** | N/A |
| R2 | G0/0 | **2002:ABCD:1435:B1::11/64** | N/A |
| R2 | G0/1 | **2002:ABCD:1435:D1::1/64** | N/A |
| PC-0 | NIC | **2002:ABCD:1435:C1::104/64** | 2002:ABCD:1435:C1::1 |
| PC-1 | NIC | **2002:ABCD:1435:D1::104/64** | 2002:ABCD:1435:D1::1 |

---

## 3. DIFERENCIAS CLAVE: OSPFv2 vs OSPFv3

| Aspecto | OSPFv2 (IPv4) | OSPFv3 (IPv6) |
|---------|--------------|--------------|
| Protocolo | IPv4 | IPv6 |
| Configuración | Global (`router ospf`) | Por interfaz (`ipv6 ospf area`) |
| Comando global obligatorio | No | `ipv6 unicast-routing` |
| Sintaxis `network` | `network [ip] [wildcard] area [n]` | No existe — se configura en la interfaz |
| Router-ID | Automático (IP más alta) | **Obligatorio** asignarlo manualmente |
| Vecinos | Descubiertos por Hello (multicast IPv4) | Descubiertos por Hello (multicast IPv6 FF02::5) |
| Rutas en tabla | Marcadas con **O** | Marcadas con **O** también |
| Distancia administrativa | 110 | 110 |

> ⚠️ **Diferencia más importante:** En OSPFv3 el Router-ID debe asignarse manualmente porque es un valor de 32 bits (formato IPv4) y el router puede no tener interfaces IPv4 activas para tomarlo automáticamente.

---

## 4. CONFIGURACIÓN COMPLETA OSPFv3

### PASO 1 — Eliminar RIPng (si venías de RIPng)

```
! En R0:
R0# configure terminal
R0(config)# no ipv6 router rip RIP1
R0(config)# interface GigabitEthernet0/0
R0(config-if)# no ipv6 rip RIP1 enable
R0(config)# interface GigabitEthernet0/1
R0(config-if)# no ipv6 rip RIP1 enable
R0(config)# end

! Repetir en R1 y R2 con sus interfaces correspondientes
```

---

### PASO 2 — Configuración de R0

```
! === ROUTER R0 — OSPFv3 ===
enable
configure terminal

hostname R0

! OBLIGATORIO: habilitar enrutamiento IPv6
ipv6 unicast-routing

! Proceso OSPFv3 con Router-ID manual (formato IPv4, valor único)
ipv6 router ospf 1
 router-id 0.0.0.10

! Interfaz G0/0 → conecta con R1
interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::10/64
 ipv6 ospf 1 area 0
 no shutdown

! Interfaz G0/1 → conecta con R2
interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::10/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

---

### PASO 3 — Configuración de R1

```
! === ROUTER R1 — OSPFv3 ===
enable
configure terminal

hostname R1

! OBLIGATORIO: habilitar enrutamiento IPv6
ipv6 unicast-routing

! Proceso OSPFv3 con Router-ID manual
ipv6 router ospf 1
 router-id 0.0.0.11

! Interfaz G0/0 → conecta con R0
interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::11/64
 ipv6 ospf 1 area 0
 no shutdown

! Interfaz G0/1 → conecta con Switch0 / PC-0
interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

---

### PASO 4 — Configuración de R2

```
! === ROUTER R2 — OSPFv3 ===
enable
configure terminal

hostname R2

! OBLIGATORIO: habilitar enrutamiento IPv6
ipv6 unicast-routing

! Proceso OSPFv3 con Router-ID manual
ipv6 router ospf 1
 router-id 0.0.0.12

! Interfaz G0/0 → conecta con R0
interface GigabitEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::11/64
 ipv6 ospf 1 area 0
 no shutdown

! Interfaz G0/1 → conecta con Switch1 / PC-1
interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

---

### PASO 5 — Configuración de PCs (igual que RIPng)

> En Packet Tracer: PC → **Desktop** → **IP Configuration** → sección **IPv6** → **Static**

**PC-0:**
```
IPv6 Address        : 2002:ABCD:1435:C1::104
Prefix Length       : 64
IPv6 Default Gateway: 2002:ABCD:1435:C1::1
```

**PC-1:**
```
IPv6 Address        : 2002:ABCD:1435:D1::104
Prefix Length       : 64
IPv6 Default Gateway: 2002:ABCD:1435:D1::1
```

---

## 5. VALIDACIÓN

### PASO 6a — Verificar vecinos OSPFv3 (esperar ~10 segundos)

```
R0# show ipv6 ospf neighbor
```
**Salida esperada en R0:**
```
Neighbor ID     Pri   State     Dead Time   Interface ID   Interface
0.0.0.11          1   FULL/DR   00:00:35    1              GigabitEthernet0/0
0.0.0.12          1   FULL/DR   00:00:35    1              GigabitEthernet0/1
```
> El estado debe ser **FULL**. Los Neighbor IDs son los Router-IDs asignados manualmente.

```
R1# show ipv6 ospf neighbor
```
**Salida esperada en R1:**
```
Neighbor ID     Pri   State      Dead Time   Interface ID   Interface
0.0.0.10          1   FULL/BDR   00:00:35    1              GigabitEthernet0/0
```

```
R2# show ipv6 ospf neighbor
```
**Salida esperada en R2:**
```
Neighbor ID     Pri   State      Dead Time   Interface ID   Interface
0.0.0.10          1   FULL/BDR   00:00:35    1              GigabitEthernet0/0
```

---

### PASO 6b — Verificar tabla de enrutamiento IPv6

```
R0# show ipv6 route
```
**Salida esperada en R0:**
```
C   2001:ABCD:1435:A1::/64 [0/0]
     via GigabitEthernet0/0, directly connected
C   2002:ABCD:1435:B1::/64 [0/0]
     via GigabitEthernet0/1, directly connected
O   2002:ABCD:1435:C1::/64 [110/2]
     via FE80::..., GigabitEthernet0/0
O   2002:ABCD:1435:D1::/64 [110/2]
     via FE80::..., GigabitEthernet0/1
```
> Rutas OSPFv3 marcadas con **O**, distancia administrativa **110**.

```
R1# show ipv6 route
R2# show ipv6 route
```

---

### PASO 6c — Verificar proceso OSPFv3

```
! Info general del proceso (Router-ID, área, etc.):
R0# show ipv6 ospf

! Estado de interfaces OSPFv3:
R0# show ipv6 ospf interface

! Base de datos link-state IPv6:
R0# show ipv6 ospf database

! Solo rutas OSPFv3 en tabla:
R0# show ipv6 route ospf
```

---

### PASO 6d — Prueba de conectividad

```
! Desde R0:
R0# ping 2001:ABCD:1435:A1::11    ! → R1 G0/0
R0# ping 2002:ABCD:1435:B1::11    ! → R2 G0/0
R0# ping 2002:ABCD:1435:C1::1     ! → R1 G0/1
R0# ping 2002:ABCD:1435:D1::1     ! → R2 G0/1

! Desde PC-0 (Desktop → Command Prompt):
ping 2002:ABCD:1435:D1::104       ! → PC-1
ping 2002:ABCD:1435:C1::1         ! → gateway R1

! Desde PC-1:
ping 2002:ABCD:1435:C1::104       ! → PC-0
```

---

## 6. ERRORES COMUNES OSPFv3

### Error 1: `% OSPFv3: Router-ID is not configured`
**Síntoma:** OSPFv3 no inicia porque no tiene Router-ID
**Causa:** No se configuró `router-id` en el proceso y no hay interfaces IPv4 activas
```
R0(config)# ipv6 router ospf 1
R0(config-rtr)# router-id 0.0.0.10
! Luego refrescar el proceso:
R0# clear ipv6 ospf process
! Responder "yes"
```

---

### Error 2: Vecinos no aparecen / estado distinto a FULL
**Síntoma:** `show ipv6 ospf neighbor` vacío o con estado INIT/EXSTART
**Causas:**
- `ipv6 unicast-routing` no activado
- `ipv6 ospf 1 area 0` no aplicado en la interfaz
- Router-IDs duplicados entre routers

```
! Verificar que ipv6 unicast-routing esté activo:
R0# show running-config | include ipv6 unicast-routing

! Verificar interfaces con OSPFv3:
R0# show ipv6 ospf interface brief

! Verificar Router-IDs (deben ser distintos):
R0# show ipv6 ospf | include Router ID
R1# show ipv6 ospf | include Router ID
R2# show ipv6 ospf | include Router ID
```

---

### Error 3: Rutas no aparecen con `O` en tabla
**Síntoma:** `show ipv6 route` solo muestra `C` (connected)
**Causa:** Falta `ipv6 ospf 1 area 0` en alguna interfaz del router vecino

```
! Verificar qué interfaces tienen OSPFv3 habilitado:
R1# show ipv6 ospf interface
! Deben aparecer G0/0 y G0/1

! Si falta alguna:
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ipv6 ospf 1 area 0
```

---

### Error 4: RIPng y OSPFv3 corriendo al mismo tiempo
**Síntoma:** `show ipv6 route` mezcla rutas `R` y `O`
**Causa:** No se eliminó RIPng antes de configurar OSPFv3
```
! Verificar:
R0# show running-config | include ipv6 router rip
R0# show ipv6 rip

! Si existe, eliminar:
R0(config)# no ipv6 router rip RIP1
R0(config)# interface GigabitEthernet0/0
R0(config-if)# no ipv6 rip RIP1 enable
R0(config)# interface GigabitEthernet0/1
R0(config-if)# no ipv6 rip RIP1 enable
```

---

### Error 5: PC no hace ping al gateway IPv6
**Síntoma:** Ping desde PC-0 a 2002:ABCD:1435:C1::1 falla
**Causas posibles:**
1. IPv6 configurado en campo IPv4 del PC (verificar en IP Configuration)
2. Prefijo incorrecto (debe ser /64, no /24 ni /32)
3. La dirección IPv6 del router no coincide con el gateway configurado en el PC

```
! Verificar que R1 G0/1 tenga la IP correcta:
R1# show ipv6 interface GigabitEthernet0/1
! Debe mostrar: 2002:ABCD:1435:C1::1
```

---

## 7. COMPARATIVA FINAL: RIPng vs OSPFv3

| Comando | RIPng | OSPFv3 |
|---------|-------|--------|
| Global obligatorio | `ipv6 unicast-routing` | `ipv6 unicast-routing` |
| Proceso global | `ipv6 router rip RIP1` | `ipv6 router ospf 1` + `router-id` |
| Activar en interfaz | `ipv6 rip RIP1 enable` | `ipv6 ospf 1 area 0` |
| Ver vecinos | `show ipv6 rip` | `show ipv6 ospf neighbor` |
| Ver rutas | `show ipv6 route` → `R` | `show ipv6 route` → `O` |
| Ver proceso | `show ipv6 rip database` | `show ipv6 ospf database` |

---

## 8. CHECKLIST OSPFv3

- [ ] RIPng eliminado en R0, R1 y R2 (si venías de RIPng)
- [ ] `ipv6 unicast-routing` activado en R0, R1 y R2
- [ ] R0: `ipv6 router ospf 1` con `router-id 0.0.0.10`
- [ ] R1: `ipv6 router ospf 1` con `router-id 0.0.0.11`
- [ ] R2: `ipv6 router ospf 1` con `router-id 0.0.0.12`
- [ ] R0: `ipv6 ospf 1 area 0` en G0/0 y G0/1
- [ ] R1: `ipv6 ospf 1 area 0` en G0/0 y G0/1
- [ ] R2: `ipv6 ospf 1 area 0` en G0/0 y G0/1
- [ ] `show ipv6 ospf neighbor` en R0 → 2 vecinos en estado **FULL**
- [ ] `show ipv6 route` en R0 muestra 2 rutas `C` + 2 rutas `O`
- [ ] PC-0 hace ping exitoso a PC-1
- [ ] Captura Wireshark: ICMPv6 exitosos + paquetes OSPFv3 Hello visibles
