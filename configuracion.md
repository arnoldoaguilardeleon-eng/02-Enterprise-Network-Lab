# Configuración Completa — Enterprise Network Lab

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

---

## Comandos CoreSwitch

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

interface FastEthernet0/1
 switchport mode trunk
 no shutdown

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 40
 no shutdown

interface FastEthernet0/3
 switchport mode trunk
 no shutdown

interface FastEthernet0/4
 switchport mode access
 switchport access vlan 20
 no shutdown

interface FastEthernet0/5
 switchport mode trunk
 no shutdown

interface FastEthernet0/6
 switchport mode access
 switchport access vlan 30
 no shutdown

end
write memory
```

---

## Comandos Router CDMX (Router-on-a-Stick)

```cisco
enable
configure terminal
hostname CDMX

interface GigabitEthernet0/0
 no ip address
 no shutdown

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 172.16.10.1 255.255.255.224
 ip helper-address 172.16.40.9
 no shutdown

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 172.16.20.1 255.255.255.240
 ip helper-address 172.16.40.9
 no shutdown

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 172.16.30.1 255.255.255.240
 ip helper-address 172.16.40.9
 no shutdown

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 172.16.40.1 255.255.255.192
 ip helper-address 172.16.40.9
 no shutdown

end
write memory
```

---

## OSPF

```cisco
router ospf 1
 network 172.16.10.0 0.0.0.31 area 0
 network 172.16.20.0 0.0.0.15 area 0
 network 172.16.30.0 0.0.0.15 area 0
 network 172.16.40.0 0.0.0.63 area 0
 network 10.10.0.0 0.0.0.3 area 0
end
write memory
```

---

## VPN IPSec Sitio a Sitio (CDMX ↔ GDL)

```cisco
access-list 110 permit ip 172.16.40.0 0.0.0.63 172.16.2.0 0.0.0.255
access-list 110 permit ip 172.16.30.0 0.0.0.15 172.16.2.0 0.0.0.255
access-list 110 permit ip 172.16.10.0 0.0.0.31 172.16.2.0 0.0.0.255

crypto isakmp policy 10
 encryption aes 256
 authentication pre-share
 group 5
exit

crypto isakmp key vpn123 address 10.10.0.2

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

---

## Problemas resueltos durante el proyecto

1. Puerto Fa0/4 sin configurar causaba DHCP failed en VLAN Ventas
2. Subinterfaz G0/0.40 con máscara /24 en lugar de /26
3. Pool DHCP con Start IP antes que la IP del servidor
4. HomeRouter con Start IP diferente a Router IP
5. Router 1941 requiere licencia securityk9 para VPN
6. Crypto map apuntando a IP propia en lugar del peer
7. Access-list con red incorrecta (172.10.2.0 vs 172.16.2.0)
8. PC-DESTINO-VPN con IP en red inexistente
9. OSPF en CDMX no anunciaba redes de VLANs
