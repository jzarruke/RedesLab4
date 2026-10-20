# LABORATORIO 4 — OSPFv3
## Sección 4 · Grupo 10 (X = 4, Y = 10)
### Topología 3 — IPv6

---

## 1. TOPOLOGÍA

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

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red IPv6 |
|---|----------|---------------|----------|
| 1 | R0 **GE0/0** → R1 **GE0/0** | Copper Cross-Over | 2001:ABCD:1435:A1::/64 |
| 2 | R0 **GE0/1** → R2 **GE0/0** | Copper Cross-Over | 2002:ABCD:1435:B1::/64 |
| 3 | R1 **GE0/1** → Switch0 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 4 | Switch0 **Fa0/2** → PC-0 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 5 | R2 **GE0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |
| 6 | Switch1 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |

> Los puertos del switch (Fa0/1, Fa0/2) pueden ser cualquier FastEthernet libre; los números son de ejemplo.

---

## 2. TABLA DE DIRECCIONAMIENTO (X=4, Y=10)

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

## 3. CONCEPTO OSPFv3

OSPFv3 es la versión de OSPF diseñada para **IPv6**. Funciona igual que OSPFv2 en cuanto a algoritmo (Dijkstra, áreas, LSAs), pero tiene diferencias importantes en la configuración:

| Aspecto | OSPFv2 (IPv4) | OSPFv3 (IPv6) |
|---------|--------------|--------------|
| Protocolo | IPv4 | IPv6 |
| Comando global obligatorio | No | `ipv6 unicast-routing` |
| Configuración de redes | Global (`network` + wildcard) | Por interfaz (`ipv6 ospf area`) |
| Router-ID | Automático (IP más alta) | **Obligatorio** asignarlo manualmente |
| Vecinos | Multicast IPv4 (224.0.0.5) | Multicast IPv6 (FF02::5) |
| Rutas en tabla | Marcadas con **O** | Marcadas con **O** también |
| Distancia administrativa | 110 | 110 |

> ⚠️ **El Router-ID en OSPFv3 debe asignarse manualmente** porque es un valor de 32 bits (formato IPv4) y el router puede no tener interfaces IPv4 activas.

---

## 4. CONFIGURACIÓN COMPLETA OSPFv3

### PASO 1 — Configuración de R0

```
enable
configure terminal

hostname R0

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 0.0.0.10

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::10/64
 ipv6 ospf 1 area 0
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::10/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

---

### PASO 2 — Configuración de R1

```
enable
configure terminal

hostname R1

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 0.0.0.11

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::11/64
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

### PASO 3 — Configuración de R2

```
enable
configure terminal

hostname R2

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 0.0.0.12

interface GigabitEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::11/64
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

### PASO 4 — Configuración de PCs

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

### PASO 5a — Verificar vecinos OSPFv3 (esperar ~10 segundos)

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

### PASO 5b — Verificar tabla de enrutamiento IPv6

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

### PASO 5c — Verificar proceso OSPFv3

```
R0# show ipv6 ospf
R0# show ipv6 ospf interface
R0# show ipv6 ospf database
R0# show ipv6 route ospf
```

---

### PASO 5d — Prueba de conectividad

```
R0# ping 2001:ABCD:1435:A1::11
R0# ping 2002:ABCD:1435:B1::11
R0# ping 2002:ABCD:1435:C1::1
R0# ping 2002:ABCD:1435:D1::1

ping 2002:ABCD:1435:D1::104
ping 2002:ABCD:1435:C1::104
```

---

## 6. ERRORES COMUNES OSPFv3

### Error 1: `% OSPFv3: Router-ID is not configured`
**Síntoma:** OSPFv3 no inicia porque no tiene Router-ID
**Causa:** No se configuró `router-id` y no hay interfaces IPv4 activas
```
R0(config)# ipv6 router ospf 1
R0(config-rtr)# router-id 0.0.0.10
R0# clear ipv6 ospf process
```

---

### Error 2: Vecinos no aparecen / estado distinto a FULL
**Síntoma:** `show ipv6 ospf neighbor` vacío o con estado INIT/EXSTART
**Causas:**
- `ipv6 unicast-routing` no activado
- `ipv6 ospf 1 area 0` no aplicado en la interfaz
- Router-IDs duplicados entre routers

```
R0# show running-config | include ipv6 unicast-routing
R0# show ipv6 ospf interface brief
R0# show ipv6 ospf | include Router ID
R1# show ipv6 ospf | include Router ID
R2# show ipv6 ospf | include Router ID
```

---

### Error 3: Rutas no aparecen con `O` en tabla
**Síntoma:** `show ipv6 route` solo muestra `C` (connected)
**Causa:** Falta `ipv6 ospf 1 area 0` en alguna interfaz del router vecino

```
R1# show ipv6 ospf interface
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ipv6 ospf 1 area 0
```

---

### Error 4: PC no hace ping al gateway IPv6
**Síntoma:** Ping desde PC-0 a 2002:ABCD:1435:C1::1 falla
**Causas posibles:**
1. IPv6 configurado en campo IPv4 del PC — verificar en IP Configuration
2. Prefijo incorrecto (debe ser /64)
3. La dirección IPv6 del router no coincide con el gateway del PC

```
R1# show ipv6 interface GigabitEthernet0/1
```

---

## 🔍 TABLA DE PINGS — REFERENCIA RÁPIDA (IPv6)

### Conectividad directa (siempre debe funcionar)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | R1 G0/0 | `2001:ABCD:1435:A1::11` | ✅ Siempre |
| R0 | R2 G0/0 | `2002:ABCD:1435:B1::11` | ✅ Siempre |
| R1 | R0 G0/0 | `2001:ABCD:1435:A1::10` | ✅ Siempre |
| R2 | R0 G0/1 | `2002:ABCD:1435:B1::10` | ✅ Siempre |
| PC-0 | R1 gateway | `2002:ABCD:1435:C1::1` | ✅ Siempre |
| PC-1 | R2 gateway | `2002:ABCD:1435:D1::1` | ✅ Siempre |

### Conectividad remota (solo después de OSPF en estado FULL)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | R1 G0/1 | `2002:ABCD:1435:C1::1` | ✅ Con OSPFv3 |
| R0 | R2 G0/1 | `2002:ABCD:1435:D1::1` | ✅ Con OSPFv3 |
| R1 | R2 G0/0 | `2002:ABCD:1435:B1::11` | ✅ Con OSPFv3 |
| R2 | R1 G0/1 | `2002:ABCD:1435:C1::1` | ✅ Con OSPFv3 |
| **PC-0** | **PC-1** | **`2002:ABCD:1435:D1::104`** | **✅ OBJETIVO FINAL** |
| **PC-1** | **PC-0** | **`2002:ABCD:1435:C1::104`** | **✅ OBJETIVO FINAL** |

---

## 7. CHECKLIST OSPFv3

- [ ] Topología construida: 3 routers, 2 switches, 2 PCs
- [ ] `ipv6 unicast-routing` activado en R0, R1 y R2
- [ ] R0: `ipv6 router ospf 1` con `router-id 0.0.0.10`
- [ ] R1: `ipv6 router ospf 1` con `router-id 0.0.0.11`
- [ ] R2: `ipv6 router ospf 1` con `router-id 0.0.0.12`
- [ ] R0: `ipv6 ospf 1 area 0` en G0/0 y G0/1
- [ ] R1: `ipv6 ospf 1 area 0` en G0/0 y G0/1
- [ ] R2: `ipv6 ospf 1 area 0` en G0/0 y G0/1
- [ ] PC-0: IPv6 2002:ABCD:1435:C1::104/64, gateway 2002:ABCD:1435:C1::1
- [ ] PC-1: IPv6 2002:ABCD:1435:D1::104/64, gateway 2002:ABCD:1435:D1::1
- [ ] `show ipv6 ospf neighbor` en R0 → 2 vecinos en estado **FULL**
- [ ] `show ipv6 route` en R0 muestra 2 rutas `C` + 2 rutas `O`
- [ ] PC-0 hace ping exitoso a PC-1
- [ ] Captura Wireshark: ICMPv6 exitosos + paquetes OSPFv3 Hello visibles

---

## 8. CONFIGURACIÓN ALTERNATIVA — FastEthernet

> Usa esta sección si los routers del laboratorio físico tienen interfaces **FastEthernet** (Fa0/0, Fa0/1). Modelos comunes: **Cisco 1841**, **2811**, **2621XM**.
> Las direcciones IPv6, el Router-ID y la lógica OSPFv3 son exactamente los mismos — solo cambia el nombre de la interfaz.

### Conexiones con FastEthernet

| # | Conexión | Tipo de cable | Red IPv6 |
|---|----------|---------------|----------|
| 1 | R0 **Fa0/0** → R1 **Fa0/0** | Copper Cross-Over | 2001:ABCD:1435:A1::/64 |
| 2 | R0 **Fa0/1** → R2 **Fa0/0** | Copper Cross-Over | 2002:ABCD:1435:B1::/64 |
| 3 | R1 **Fa0/1** → Switch0 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 4 | Switch0 **Fa0/2** → PC-0 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 5 | R2 **Fa0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |
| 6 | Switch1 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |

### R0 — FastEthernet

```
enable
configure terminal

hostname R0

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 0.0.0.10

interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::10/64
 ipv6 ospf 1 area 0
 no shutdown

interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::10/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

### R1 — FastEthernet

```
enable
configure terminal

hostname R1

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 0.0.0.11

interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::11/64
 ipv6 ospf 1 area 0
 no shutdown

interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
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

ipv6 unicast-routing

ipv6 router ospf 1
 router-id 0.0.0.12

interface FastEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::11/64
 ipv6 ospf 1 area 0
 no shutdown

interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 ospf 1 area 0
 no shutdown

end
write memory
```

> Los comandos de verificación (`show ipv6 ospf neighbor`, `show ipv6 route`, etc.) funcionan igual. La salida mostrará `FastEthernet0/0` en lugar de `GigabitEthernet0/0`.

---

## 9. BORRAR TODA LA CONFIGURACIÓN

Para empezar de cero en cualquier router (borra todo y reinicia):

```
enable
write erase
reload
```

> Cuando pregunte `Proceed with reload? [confirm]` presiona **Enter**. El router arranca sin ninguna configuración.

Para borrar solo OSPFv3 y las direcciones IPv6 sin resetear el router completo:

```
enable
configure terminal
no ipv6 router ospf 1
interface GigabitEthernet0/0
 no ipv6 address
 no ipv6 ospf 1 area 0
interface GigabitEthernet0/1
 no ipv6 address
 no ipv6 ospf 1 area 0
no ipv6 unicast-routing
end
write memory
```
