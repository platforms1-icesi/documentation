# ANÁLISIS TÉCNICO: SELECCIÓN DE MECANISMO DE ASIGNACIÓN IPv6

**Curso:** Administración de Plataformas I  
**Universidad Icesi**  
**Fecha:** Noviembre 2025

---

## 1. CONTEXTO

El presente documento justifica la decisión de implementar SLAAC (Stateless Address Autoconfiguration) como mecanismo de asignación automática de direcciones IPv6, en lugar de DHCPv6 (stateful), para la infraestructura de red del proyecto final.

La red implementada consta de tres VLANs segmentadas (Administración, Servidores y Clientes) sobre una arquitectura Dual Stack que soporta simultáneamente IPv4 e IPv6. Para IPv4 se implementó un servidor DHCPv4 dedicado, mientras que para IPv6 se requiere determinar el mecanismo más apropiado considerando los objetivos del proyecto y las características de la infraestructura.

## 2. ANÁLISIS COMPARATIVO

### 2.1 SLAAC (Stateless Address Autoconfiguration)

SLAAC es un mecanismo definido en RFC 4862 que permite a los dispositivos IPv6 configurarse automáticamente sin necesidad de un servidor de configuración centralizado. El proceso funciona mediante:

- **Router Advertisement (RA):** El router anuncia el prefijo de red periódicamente mediante mensajes ICMPv6.
- **Generación automática:** Los clientes combinan el prefijo recibido con un identificador de interfaz (típicamente derivado de la dirección MAC) para formar una dirección IPv6 completa.
- **Gateway implícito:** Los clientes obtienen automáticamente la dirección del gateway desde el RA.

**Ventajas:**
- Configuración automática sin infraestructura adicional
- Escalabilidad inherente (sin límite de clientes por pool)
- Sin punto único de fallo (no requiere servidor dedicado)
- Implementación nativa en routers Cisco mediante `ipv6 unicast-routing`
- Menor sobrecarga administrativa
- Renumeración automática de red si cambia el prefijo

**Limitaciones:**
- Control limitado sobre direcciones asignadas
- No proporciona información adicional de configuración (servidores DNS, dominio)
- Direcciones generadas pueden ser predecibles si se usa EUI-64

### 2.2 DHCPv6 (Stateful)

DHCPv6 es un protocolo definido en RFC 3315 que proporciona asignación centralizada de direcciones IPv6 y parámetros de configuración adicionales. Funciona mediante:

- **Servidor DHCPv6:** Componente dedicado que mantiene un registro (lease database) de asignaciones.
- **Proceso de solicitud-respuesta:** Similar a DHCPv4, con mensajes Solicit, Advertise, Request, Reply.
- **Control centralizado:** Permite gestión granular de direcciones asignadas.

**Ventajas:**
- Control preciso sobre direcciones asignadas
- Capacidad de proporcionar opciones adicionales (DNS, dominio, NTP)
- Registro detallado de asignaciones (logs y leases)
- Útil para auditoría y troubleshooting
- Permite políticas de asignación específicas

**Limitaciones:**
- Requiere servidor adicional (infraestructura y mantenimiento)
- Punto único de fallo si no hay redundancia
- Mayor complejidad de implementación
- Requiere relay agent en VLANs diferentes
- Overhead administrativo adicional

### 2.3 Modo Híbrido (SLAAC + DHCPv6 Stateless)

Existe una tercera opción donde SLAAC proporciona la dirección IPv6 y DHCPv6 en modo stateless proporciona únicamente información adicional (DNS, dominio). Esta aproximación combina ventajas de ambos mecanismos pero añade complejidad.

## 3. JUSTIFICACIÓN DE LA DECISIÓN

Para el presente proyecto se ha seleccionado **SLAAC** como mecanismo de asignación IPv6 por las siguientes razones técnicas y operativas:

### 3.1 Simplicidad de Implementación

SLAAC está habilitado de manera nativa en el router Cisco ISR4321/K9 mediante el comando `ipv6 unicast-routing`. No requiere la instalación, configuración ni mantenimiento de un servidor DHCPv6 adicional, lo cual reduce la complejidad de la infraestructura y el tiempo de implementación en el contexto académico del proyecto.

### 3.2 Suficiencia Funcional

El proyecto no requiere las capacidades avanzadas de DHCPv6:
- **Direccionamiento:** SLAAC proporciona direcciones IPv6 funcionales automáticamente.
- **Gateway:** El gateway se obtiene directamente del Router Advertisement.
- **DNS:** Los servidores DNS (192.168.20.10 y 192.168.20.11) se proporcionan mediante DHCPv4 para las conexiones IPv4, y los clientes pueden usar estos mismos servidores para consultas AAAA.
- **Control de direcciones:** No se requiere auditoría estricta de direcciones IPv6 asignadas dado el alcance académico del proyecto.

### 3.3 Consistencia con IPv6

SLAAC representa el modelo de operación nativo de IPv6, diseñado para autoconfigración y plug-and-play. Su implementación demuestra comprensión de los principios fundamentales de IPv6, que es uno de los objetivos pedagógicos del curso.

### 3.4 Menor Superficie de Fallo

Al no requerir un servidor DHCPv6 adicional, se elimina:
- Un punto potencial de fallo en la infraestructura
- La necesidad de configurar relay agents DHCPv6 en el router
- La complejidad de troubleshooting de un servicio adicional
- Requisitos de sincronización y respaldo de bases de datos de leases

### 3.5 Alcance IPv6 del Proyecto

La infraestructura implementa IPv6 únicamente para comunicación interna entre VLANs, sin salida IPv6 a Internet. El tráfico IPv6 no requiere configuraciones especiales de DNS64/NAT64 para la mayoría de servicios internos. Esta limitación de alcance hace innecesaria la complejidad adicional de DHCPv6.

### 3.6 Escalabilidad

SLAAC escala automáticamente sin límites de pool. A diferencia de DHCPv4, donde se definió un pool de 101 direcciones (192.168.30.100-200), SLAAC permite que cualquier cantidad de dispositivos se conecten en el segmento /64 asignado (aproximadamente 18 trillones de direcciones disponibles).

## 4. IMPLEMENTACIÓN

La implementación de SLAAC se realizó mediante la configuración de IPv6 en las subinterfaces del router:
```cisco
ipv6 unicast-routing

interface GigabitEthernet0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ipv6 address 2001:DB8:10::1/64
 ipv6 enable

interface GigabitEthernet0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ipv6 address 2001:DB8:20::1/64
 ipv6 enable

interface GigabitEthernet0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ipv6 address 2001:DB8:30::1/64
 ipv6 enable
```

El comando `ipv6 unicast-routing` activa automáticamente el envío de Router Advertisements (RA) en todas las interfaces IPv6 configuradas. Los clientes conectados a cualquier VLAN:

1. Reciben el RA con el prefijo de red (2001:DB8:XX::/64)
2. Generan automáticamente su dirección IPv6
3. Configuran el gateway como la dirección link-local del router
4. Quedan operativos para comunicación IPv6 sin intervención manual

## 5. VALIDACIÓN

La funcionalidad de SLAAC puede verificarse mediante:

**En cliente Linux:**
```bash
ip -6 addr show
# Debe mostrar dirección IPv6 con prefijo 2001:DB8:XX::/64

ip -6 route show
# Debe mostrar ruta default vía fe80::... (link-local del router)
```

**En cliente Windows:**
```cmd
ipconfig /all
# Debe mostrar dirección IPv6 y gateway
```

**Pruebas de conectividad:**
```bash
ping6 2001:DB8:20::1  # Ping al gateway de otra VLAN
ping6 2001:DB8:20::10 # Ping a servidor DNS primario
```

## 6. CONCLUSIÓN

La selección de SLAAC como mecanismo de asignación IPv6 se fundamenta en:

1. **Adecuación técnica:** Cumple los requerimientos funcionales del proyecto sin añadir complejidad innecesaria.
2. **Simplicidad operativa:** Minimiza la infraestructura requerida y reduce puntos de fallo.
3. **Alineación pedagógica:** Demuestra comprensión de los principios nativos de IPv6.
4. **Eficiencia de recursos:** Optimiza el tiempo de implementación en el contexto académico.

DHCPv6 sería justificable en escenarios que requieran control centralizado estricto de direcciones, auditoría detallada, o provisión de opciones de configuración complejas que no son necesarias en el alcance de este proyecto.

La decisión adoptada satisface plenamente el requerimiento REQ-NET-003 del proyecto, proporcionando asignación automática funcional de direcciones IPv6 con una arquitectura simple, robusta y escalable.

---

**Referencias:**
- RFC 4862: IPv6 Stateless Address Autoconfiguration
- RFC 3315: Dynamic Host Configuration Protocol for IPv6 (DHCPv6)
- RFC 4861: Neighbor Discovery for IP version 6 (IPv6)
- Cisco IOS IPv6 Configuration Guide