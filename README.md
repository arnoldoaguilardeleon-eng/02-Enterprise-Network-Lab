# 02 - Enterprise Network Lab

## Descripción
Simulación de red empresarial con 4 sucursales (CDMX, GDL, MTY, CUN) basada en Starbucks México.

## Tecnologías implementadas
- VLANs y Trunks
- OSPF
- WAN con enlaces seriales
- Frame Relay
- DHCP con subinterfaces
- VPN punto a punto
- Conectividad a la nube

## Sucursales
| Sucursal | Red LAN | Red WAN |
|---|---|---|
| Ciudad de México | 172.16.1.0/24 | 10.10.0.17/30 |
| Guadalajara | 172.16.2.0/24 | 10.10.0.21/30 |
| Monterrey | 172.16.3.0/24 | 10.10.0.25/30 |
| Cancún | 172.16.4.0/24 | 10.10.0.29/30 |

## VLANs
| VLAN ID | Departamento | Prefijo | Máscara |
|---|---|---|---|
| 10 | Ventas | /27 | 255.255.255.224 |
| 20 | Invitados | /28 | 255.255.255.240 |
| 30 | Gerencia | /28 | 255.255.255.240 |
| 40 | Administración | /26 | 255.255.255.192 |
