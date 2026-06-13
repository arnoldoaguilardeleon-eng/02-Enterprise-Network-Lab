# Configuración Completa — Enterprise Network Lab
## Starbucks México — 4 Sucursales

---

## Índice
1. [Esquema General](#esquema-general)
2. [Tema 1 - Frame Relay + OSPF](#tema-1---frame-relay--ospf)
3. [Tema 2 - VLANs + Trunks](#tema-2---vlans--trunks)
4. [DHCP](#dhcp)
5. [OSPF con VLANs](#ospf-con-vlans)
6. [VPN IPSec Sitio a Sitio](#vpn-ipsec-sitio-a-sitio)
7. [Verificación](#verificación)
8. [Problemas resueltos](#problemas-resueltos)

---

## Esquema General

### VLANs
| VLAN ID | Nombre | Usuarios |
|---|---|---|
| 10 | Ventas | 20 |
| 20 | Invitados | 13 |
| 30 | Gerencia | 13 |
| 40 | Administración | 50 |

### Direccionamiento VLSM
| VLAN | Red | Prefijo | Máscara | Wildcard |
|---|---|---|---|---|
| 10 | 172.16.10.0 | /27 | 255.255.255.224 | 0.0.0.31 |
| 20 | 172.16.20.0 | /28 | 255.255.255.240 | 0.0.0.15 |
| 30 | 172.16.30.0 | /28 | 255.255.255.240 | 0.0.0.15 |
| 40 | 172.16.40.0 | /26 | 255.255.255.192 | 0.0.0.63 |

### Topología WAN
| Enlace | Red | Máscara | IP Lado 1 | IP Lado 2 | Broadcast |
|---|---|---|---|---|---|
| CDMX-GDL | 10.10.0.0 | 255.255.255.252 | 10.10.0.1 | 10.10.0.2 | 10.10.0.3 |
| GDL-CUN | 10.10.0.4 | 255.255.255.252 | 10.10.0.5 | 10.10.0.6 | 10.10.0.7 |
| CUN-MTY | 10.10.0.8 | 255.255.255.252 | 10.10.0.9 | 10.10.0.10 | 10.10.0.11 |
| MTY-CDMX | 10.10.0.12 | 255.255.255.252 | 10.10.0.13 | 10.10.0.14 | 10.10.0.15 |

### Enlaces hacia la nube
| Ciudad | Red WAN | IP Router | IP Cloud | Broadcast |
|---|---|---|---|---|
| CDMX | 10.10.0.16 | 10.10.0.17 | 10.10.0.18 | 10.10.0.19 |
| GDL | 10.10.0.20 | 10.10.0.21 | 10.10.0.22 | 10.10.0.23 |
| MTY | 10.10.0.24 | 10.10.0.25 | 10.10.0.26 | 10.10.0.27 |
| CUN | 10.10.0.28 | 10.10.0.29 | 10.10.0.30 | 10.10.0.31 |

### IPs de PCs por sucursal
| Sucursal | PC | IP | Máscara | Gateway |
|---|---|---|---|---|
| CDMX | PC0 | 172.16.1.2 | 255.255.255.0 | 172.16.1.1 |
| GDL | PC1 | 172.16.2.2 | 255.255.255.0 | 172.16.2.1 |
| MTY | PC2 | 172.16.3.2 | 255.255.255.0 | 172.16.3.1 |
| CUN | PC3 | 172.16.4.2 | 255.255.255.0 | 172.16.4.1 |

---

## Tema 1 - Frame Relay + OSPF

### Configuración básica (todos los routers)

```cisco
enable
configure terminal
hostname NOMBRE_DEL_ROUTER
no ip domain-lookup
```

### Interfaces LAN

```cisco
! CDMX
interface GigabitEthernet0/0
 ip address 172.16.1.1 255.255.255.0
 no shutdown

! GDL
interface GigabitEthernet0/0
 ip address 172.16.2.1 255.255.255.0
 no shutdown

! MTY
interface GigabitEthernet0/0
 ip address 172.16.3.1 255.255.255.0
 no shutdown

! CUN
interface GigabitEthernet0/0
 ip address 172.16.4.1 255.255.255.0
 no shutdown
```

### Frame Relay

```cisco
interface Serial0/0/0
 encapsulation frame-relay
 ip address 10.10.0.17 255.255.255.252
 no shutdown

! Asignar DLCI
frame-relay interface-dlci 200

! Si es multipunto
frame-relay map ip 10.10.0.X 200 broadcast
```

### OSPF por router

```cisco
! CDMX
router ospf 1
 router-id 1.1.1.1
 network 172.16.1.0 0.0.0.255 area 0
 network 10.10.0.0 0.0.0.3 area 0
 network 10.10.0.12 0.0.0.3 area 0
 network 10.10.0.16 0.0.0.3 area 0
exit

! GDL
router ospf 1
 router-id 2.2.2.2
 network 172.16.2.0 0.0.0.255 area 0
 network 10.10.0.0 0.0.0.3 area 0
 network 10.10.0.4 0.0.0.3 area 0
 network 10.10.0.20 0.0.0.3 area 0
exit

! MTY
router ospf 1
 router-id 3.3.3.3
 network 172.16.3.0 0.0.0.255 area 0
 network 10.10.0.8 0.0.0.3 area 0
 network 10.10.0.12 0.0.0.3 area 0
 network 10.10.0.24 0.0.0.3 area 0
exit

! CUN
router ospf 1
 router-id 4.4.4.4
 network 172.16.4.0 0.0.0.255 area 0
 network 10.10.0.4 0.0.0.3 area 0
 network 10.10.0.8 0.0.0.3 area 0
 network 10.10.0.28 0.0.0.3 area 0
exit
```

---

## Tema 2 - VLANs + Trunks

### CoreSwitch

```cisco
enable
configure terminal
hostname coreSwitch

vlan 10
 name Ventas
vlan 20
 name Invitados
vlan 30
 name Gerencia
vlan 40
 name Administracion

! Puerto hacia Router CDMX (trunk)
interface FastEthernet0/1
 switchport mode trunk
 no shutdown

! Puerto hacia Servidor (access VLAN 40)
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 40
 no shutdown

! Puerto hacia Switch Ventas (trunk)
interface FastEthernet0/3
 switchport mode trunk
 no shutdown

! Puerto hacia HomeRouter Invitados (access VLAN 20)
interface FastEthernet0/4
 switchport mode access
 switchport access vlan 20
 no shutdown

! Puerto hacia Switch Administración (trunk)
interface FastEthernet0/5
 switchport mode trunk
 no shutdown

! Puerto hacia HomeRouter Gerencia (access VLAN 30)
interface FastEthernet0/6
 switchport mode access
 switchport access vlan 30
 no shutdown

end
write memory
```

### Switch Ventas

```cisco
enable
configure terminal
hostname Ventas

vlan 10
 name Ventas

interface FastEthernet0/1
 switchport mode trunk
 no shutdown

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 10
 no shutdown

interface FastEthernet0/4
 switchport mode access
 switchport access vlan 10
 no shutdown

end
write memory
```

### Switch Administración

```cisco
enable
configure terminal
hostname Administracion

vlan 40
 name Administracion

interface FastEthernet0/1
 switchport mode trunk
 no shutdown

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 40
 no shutdown

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 40
 no shutdown

interface FastEthernet0/4
 switchport mode access
 switchport access vlan 40
 no shutdown

end
write memory
```

### Router CDMX — Router-on-a-Stick con subinterfaces

```cisco
enable
configure terminal
hostname CDMX

interface GigabitEthernet0/0
 no ip address
 no shutdown

! VLAN 10 - Ventas
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 172.16.10.1 255.255.255.224
 ip helper-address 172.16.40.9
 no shutdown

! VLAN 20 - Invitados
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 172.16.20.1 255.255.255.240
 ip helper-address 172.16.40.9
 no shutdown

! VLAN 30 - Gerencia
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 172.16.30.1 255.255.255.240
 ip helper-address 172.16.40.9
 no shutdown

! VLAN 40 - Administración
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 172.16.40.1 255.255.255.192
 ip helper-address 172.16.40.9
 no shutdown

end
write memory
```

---

## DHCP

### Servidor CDMX

```
IP Address:      172.16.40.9
Subnet Mask:     255.255.255.192
Default Gateway: 172.16.40.1
```

### Pools DHCP

| Pool | Gateway | Start IP | Máscara | Max Users |
|---|---|---|---|---|
| ventas | 172.16.10.1 | 172.16.10.10 | 255.255.255.224 | 20 |
| invitados | 172.16.20.1 | 172.16.20.10 | 255.255.255.240 | 13 |
| gerencia | 172.16.30.1 | 172.16.30.10 | 255.255.255.240 | 13 |
| administracion | 172.16.40.1 | 172.16.40.10 | 255.255.255.192 | 50 |

### HomeRouter Invitados (VLAN 20)

```
Router IP:     172.16.20.1
Subnet Mask:   255.255.255.240
DHCP Start IP: 172.16.20.1
Max Users:     13
```

### HomeRouter Gerencia (VLAN 30)

```
Router IP:     172.16.30.1
Subnet Mask:   255.255.255.240
DHCP Start IP: 172.16.30.1
Max Users:     13
```

---

## OSPF con VLANs

```cisco
! En Router CDMX
router ospf 1
 network 172.16.10.0 0.0.0.31 area 0
 network 172.16.20.0 0.0.0.15 area 0
 network 172.16.30.0 0.0.0.15 area 0
 network 172.16.40.0 0.0.0.63 area 0
 network 10.10.0.0 0.0.0.3 area 0
end
write memory

! En Router GDL
router ospf 1
 router-id 2.2.2.2
 network 172.16.2.0 0.0.0.255 area 0
 network 10.10.0.4 0.0.0.3 area 0
 network 10.10.0.20 0.0.0.3 area 0
```

---

## VPN IPSec Sitio a Sitio

### Paso 1 — Activar licencia (ambos routers)

```cisco
license boot module c1900 technology-package securityk9
end
write memory
reload
```

### Paso 2 — Configurar VPN en CDMX

```cisco
! Tráfico interesante
access-list 110 permit ip 172.16.40.0 0.0.0.63 172.16.2.0 0.0.0.255
access-list 110 permit ip 172.16.30.0 0.0.0.15 172.16.2.0 0.0.0.255
access-list 110 permit ip 172.16.10.0 0.0.0.31 172.16.2.0 0.0.0.255

! Fase 1 - ISAKMP
crypto isakmp policy 10
 encryption aes 256
 authentication pre-share
 group 5
exit

crypto isakmp key vpn123 address 10.10.0.2

! Fase 2
crypto ipsec transform-set VPN-SET esp-aes 256 esp-sha-hmac

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.10.0.2
 set transform-set VPN-SET
 set pfs group5
 match address 110
exit

interface Serial0/1/0
 crypto map VPN-MAP
exit

end
write memory
```

### Paso 3 — Configurar VPN en GDL

```cisco
access-list 110 permit ip 172.16.2.0 0.0.0.255 172.16.40.0 0.0.0.63
access-list 110 permit ip 172.16.2.0 0.0.0.255 172.16.30.0 0.0.0.15
access-list 110 permit ip 172.16.2.0 0.0.0.255 172.16.10.0 0.0.0.31

crypto isakmp policy 10
 encryption aes 256
 authentication pre-share
 group 5
exit

crypto isakmp key vpn123 address 10.10.0.1

crypto ipsec transform-set VPN-SET esp-aes 256 esp-sha-hmac

crypto map VPN-MAP 10 ipsec-isakmp
 set peer 10.10.0.1
 set transform-set VPN-SET
 set pfs group5
 match address 110
exit

interface Serial0/1/0
 crypto map VPN-MAP
exit

end
write memory
```

---

## Verificación

```cisco
! OSPF
show ip ospf neighbor
show ip route
show ip interface brief

! Frame Relay
show frame-relay map
show frame-relay pvc

! VPN
show crypto isakmp sa
show crypto ipsec sa
show crypto session
```

Resultado esperado VPN:
```
dst          src          state     conn-id  slot  status
10.10.0.2    10.10.0.1    QM_IDLE   1043     0     ACTIVE
```

---

## Problemas resueltos

| # | Problema | Solución |
|---|---|---|
| 1 | Puerto Fa0/4 sin configurar causaba DHCP failed en VLAN Ventas | Agregar `switchport access vlan 10` |
| 2 | Subinterfaz G0/0.40 con máscara /24 en lugar de /26 | Corregir a `255.255.255.192` |
| 3 | Pool DHCP con Start IP .3 antes que el servidor .9 | Cambiar Start IP a `.10` |
| 4 | HomeRouter con Start IP diferente a Router IP | Start IP = Router IP |
| 5 | Router 1941 no acepta crypto sin licencia securityk9 | Activar licencia y hacer reload |
| 6 | Crypto map con set peer apuntando a IP propia | Corregir IP del peer |
| 7 | Access-list con red 172.10.2.0 en lugar de 172.16.2.0 | Corregir dirección |
| 8 | PC-DESTINO-VPN con IP en red inexistente | Asignar 172.16.2.2 / gateway 172.16.2.1 |
| 9 | OSPF en CDMX no anunciaba redes de VLANs | Agregar network de cada VLAN con wildcard VLSM |
