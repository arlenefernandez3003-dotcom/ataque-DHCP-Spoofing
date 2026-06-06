# 🔴 Ataque 3: DHCP Spoofing — Laboratorio de Ciberseguridad

> **Institución:** ITLA – Instituto Tecnológico de las Américas  
> **Carrera:** Seguridad de la Información  
> **Matrícula:** 20250730  
> **Entorno:** Laboratorio académico controlado y autorizado  

---


---

## 📋 Objetivo

Demostrar el funcionamiento del ataque **DHCP Spoofing**, donde un servidor DHCP rogue responde a solicitudes de clientes antes que el servidor legítimo, asignando parámetros de red maliciosos (gateway controlado por el atacante) para ejecutar un ataque **Man-in-the-Middle (MitM)**.

---

## 🗺️ Topología

```
┌─────────────────────────────────────────────────────────┐
│                  RED: 202.25.73.0/24                    │
│                                                         │
│  [ROUTER/GW]          [SWITCH Cisco]                    │
│  202.25.73.1    ──────    (SW1)      ──── [DHCP-SRV]   │
│  (Gateway)       Gi0/0   /    \       Gi0/1 202.25.73.10│
│                         /      \                        │
│                    Gi0/2       Gi0/3                    │
│                 [ATACANTE]   [VÍCTIMA]                  │
│                202.25.73.50  DHCP Client                │
│                Kali Linux    Ubuntu/Win                 │
└─────────────────────────────────────────────────────────┘
```

### Tabla de Direccionamiento

| Dispositivo | IP | Rol |
|---|---|---|
| Router/GW | 202.25.73.1 | Puerta de enlace real |
| DHCP-SRV | 202.25.73.10 | Servidor DHCP legítimo |
| ATACANTE (Kali) | 202.25.73.50 | Servidor DHCP rogue / Gateway MitM |
| VÍCTIMA | 202.25.73.201–220 | Cliente DHCP |
| Pool rogue | 202.25.73.201–220 | IPs asignadas por el atacante |

---

## 🔧 Requisitos

- **OS Atacante:** Kali Linux 2024.1 o superior
- **OS Víctima:** Ubuntu Desktop 22.04 / Windows 10
- **Python:** 3.8 o superior
- **Librería:** Scapy 2.5.0+
- **Privilegios:** root / sudo
- **Herramientas adicionales:** Wireshark, tcpdump

---

## 📦 Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/TU_USUARIO/ataque3-dhcp-spoofing.git
cd ataque3-dhcp-spoofing

# 2. Instalar dependencias
sudo pip3 install scapy

# 3. Habilitar IP forwarding (para actuar como gateway MitM)
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# 4. Configurar IP estática en el atacante
sudo ip addr add 202.25.73.50/24 dev eth0
sudo ip link set eth0 up
```

---

## 🚀 Uso

```bash
# Ejecutar el servidor DHCP rogue
sudo python3 dhcp_spoof.py
```

### En paralelo – Captura de tráfico (Wireshark/tcpdump)

```bash
sudo tcpdump -i eth0 -n port 67 or port 68 -w captura_dhcp.pcap
```

### En la víctima – Renovar IP para activar el ataque

```bash
# Linux
sudo dhclient -r && sudo dhclient eth0

# Windows
ipconfig /release && ipconfig /renew
```

### Verificar en la víctima que el gateway es el atacante

```bash
ip route show     # Linux: gateway debe ser 202.25.73.50
route print       # Windows
```

---

## 🖥️ Salida Esperada del Script

```
╔══════════════════════════════════════════════════════════════════╗
║          DHCP SPOOFING – Servidor DHCP Rogue (Educativo)        ║
║          ITLA – Seguridad de la Información | Mat: 20250730     ║
╠══════════════════════════════════════════════════════════════════╣
║  Red        : 202.25.73.0/24                                    ║
║  Atacante   : 202.25.73.50 (Gateway rogue)                      ║
║  Pool rogue : 202.25.73.201 – 202.25.73.220                     ║
╚══════════════════════════════════════════════════════════════════╝

2025-07-30 22:15:01  INFO      IP Forwarding habilitado.
2025-07-30 22:15:01  INFO      Iniciando servidor DHCP rogue en interfaz 'eth0'...
2025-07-30 22:15:01  INFO      Esperando solicitudes DHCP DISCOVER...

2025-07-30 22:15:12  INFO      [DISCOVER → OFFER]  Cliente MAC=aa:bb:cc:dd:ee:ff  Ofreciendo IP=202.25.73.201
2025-07-30 22:15:12  INFO      [REQUEST  → ACK  ]  Cliente MAC=aa:bb:cc:dd:ee:ff  Confirmando IP=202.25.73.201  GW=202.25.73.50
```

---

## 📸 Evidencias Requeridas

| # | Evidencia | Descripción |
|---|---|---|
| 1 | `captura_dhcp.pcap` | Captura Wireshark del intercambio DHCP |
| 2 | Screenshot gateway víctima | `ip route show` mostrando GW = 202.25.73.50 |
| 3 | Screenshot consola script | Clientes comprometidos en tiempo real |
| 4 | Screenshot Wireshark | DISCOVER / OFFER / REQUEST / ACK |
| 5 | Video demostrativo | Menor de 5 minutos con voz |

---

## 🛡️ Contramedidas

### DHCP Snooping en Switch Cisco

```
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 1
SW1(config)# no ip dhcp snooping information option

! Puerto del servidor DHCP legítimo → TRUSTED
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# ip dhcp snooping trust
SW1(config-if)# exit
SW1(config)# end
SW1# write memory

! Verificar
SW1# show ip dhcp snooping
SW1# show ip dhcp snooping binding
```

**Efecto:** El switch descarta automáticamente cualquier DHCP OFFER proveniente de puertos no confiables (el atacante). Solo el servidor legítimo en el puerto trusted puede enviar respuestas DHCP.

### Contramedidas Adicionales

- **Dynamic ARP Inspection (DAI):** Valida respuestas ARP contra la tabla DHCP Snooping.
- **IP Source Guard:** Previene uso de IPs no asignadas legítimamente.
- **802.1X:** Autenticación de dispositivos antes de permitir acceso a la red.
- **Monitoreo IDS/IPS:** Snort/Suricata con reglas para detectar servidores DHCP rogue.

---

## 📁 Estructura del Repositorio

```
ataque3-dhcp-spoofing/
├── dhcp_spoof.py                          # Script servidor DHCP rogue
├── README.md                              # Este archivo
├── Ataque3_DHCP_Spoofing_Documentacion.docx  # Documentación técnica completa
├── dhcp_spoof.log                         # Log generado al ejecutar (auto)
├── capturas/
│   ├── captura_dhcp.pcap                  # Captura Wireshark
│   ├── 01_gateway_victima.png             # Gateway víctima = atacante
│   ├── 02_wireshark_dhcp.png              # Intercambio DHCP
│   ├── 03_consola_script.png              # Consola del atacante
│   └── 04_contramedida_snooping.png       # DHCP Snooping aplicado
└── video/
    └── demo_dhcp_spoofing.mp4             # Video demostrativo < 5 min
```

---

## 👤 Autor

| Campo | Detalle |
|---|---|
| **Nombre** | Arlene Fernandez|
| **Matrícula** | 20250730 |
| **Institución** | ITLA – Instituto Tecnológico de las Américas |
| **Carrera** | Seguridad de la Información |
| **Fecha** | 2025 |

---

*Repositorio creado con fines académicos y educativos. ITLA © 2025.*
