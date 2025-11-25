# Documentación del Proyecto Final - Plataformas I

**Institución:** Universidad Icesi
**Curso:** Plataformas I
**Período Académico:** 2025-1

## Integrantes

- Esteban Gaviria Zambrano - A00396019
- Juan Manuel Díaz Moreno - A00394477
- Santiago Valencia García - A00395902

---

## Descripción del Proyecto

Este repositorio contiene la documentación técnica completa de la infraestructura de red implementada como proyecto final del curso. La solución incluye servicios críticos de red desplegados mediante infraestructura como código (IaC), garantizando automatización, reproducibilidad y gestión eficiente de la configuración.

---

## Infraestructura Implementada

La infraestructura está compuesta por seis repositorios independientes, cada uno dedicado a un servicio específico o componente del sistema:

### 1. DNS Primario y Secundario
**Repositorio:** [primary-and-secondary-dns-server](https://github.com/platforms1-icesi/primary-and-secondary-dns-server)

Sistema de resolución de nombres autoritativo con redundancia para el dominio `plataformas.local`. Incluye soporte para dual-stack IPv4/IPv6, transferencias de zona autenticadas con TSIG y funcionalidad DNS64 para síntesis de registros AAAA.

**Herramientas de IaC:**
- Vagrant (virtualización)
- Ansible (gestión de configuración)
- Jinja2 (templating de archivos de zona)

**Artefactos incluidos:**
- Configuraciones de BIND9
- Archivos de zona DNS
- Claves TSIG
- Playbooks de Ansible

---

### 2. Servidor NTP
**Repositorio:** [ntp-server](https://github.com/platforms1-icesi/ntp-server)

Servidor de sincronización horaria basado en Chrony que mantiene la precisión temporal en toda la infraestructura. Sincroniza con pools públicos de NTP y sirve como fuente de tiempo para las VLANs internas.

**Herramientas de IaC:**
- Vagrant (virtualización)
- Shell Scripts (aprovisionamiento y utilidades)

**Artefactos incluidos:**
- Configuración de Chrony
- Scripts de aprovisionamiento
- Scripts de verificación de sincronización
- Scripts de configuración de clientes

---

### 3. Servidor de Correo Electrónico
**Repositorio:** [email-server](https://github.com/platforms1-icesi/email-server)

Infraestructura completa de correo electrónico con Postfix (MTA) y Dovecot (MDA). Implementa seguridad mediante TLS 1.2+, autenticación SASL y medidas anti-spam. Soporta protocolos SMTP, IMAP y POP3 con sus variantes seguras.

**Herramientas de IaC:**
- Vagrant (virtualización)
- Ansible (despliegue automatizado)
- Python (suite de pruebas)

**Artefactos incluidos:**
- Configuraciones de Postfix y Dovecot
- Playbooks de Ansible
- Certificados TLS
- Scripts de monitoreo y pruebas

---

### 4. Servidor DHCPv4
**Repositorio:** [dhcpv4-server](https://github.com/platforms1-icesi/dhcpv4-server)

Servidor de asignación dinámica de direcciones IPv4 basado en Kea DHCP. Proporciona configuración automática de red (direcciones IP, gateway, DNS) para clientes en las VLANs definidas.

**Herramientas de IaC:**
- Vagrant (virtualización)
- Shell Scripts (aprovisionamiento)

**Artefactos incluidos:**
- Configuración de Kea DHCP4
- Scripts de aprovisionamiento
- Utilidades de gestión (logs, leases, captura de tráfico)

---

### 5. Infraestructura de Monitoreo
**Repositorio:** [monitoring-infrastructure](https://github.com/platforms1-icesi/monitoring-infrastructure)

Sistema centralizado de monitoreo y visualización basado en Prometheus y Grafana. Recolecta métricas de todos los servicios mediante exporters especializados y las presenta en dashboards interactivos.

**Herramientas de IaC:**
- Vagrant (virtualización)
- Ansible (despliegue de stack de monitoreo y exporters)

**Artefactos incluidos:**
- Configuración de Prometheus
- Dashboards de Grafana
- Playbooks de Ansible para exporters
- Inventario de servidores

---

### 6. Documentación
**Repositorio:** [documentation](https://github.com/platforms1-icesi/documentation) (este repositorio)

Repositorio centralizado que contiene toda la documentación técnica, análisis de requerimientos, diseño lógico, justificaciones de decisiones arquitectónicas y evidencias de funcionamiento de los servicios implementados.

---

## Contenido de este Repositorio

### Documentos Técnicos

Los documentos se encuentran en la carpeta [`docs/`](docs/) e incluyen:

| Documento | Descripción |
|-----------|-------------|
| [ANÁLISIS_DE_REQUERIMIENTOS.md](docs/ANÁLISIS_DE_REQUERIMIENTOS.md) | Análisis detallado de los requerimientos técnicos y funcionales del proyecto |
| [TABLA_DE_EQUIPOS.md](docs/TABLA_DE_EQUIPOS.md) | Inventario del hardware físico utilizado en la implementación |
| [Asignación_de_Responsabilidades_Proyecto_Plataformas_I.xlsx](docs/Asignación_de_Responsabilidades_Proyecto_Plataformas_I.xlsx) | Matriz de asignación de tareas y seguimiento de completitud del proyecto |
| [SELECCIÓN_DE_MECANISMO_DE_ASIGNACIÓN_IPv6.md](docs/SELECCIÓN_DE_MECANISMO_DE_ASIGNACIÓN_IPv6.md) | Justificación técnica de la estrategia de asignación IPv6 seleccionada |
| [JUSTIFICACIÓN_DE_LA_NO_IMPLEMENTACIÓN_DE_NAT64.md](docs/JUSTIFICACIÓN_DE_LA_NO_IMPLEMENTACIÓN_DE_NAT64.md) | Análisis de la decisión de no implementar NAT64 en la topología |
| [DISEÑO_LOGICO_SMTP_SERVER.md](docs/DISEÑO_LOGICO_SMTP_SERVER.md) | Arquitectura y diseño del servidor de correo electrónico |
| [CONFIGURACIÓN_ROUTER_Y_SWITCH.md](docs/CONFIGURACIÓN_ROUTER_Y_SWITCH.md) | Configuración detallada de los dispositivos de red físicos (Router Cisco ISR 4321 y Switch Dell Catalyst 2960) |
| [PRUEBAS_DNS.md](docs/PRUEBAS_DNS.md) | Comandos de validación y evidencias de funcionamiento del servicio DNS |
| [PRUEBAS_NTP.md](docs/PRUEBAS_NTP.md) | Comandos de validación y evidencias de funcionamiento del servicio NTP |
| [PRUEBAS_DHCPV4.md](docs/PRUEBAS_DHCPV4.md) | Comandos de validación y evidencias de funcionamiento del servicio DHCPv4 |

### Diagramas

Los diagramas de red y topología se encuentran en la carpeta [`docs/diagrams/`](docs/diagrams/) y proporcionan representaciones visuales de la infraestructura implementada.

---

## Arquitectura General

### Segmentación de Red

La infraestructura está segmentada en tres VLANs principales:

- **VLAN 10 (192.168.10.0/24, 2001:db8:10::/64):** Red de administración
- **VLAN 20 (192.168.20.0/24, 2001:db8:20::/64):** Red de servidores
- **VLAN 30 (192.168.30.0/24, 2001:db8:30::/64):** Red de clientes

### Servicios Desplegados

| Servicio | IP | Función |
|----------|-----|---------|
| DNS Primario | 192.168.20.10 | Resolución de nombres autoritativa |
| DNS Secundario | 192.168.20.11 | Redundancia DNS |
| Servidor Email | 192.168.20.20 | Envío y recepción de correo |
| Servidor NTP | 192.168.20.30 | Sincronización horaria |
| Servidor Monitoreo | 192.168.20.40 | Métricas y visualización |
| Servidor DHCP | 192.168.20.50 | Asignación automática de direcciones |

### Tecnologías Principales

- **Virtualización:** VirtualBox, Vagrant
- **Automatización:** Ansible, Shell Scripts
- **Sistemas Operativos:** Ubuntu 22.04 LTS
- **DNS:** BIND9
- **Email:** Postfix, Dovecot
- **NTP:** Chrony
- **DHCP:** Kea DHCP4
- **Monitoreo:** Prometheus, Grafana
- **Routing:** Cisco 4321 (Inter-VLAN, Dual-Stack, NAT)
- **Switching:** Dell Catalyst 2960 (VLANs, Trunk, Access)

---

## Acceso a Repositorios

Todos los repositorios están disponibles públicamente en la organización de GitHub:

**Organización:** [platforms1-icesi](https://github.com/platforms1-icesi)

**Repositorios:**
- https://github.com/platforms1-icesi/primary-and-secondary-dns-server
- https://github.com/platforms1-icesi/ntp-server
- https://github.com/platforms1-icesi/email-server
- https://github.com/platforms1-icesi/dhcpv4-server
- https://github.com/platforms1-icesi/monitoring-infrastructure
- https://github.com/platforms1-icesi/documentation

---

## Despliegue de la Infraestructura

Para desplegar la infraestructura completa, siga el orden recomendado:

1. **Servidor NTP** - Garantiza sincronización temporal desde el inicio
2. **DNS Primario y Secundario** - Habilita resolución de nombres
3. **Servidor DHCP** - Permite configuración automática de clientes
4. **Servidor Email** - Requiere DNS funcional
5. **Infraestructura de Monitoreo** - Requiere todos los servicios activos

Cada repositorio contiene su propio README con instrucciones detalladas de despliegue.

---

## Requisitos del Sistema

### Software Requerido
- VirtualBox 6.0 o superior
- Vagrant 2.2 o superior
- Ansible 2.9 o superior (para repositorios que lo requieran)

### Hardware Recomendado
- CPU: 4 núcleos físicos o superior
- RAM: 16 GB mínimo (para ejecutar todos los servicios simultáneamente)
- Almacenamiento: 50 GB disponibles
- Red: Adaptador de red con soporte para modo promiscuo

---

## Estructura del Repositorio

```
documentation/
├── README.md                           # Este archivo
├── docs/                               # Documentos técnicos
│   ├── ANÁLISIS_DE_REQUERIMIENTOS.md
│   ├── TABLA_DE_EQUIPOS.md
│   ├── SELECCIÓN_DE_MECANISMO_DE_ASIGNACIÓN_IPv6.md
│   ├── JUSTIFICACIÓN_DE_LA_NO_IMPLEMENTACIÓN_DE_NAT64.md
│   ├── DISEÑO_LOGICO_SMTP_SERVER.md
│   ├── CONFIGURACIÓN_ROUTER_Y_SWITCH.md
│   ├── PRUEBAS_DNS.md
│   ├── PRUEBAS_NTP.md
│   ├── PRUEBAS_DHCPV4.md
│   └── diagrams/                       # Diagramas de red
└── imgs/                               # Imágenes y capturas de pantalla
```

---

## Contacto

Para consultas o reportar problemas relacionados con la documentación, por favor contactar a los integrantes del equipo a través de los canales institucionales de la Universidad Icesi.

---

## Licencia

Este proyecto es desarrollado con fines académicos como parte del curso de Plataformas I de la Universidad Icesi.
