# Taller - Raspberry Pi: Dashboards y Auditoría de Red

## Descripción del Proyecto
Implementación completa en Raspberry Pi 3 que incluye configuración del sistema, desarrollo de dashboards con Streamlit y Grafana, automatización con Cron, y análisis de red con Nmap.

## Estructura del Repositorio

taller/
│
├── docs/
│ ├── instalacion_raspberry.md
│ ├── configuracion_red.md
│ └── overleaf_document.md
│
├── scripts/
│ ├── dashboard_streamlit.py
│ ├── cron_automatizacion.sh
│ └── analisis_red.sh
│
├── config/
│ ├── crontab_example
│ └── grafana_dashboard.json
│
├── evidences/
│ ├── instalacion/
│ ├── dashboards/
│ └── redes/
│
└── README.md

## 1. Configuración e Instalación de Raspberry Pi

### Flasheo del Sistema Operativo

```bash
# Descargar Raspberry Pi Imager desde oficial website
# Seleccionar Raspberry Pi OS (64-bit) Lite
# Configurar previamente:
# - Hostname: raspberry-taller
# - Habilitar SSH
# - Configurar usuario y contraseña
# - Configurar Wi-Fi credentials
```
### Configuración Inicial del Sistema
``` bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Verificar conexión de red
hostname -I
ping -c 4 8.8.8.8

# Instalar herramientas esenciales
sudo apt install -y python3-pip python3-venv git vim
```
### Solución de Problemas de Conectividad
```bash
# Verificar interfaces de red
ip a

# Configurar Wi-Fi manualmente si es necesario
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

# Agregar configuración de red
network={
    ssid="NOMBRE_RED"
    psk="CONTRASEÑA"
    key_mgmt=WPA-PSK
}

# Reiniciar servicio de red
sudo wpa_cli -i wlan0 reconfigure
sudo systemctl restart networking

# Verificar conectividad
ping -c 4 google.com
```
## 2. Desarrollo de Dashboards
Dashboard con Streamlit
```python
# scripts/dashboard_streamlit.py
import streamlit as st
import pandas as pd
import numpy as np
import psutil
import datetime
import socket

# Configuración de la página
st.set_page_config(
    page_title="Dashboard Raspberry Pi",
    page_icon="🍓",
    layout="wide"
)

# Título principal
st.title("🍓 Dashboard de Monitoreo - Raspberry Pi 3")

# Obtener información del sistema
def get_system_info():
    # Información de CPU
    cpu_percent = psutil.cpu_percent(interval=1)
    cpu_freq = psutil.cpu_freq()
    
    # Información de memoria
    memory = psutil.virtual_memory()
    
    # Información de disco
    disk = psutil.disk_usage('/')
    
    # Información de red
    hostname = socket.gethostname()
    ip_address = socket.gethostbyname(hostname)
    
    return {
        'cpu_percent': cpu_percent,
        'cpu_freq': cpu_freq.current if cpu_freq else 0,
        'memory_percent': memory.percent,
        'memory_used_gb': memory.used / (1024**3),
        'memory_total_gb': memory.total / (1024**3),
        'disk_percent': disk.percent,
        'disk_used_gb': disk.used / (1024**3),
        'disk_total_gb': disk.total / (1024**3),
        'hostname': hostname,
        'ip_address': ip_address,
        'timestamp': datetime.datetime.now()
    }

# Crear columnas para métricas
col1, col2, col3, col4 = st.columns(4)

system_info = get_system_info()

with col1:
    st.metric(
        label="Uso de CPU",
        value=f"{system_info['cpu_percent']}%",
        delta=f"{system_info['cpu_freq']} MHz"
    )

with col2:
    st.metric(
        label="Uso de Memoria",
        value=f"{system_info['memory_percent']}%",
        delta=f"{system_info['memory_used_gb']:.1f} GB"
    )

with col3:
    st.metric(
        label="Uso de Disco",
        value=f"{system_info['disk_percent']}%",
        delta=f"{system_info['disk_used_gb']:.1f} GB"
    )

with col4:
    st.metric(
        label="IP Address",
        value=system_info['ip_address']
    )

# Gráficos de rendimiento
st.subheader("Métricas en Tiempo Real")

# Simular datos históricos para demostración
chart_data = pd.DataFrame({
    'CPU': np.random.randn(20).cumsum() + 50,
    'Memoria': np.random.randn(20).cumsum() + 60,
    'Disco': np.random.randn(20).cumsum() + 70
}, index=pd.date_range('2024-01-01', periods=20, freq='H'))

st.line_chart(chart_data)

# Información del sistema
st.subheader("Información del Sistema")
st.json({
    "hostname": system_info['hostname'],
    "ip_address": system_info['ip_address'],
    "memory_total_gb": round(system_info['memory_total_gb'], 2),
    "disk_total_gb": round(system_info['disk_total_gb'], 2),
    "timestamp": system_info['timestamp'].isoformat()
})

# Ejecutar con: streamlit run scripts/dashboard_streamlit.py
```
### Instalación y Configuración de Streamlit
```bash
# Crear entorno virtual
python3 -m venv ~/streamlit_env
source ~/streamlit_env/bin/activate

# Instalar Streamlit en el entorno virtual
pip install streamlit pandas numpy psutil

# Verificar instalación
streamlit version

# Ejecutar aplicación (en puerto 8501 por defecto)
streamlit run scripts/dashboard_streamlit.py --server.port 8501 --server.address 0.0.0.0
```
### Dashboard con Grafana
```bash
# Instalar Grafana
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana

# Habilitar e iniciar servicio
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

# Acceder via navegador: http://[IP_RASPBERRY]:3000
# Usuario: admin, Contraseña: admin (cambiar en primer login)
```
### Configuración de Data Source y Dashboard Sencillo
```json
// config/grafana_dashboard.json
{
  "dashboard": {
    "title": "Raspberry Pi Monitoring",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "stat",
        "targets": [
          {
            "expr": "100 - (avg by (instance) (irate(node_cpu_seconds_total{mode='idle'}[5m])) * 100)",
            "legendFormat": "CPU Usage"
          }
        ],
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0}
      },
      {
        "title": "Memory Usage",
        "type": "gauge",
        "targets": [
          {
            "expr": "(node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100",
            "legendFormat": "Memory Usage %"
          }
        ],
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0}
      }
    ]
  }
}
```
## 3. Automatización con Cron y Crontab
### Configuración de Tareas Automatizadas
```bash
# scripts/cron_automatizacion.sh
#!/bin/bash

# Script de automatización para Raspberry Pi
LOG_FILE="/home/pi/taller/logs/cron_$(date +\%Y\%m\%d).log"

echo "=== EJECUCIÓN CRON $(date) ===" >> $LOG_FILE

# 1. Respaldo de logs del sistema
echo "1. Respaldando logs..." >> $LOG_FILE
sudo cp /var/log/syslog /home/pi/backups/syslog_$(date +\%H\%M).bak

# 2. Limpieza de archivos temporales
echo "2. Limpiando archivos temporales..." >> $LOG_FILE
sudo find /tmp -name "*.tmp" -mtime +1 -delete >> $LOG_FILE 2>&1

# 3. Análisis de espacio en disco
echo "3. Verificando espacio en disco..." >> $LOG_FILE
df -h >> $LOG_FILE

# 4. Monitoreo de procesos
echo "4. Procesos activos:" >> $LOG_FILE
ps aux --sort=-%cpu | head -10 >> $LOG_FILE

# 5. Test de conectividad
echo "5. Test de conectividad..." >> $LOG_FILE
ping -c 3 8.8.8.8 >> $LOG_FILE 2>&1

echo "=== FIN EJECUCIÓN ===" >> $LOG_FILE
```
### Configuración de Crontab
```bash
# Editar crontab del usuario pi
crontab -e

# Agregar las siguientes líneas:
# config/crontab_example

# Ejecutar limpieza diaria a las 2:00 AM
0 2 * * * /home/pi/taller/scripts/cron_automatizacion.sh

# Monitoreo cada hora (minuto 5)
5 * * * * /usr/bin/df -h > /home/pi/taller/logs/disk_usage.log

# Reinicio semanal del servicio Streamlit cada domingo a las 3:00 AM
0 3 * * 0 /usr/bin/systemctl restart streamlit

# Respaldo de dashboards cada lunes a la 1:00 AM
0 1 * * 1 /bin/cp -r /home/pi/taller/dashboards /home/pi/backups/

# Verificar tareas programadas
crontab -l
```
## 4. Análisis y Auditoría de Red
### Script Completo de Análisis de Red
```bash
# scripts/analisis_red.sh
#!/bin/bash

echo "=== AUDITORÍA DE RED - RASPBERRY PI ==="
echo "Fecha: $(date)"
echo "Hostname: $(hostname)"
echo

# 1. INFORMACIÓN DE INTERFACES DE RED
echo "1. INTERFACES DE RED:"
ip addr show
echo

# 2. TABLA DE RUTEO
echo "2. TABLA DE RUTEO:"
ip route show
echo

# 3. VECINOS CERCANOS (ARP Table)
echo "3. TABLA ARP - DISPOSITIVOS LOCALES:"
arp -a
echo

# 4. ESCANEO DE RED LOCAL CON NMAP
echo "4. ESCANEO DE RED LOCAL:"
LOCAL_IP=$(hostname -I | awk '{print $1}')
NETWORK=$(echo $LOCAL_IP | cut -d. -f1-3)

echo "Escaneando red: ${NETWORK}.0/24"
nmap -sn ${NETWORK}.0/24
echo

# 5. ESCANEO DE PUERTOS EN GATEWAY
echo "5. ESCANEO DE PUERTOS - GATEWAY:"
GATEWAY=$(ip route | grep default | awk '{print $3}')
echo "Gateway: $GATEWAY"
nmap -sS -p 22,53,80,443,8080,8443 $GATEWAY
echo

# 6. ESCANEO DE SEGURIDAD LOCAL
echo "6. ESCANEO DE SEGURIDAD LOCAL:"
nmap -sS -sV -sC -O localhost
echo

# 7. DETECCIÓN DE SERVICIOS VULNERABLES
echo "7. DETECCIÓN DE VULNERABILIDADES:"
nmap --script vuln $GATEWAY
echo

# 8. MONITOREO DE CONEXIONES ACTIVAS
echo "8. CONEXIONES DE RED ACTIVAS:"
netstat -tulpn
echo

# 9. INFORMACIÓN DE DNS
echo "9. CONFIGURACIÓN DNS:"
cat /etc/resolv.conf
echo "Test DNS:"
nslookup google.com
echo

# 10. PRUEBAS DE CONECTIVIDAD
echo "10. PRUEBAS DE CONECTIVIDAD:"
ping -c 3 8.8.8.8
ping -c 3 google.com
echo

echo "=== AUDITORÍA COMPLETADA ==="
```
### Comandos Individuales de Análisis
```bash
# Instalación de Nmap
sudo apt install nmap -y

# Escaneo básico de red
sudo nmap -sn 192.168.1.0/24

# Identificación de dispositivos conectados
sudo arp-scan --localnet

# Escaneo de puertos específicos
sudo nmap -sS -p 22,80,443,8080 192.168.1.1

# Detección de sistema operativo remoto
sudo nmap -O 192.168.1.1

# Análisis de servicios y versiones
sudo nmap -sV 192.168.1.1
```
