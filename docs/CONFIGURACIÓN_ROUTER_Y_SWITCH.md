# Configuración de Equipos de Red - Router y Switch

## Descripción

Este documento detalla la configuración implementada en los dispositivos de red físicos que conforman la columna vertebral de la infraestructura: el router Cisco ISR 4321 (Main-Router) y el switch Dell Catalyst 2960. Ambos dispositivos gestionan la segmentación de VLANs, el enrutamiento inter-VLAN, la conectividad dual-stack IPv4/IPv6 y la salida a Internet mediante NAT.

---

## 1. Configuración del Router Cisco ISR 4321

**Modelo:** Cisco ISR 4321/K9
**Sistema Operativo:** IOS XE 15.5 (03.16.04b.S)
**Hostname:** Main-Router
**Función:** Gateway principal, enrutamiento inter-VLAN, NAT, salida a Internet

### 1.1 Información General

El router opera como punto central de enrutamiento para todas las VLANs definidas en la topología. Se configuró con licencia `securityk9` para habilitar funcionalidades avanzadas de seguridad y NAT.

```cisco
hostname Main-Router
ip domain name plataformas.local
license boot level securityk9
```

### 1.2 Sincronización Temporal

El router está sincronizado con el servidor NTP interno para mantener coherencia temporal en logs y eventos del sistema.

```cisco
clock timezone COT -5 0
ntp server 192.168.20.30
```

### 1.3 Segmentación de VLANs mediante Subinterfaces

El router implementa enrutamiento inter-VLAN mediante subinterfaces en `GigabitEthernet0/0/0`, que actúa como enlace troncal hacia el switch.

#### VLAN 10 - Administración

```cisco
interface GigabitEthernet0/0/0.10
 description VLAN 10 - Administracion
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
 ipv6 address 2001:DB8:10::1/64
 ipv6 enable
 ipv6 nd ra dns server 2001:DB8:20::10
 ipv6 nd ra dns server 2001:DB8:20::11
```

- **Propósito:** Red de gestión para administradores
- **Gateway IPv4:** 192.168.10.1
- **Gateway IPv6:** 2001:DB8:10::1
- **DNS anunciados vía RA:** 2001:DB8:20::10, 2001:DB8:20::11

#### VLAN 20 - Servidores

```cisco
interface GigabitEthernet0/0/0.20
 description VLAN 20 - Servidores
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
 ipv6 address 2001:DB8:20::1/64
 ipv6 enable
 ipv6 nd ra dns server 2001:DB8:20::10
 ipv6 nd ra dns server 2001:DB8:20::11
```

- **Propósito:** Segmento donde residen los servicios (DNS, NTP, SMTP, DHCP, Monitoreo)
- **Servidores críticos:** 192.168.20.10 (DNS Primary), 192.168.20.30 (NTP), 192.168.20.50 (DHCP)

#### VLAN 30 - Clientes

```cisco
interface GigabitEthernet0/0/0.30
 description VLAN 30 - Clientes
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip helper-address 192.168.20.50
 ip nat inside
 ipv6 address 2001:DB8:30::1/64
 ipv6 enable
 ipv6 nd ra dns server 2001:DB8:20::10
 ipv6 nd ra dns server 2001:DB8:20::11
```

- **Propósito:** Red de usuarios finales
- **DHCP Relay:** Configurado mediante `ip helper-address 192.168.20.50` para reenviar solicitudes DHCP al servidor

### 1.4 Interfaz WAN

```cisco
interface GigabitEthernet0/0/1
 description WAN - Internet
 ip address dhcp
 ip nat outside
 negotiation auto
```

- **Asignación de IP:** Dinámica vía DHCP del ISP
- **Función:** Interfaz externa para salida a Internet

### 1.5 NAT para IPv4 (NAT44)

El router implementa NAT sobrecargado (PAT) para permitir que las tres VLANs internas accedan a Internet compartiendo la única IP pública dinámica.

```cisco
ip nat inside source list 100 interface GigabitEthernet0/0/1 overload

access-list 100 permit ip 192.168.10.0 0.0.0.255 any
access-list 100 permit ip 192.168.20.0 0.0.0.255 any
access-list 100 permit ip 192.168.30.0 0.0.0.255 any
```

- **Lista de acceso 100:** Define las redes internas autorizadas para NAT
- **Overload:** Múltiples hosts internos comparten la misma IP pública mediante multiplexación de puertos

### 1.6 Enrutamiento IPv6

El router habilita enrutamiento nativo para IPv6, permitiendo comunicación directa entre VLANs sin necesidad de NAT.

```cisco
ipv6 unicast-routing
```

Los prefijos IPv6 asignados pertenecen al espacio de documentación `2001:DB8::/32` según RFC 3849.

### 1.7 Configuración de Seguridad SSH

El acceso remoto al router está restringido únicamente a la VLAN de administración mediante SSH v2.

```cisco
username admin privilege 15 secret 5 $1$6NMN$UnFn7EYiXYuffOqWkCwJR1
ip ssh version 2
ip ssh time-out 60

line vty 0 4
 access-class 10 in
 login local
 transport input ssh

access-list 10 permit 192.168.10.0 0.0.0.255
access-list 10 deny   any log
```

- **ACL 10:** Solo permite conexiones SSH desde la VLAN 10 (Administración)
- **Autenticación:** Basada en usuario/contraseña local

### 1.8 Configuración NAT64 (No Funcional)

El router contiene configuración residual de NAT64 stateful que se intentó implementar sin éxito. Esta configuración permanece en el sistema pero **no está operativa** debido a limitaciones técnicas documentadas.

```cisco
interface Loopback64
 description Interfaz Dedicada NAT64
 no ip address
 ipv6 enable
 nat64 enable

nat64 prefix stateful 2001:DB8:64::/96
nat64 v6v4 list NAT64-ACL pool NAT64-WAN overload

ipv6 access-list NAT64-ACL
 permit ipv6 2001:DB8:20::/64 2001:DB8:64::/96
 permit ipv6 2001:DB8:30::/64 2001:DB8:64::/96
 permit ipv6 2001:DB8:10::/64 2001:DB8:64::/96

ip nat pool NAT64-WAN 192.168.130.30 192.168.130.30 prefix-length 24
ipv6 route 2001:DB8:64::/96 Loopback64
```

**Nota Técnica:** La coexistencia de NAT44 y NAT64 sobre la misma interfaz WAN con una única dirección IP pública dinámica resultó inviable en IOS XE 15.5. El mecanismo de traducción no se activa correctamente, generando pérdida total de paquetes en las pruebas realizadas. Para mayor detalle sobre las limitaciones identificadas, consultar el documento [JUSTIFICACIÓN_DE_LA_NO_IMPLEMENTACIÓN_DE_NAT64.md](JUSTIFICACIÓN_DE_LA_NO_IMPLEMENTACIÓN_DE_NAT64.md).

---

## 2. Configuración del Switch Dell Catalyst 2960

**Modelo:** Dell Catalyst 2960
**Sistema Operativo:** IOS 15.2
**Hostname:** switch-1
**Función:** Segmentación de VLANs, distribución de puertos de acceso, enlace troncal

### 2.1 Información General

El switch opera en modo Layer 2, proporcionando conectividad a nivel de enlace de datos para todos los dispositivos finales y servidores.

```cisco
hostname switch-1
ip domain-name plataformas.local
spanning-tree mode pvst
```

### 2.2 VLANs Definidas

Las VLANs se crean implícitamente al asignar puertos a cada segmento. El switch gestiona tres VLANs principales:

- **VLAN 10:** Administración
- **VLAN 20:** Servidores
- **VLAN 30:** Clientes
- **VLAN 1000:** VLAN nativa del trunk (sin uso de datos)
- **VLAN 999:** VLAN de cuarentena para puertos no utilizados

### 2.3 Asignación de Puertos

#### Puertos de Administración (VLAN 10)

```cisco
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access

interface FastEthernet0/2
 switchport access vlan 10
 switchport mode access
```

- **Uso:** Estaciones de trabajo de administradores

#### Puertos de Servidores (VLAN 20)

```cisco
interface FastEthernet0/3
 switchport access vlan 20
 switchport mode access

interface FastEthernet0/4
 switchport access vlan 20
 switchport mode access
```

- **Conectados:** Servidores virtuales con adaptadores bridge a interfaces físicas del host

#### Puertos de Clientes (VLAN 30)

```cisco
interface range FastEthernet0/5-24
 switchport access vlan 30
 switchport mode access
```

- **Rango:** Fa0/5 a Fa0/24 (20 puertos disponibles para usuarios finales)

### 2.4 Puerto Troncal hacia el Router

```cisco
interface GigabitEthernet0/1
 description Trunk to Main-Router
 switchport trunk native vlan 1000
 switchport trunk allowed vlan 10,20,30
 switchport mode trunk
```

- **Protocolo de etiquetado:** IEEE 802.1Q
- **VLANs transportadas:** 10, 20, 30
- **VLAN nativa:** 1000 (tráfico sin etiquetar, no utilizado para evitar ataques de VLAN hopping)

### 2.5 Interfaz de Gestión

El switch posee una interfaz IP en VLAN 10 para gestión remota.

```cisco
interface Vlan10
 ip address 192.168.10.2 255.255.255.0

ip default-gateway 192.168.10.1
```

- **IP de gestión:** 192.168.10.2
- **Gateway:** 192.168.10.1 (Main-Router)

### 2.6 Seguridad

```cisco
interface GigabitEthernet0/2
 switchport access vlan 999
 switchport mode access
 shutdown
```

- **Buena práctica:** Puertos no utilizados se asignan a una VLAN aislada y se deshabilitan administrativamente

---

## 3. Topología Resultante

La configuración conjunta de ambos dispositivos establece la siguiente arquitectura:

```
                    Internet
                       |
                   [WAN DHCP]
                       |
              ┌────────┴────────┐
              │  Main-Router    │
              │  (ISR 4321)     │
              │                 │
              │  Gi0/0/0 Trunk  │
              └────────┬────────┘
                       │ 802.1Q (VLANs 10,20,30)
              ┌────────┴────────┐
              │    Switch-1     │
              │  (Catalyst 2960)│
              └─────────────────┘
                /      |       \
            VLAN10  VLAN20   VLAN30
           (Admin) (Servers) (Clients)
```

### Flujo de Tráfico

1. **Tráfico Intra-VLAN:** Conmutado directamente por el switch (Layer 2)
2. **Tráfico Inter-VLAN:** Enrutado por el router mediante subinterfaces
3. **Tráfico a Internet (IPv4):** Traducido por NAT44 en el router
4. **Tráfico a Internet (IPv6):** Enrutado nativamente sin traducción (Dual-Stack)

---

## 4. Archivos de Configuración

Los archivos completos de configuración se encuentran disponibles en la carpeta `configurations/`:

- [router-config.txt](../configurations/router-config.txt) - Configuración completa del Cisco ISR 4321
- [switch-config.txt](../configurations/switch-config.txt) - Configuración completa del Dell Catalyst 2960

---

## 5. Referencias

- [JUSTIFICACIÓN_DE_LA_NO_IMPLEMENTACIÓN_DE_NAT64.md](JUSTIFICACIÓN_DE_LA_NO_IMPLEMENTACIÓN_DE_NAT64.md) - Análisis técnico de las limitaciones de NAT64
- [TABLA_DE_EQUIPOS.md](TABLA_DE_EQUIPOS.md) - Especificaciones del hardware utilizado
