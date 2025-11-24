# Tabla de equipos de la infraestructura

| Tipo de Equipo            | Fabricante | Modelo            | Función                                                                                                                                       |
|---------------------------|------------|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| Computador de Escritorio  | Dell       | OptiPlex 3050 MT  | Servidor 1: Actúa como host de virtualización secundario dedicado al servicio NTP para mantener la sincronización horaria en toda la topología. |
| Computador de Escritorio  | Dell       | OptiPlex 3050 MT  | Servidor 2: Funciona como host de virtualización principal para alojar los servicios críticos de DNS, SMTP y DHCP.                            |
| Switch Capa 2             | Dell       | Catalyst 2960     | Gestiona el acceso y la segmentación de tráfico mediante VLANs para interconectar los servidores físicos y los clientes de la red.            |
| Router                    | Cisco      | Cisco 4321        | Opera como puerta de enlace encargada del enrutamiento Inter-VLAN, la implementación de Dual Stack y la salida a internet mediante NAT.       |
