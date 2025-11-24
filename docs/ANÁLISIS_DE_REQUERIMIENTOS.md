# ANÁLISIS DE REQUERIMIENTOS

**Curso:** Administración de Plataformas I  
**Universidad Icesi**  
**Fecha:** Noviembre 2024

---

## PARTE I: REQUERIMIENTOS FUNCIONALES

### 1. REQUERIMIENTOS DE INFRAESTRUCTURA DE RED

### REQ-NET-001

**La infraestructura debe implementar Dual Stack** permitiendo el funcionamiento simultáneo de los protocolos IPv4 e IPv6 en todos los dispositivos de red y servidores.

**Criterio de aceptación**:

- Todos los dispositivos deben tener configuración IPv4 e IPv6
- Conectividad funcional en ambos protocolos

### REQ-NET-002

**CUANDO un cliente se conecta a la red, el sistema debe asignar automáticamente una dirección IPv4** mediante el servidor DHCPv4 configurado.

**Criterio de aceptación**: Cliente recibe configuración IP automática incluyendo dirección, máscara, gateway y DNS.

### REQ-NET-003

**El sistema debe implementar un mecanismo de asignación automática de direcciones IPv6** utilizando DHCPv6 (stateful) ó SLAAC (Stateless Address Auto-configuration) para los clientes de la red.

**Criterio de aceptación**:

- Documento de análisis técnico que justifique la elección entre DHCPv6 o SLAAC basándose en los requerimientos del proyecto y las características de cada mecanismo
- **Si se implementa DHCPv6**:
    - Servidor DHCPv6 configurado y operativo
    - Clientes reciben configuración IPv6 completa (dirección, gateway, DNS) mediante DHCPv6
    - Funcionalidad verificable mediante herramientas de diagnóstico (`dhclient`, `tcpdump`)
- **Si se implementa SLAAC**:
    - Router Advertisement (RA) configurado correctamente
    - Clientes generan direcciones IPv6 automáticamente mediante SLAAC
    - Prefijo IPv6 anunciado correctamente
    - Funcionalidad verificable mediante herramientas de diagnóstico (`radvdump`, `ip -6 addr`)

---

### 2. REQUERIMIENTOS DE SERVICIO DNS

### REQ-DNS-001

**El sistema debe implementar al menos dos servidores DNS privados** funcionando en configuración primario-secundario con sincronización de zonas.

**Criterio de aceptación**:

- Dos servidores DNS operativos
- Transferencia de zona configurada entre primario y secundario

### REQ-DNS-002

**Los servidores DNS deben implementar DNSSEC** para proporcionar autenticación de origen y verificación de integridad de datos.

**Criterio de aceptación**:

- Zonas firmadas con DNSSEC
- Validación de firmas DNSSEC funcional

### REQ-DNS-003

**Los servidores DNS deben implementar TSIG (Transaction Signature)** para asegurar las comunicaciones entre servidores DNS.

**Criterio de aceptación**:

- Configuración de llaves TSIG
- Transferencias de zona autenticadas con TSIG

### REQ-DNS-004

**El sistema debe incluir un servidor DNS autoritativo**, responsable de una zona DNS específica.

**Criterio de aceptación**:

- Servidor DNS autoritativo operativo para la zona asignada
- Respuestas autoritativas verificables

### REQ-DNS-005

**El sistema debe implementar DNS64** para permitir la comunicación entre clientes IPv6-only y servidores IPv4.

**Criterio de aceptación**:

- Servidor DNS64 configurado y operativo
- Traducción funcional de registros AAAA para direcciones IPv4

---

### 3. REQUERIMIENTOS DE SERVICIO DE CORREO

### REQ-MAIL-001

**El sistema debe implementar un servidor de correo electrónico** con capacidad de enviar y recibir correos usando protocolos estándar (`SMTP`, `POP3/IMAP`).

**Criterio de aceptación**:

- Servidor de correo operativo
- Envío y recepción de correos funcional

### REQ-MAIL-002

**El sistema de correo debe tener al menos dos clientes configurados** capaces de enviar y recibir correos electrónicos.

**Criterio de aceptación**:

- Dos clientes de correo configurados
- Comunicación exitosa entre ambos clientes

---

### 4. REQUERIMIENTOS DE SINCRONIZACIÓN DE TIEMPO

### REQ-TIME-001

**Todos los servidores del sistema deben estar sincronizados** contra un servidor NTP, ya sea uno propio del proyecto o los servidores del pool global de NTP.

**Criterio de aceptación**:

- Todos los servidores con NTP configurado
- Diferencia de tiempo entre servidores menor a 1 segundo

### REQ-TIME-002

**CUANDO un servidor inicie o reinicie, el sistema debe sincronizar automáticamente** su reloj con el servidor NTP configurado.

**Criterio de aceptación**: Sincronización automática verificable en logs del sistema.

## PARTE II: REQUERIMIENTOS NO FUNCIONALES

### 5. REQUERIMIENTOS DE RENDIMIENTO

### REQ-PERF-001

**El servidor DHCP debe responder a solicitudes de direcciones IP** en un tiempo máximo de 2 segundos.

**Criterio de aceptación:**

- Proceso DORA completo < 2 segundos (medido con Wireshark)

### REQ-PERF-002

**La sincronización NTP debe mantener un offset** menor a 100 milisegundos.

**Criterio de aceptación:**

- `chronyc tracking` muestra offset < 100ms

---

### 8. REQUERIMIENTOS DE SEGURIDAD

### REQ-SEC-001

**El acceso SSH a dispositivos de red debe estar restringido** a la VLAN de administración.

**Criterio de aceptación:**

- ACL configurada en router
- Pruebas desde otras VLANs fallan

### REQ-SEC-002

**Todos los servidores Linux deben tener firewall activo** permitiendo solo tráfico necesario.

**Criterio de aceptación:**

- `ufw status` muestra activo
- Reglas específicas por servicio configuradas

---

### 9. REQUERIMIENTOS DE CONFIABILIDAD

### REQ-REL-001

**El servidor NTP debe mantener stratum ≤ 5**.

**Criterio de aceptación:**

- `chronyc tracking` muestra stratum ≤ 5

### REQ-REL-002

**CUANDO un servidor se reinicie**, los servicios deben iniciarse automáticamente.

**Criterio de aceptación:**

- Servicios habilitados con `systemctl enable`
- Prueba de reinicio exitosa

---

### 10. REQUERIMIENTOS DE MANTENIBILIDAD

### REQ-MAINT-001

**Los servicios implementados deben exponer su estado operativo** de manera que pueda ser consultado y verificado mediante herramientas estándar de monitoreo y diagnóstico.

**Criterio de aceptación:**

- Estado de servicios consultable con `systemctl status`
- Métricas de red verificables con `ping`, `netstat`, `ss`
- Resolución DNS comprobable con `dig`, `nslookup`
- Sincronización NTP verificable con `chronyc tracking`
- Leases DHCP consultables desde el servidor
- Tráfico de red capturable con `tcpdump` o `Wireshark`

### REQ-MAINT-002

**La infraestructura debe estar completamente documentada**.

**Criterio de aceptación:**

- Diagramas de red actualizados
- Tabla de direccionamiento IP
- `README` con comandos básicos e instrucciones de uso en cada servicio