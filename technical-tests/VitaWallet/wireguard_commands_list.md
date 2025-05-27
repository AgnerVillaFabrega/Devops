# Lista de Comandos - Implementación VPN WireGuard en GCP

## 🏗️ Configuración de Proyectos GCP

### Cuenta Principal
```bash
# Crear el proyecto
gcloud projects create mi-proyecto-vpn-460916 --name="Mi Proyecto VPN"
```
**Descripción:** Crea un nuevo proyecto en Google Cloud Platform

```bash
# Configurar proyecto activo
gcloud config set project mi-proyecto-vpn-460916
```
**Descripción:** Establece el proyecto como activo en la CLI de gcloud

```bash
# Habilitar servicios de cómputo
gcloud services enable compute.googleapis.com
```
**Descripción:** Habilita la API de Compute Engine para crear instancias VM

### Cuenta Tercero/Cliente
```bash
# Crear proyecto del tercero
gcloud projects create tercero-proyecto-vpn --name="Tercero Proyecto VPN"
gcloud config set project tercero-proyecto-vpn
gcloud services enable compute.googleapis.com
```
**Descripción:** Mismos comandos para configurar la cuenta del cliente VPN

## 🌐 Configuración de Redes

### VPC y Subredes
```bash
# Crear VPC personalizada
gcloud compute networks create vpc-propia \
 --subnet-mode=custom \
 --bgp-routing-mode=regional
```
**Descripción:** Crea una red VPC con modo de subred personalizado

```bash
# Crear subred
gcloud compute networks subnets create subnet-propia \
 --network=vpc-propia \
 --region=us-central1 \
 --range=10.0.1.0/24
```
**Descripción:** Crea una subred dentro de la VPC con rango IP específico

### Reglas de Firewall
```bash
# Regla para WireGuard
gcloud compute firewall-rules create allow-wireguard \
 --network=vpc-propia \
 --allow=udp:51820 \
 --source-ranges=0.0.0.0/0
```
**Descripción:** Permite tráfico UDP en puerto 51820 para WireGuard

```bash
# Regla para SSH
gcloud compute firewall-rules create allow-ssh \
 --network=vpc-propia \
 --allow=tcp:22 \
 --source-ranges=0.0.0.0/0
```
**Descripción:** Permite conexiones SSH en puerto 22

## 🖥️ Creación de Instancias

```bash
# Crear instancia servidor
gcloud compute instances create wg-server \
 --zone=us-central1-a \
 --machine-type=e2-micro \
 --network-interface=network=vpc-propia,subnet=subnet-propia \
 --image-family=ubuntu-2004-lts \
 --image-project=ubuntu-os-cloud \
 --boot-disk-size=10GB \
 --tags=wireguard-server
```
**Descripción:** Crea la instancia VM que actuará como servidor VPN

```bash
# Crear instancia cliente
gcloud compute instances create wg-client \
 --zone=us-central1-a \
 --machine-type=e2-micro \
 --network-interface=network=vpc-tercero,subnet=subnet-tercero \
 --image-family=ubuntu-2004-lts \
 --image-project=ubuntu-os-cloud \
 --boot-disk-size=10GB \
 --tags=wireguard-client
```
**Descripción:** Crea la instancia VM que actuará como cliente VPN

## 🔐 Instalación y Configuración de WireGuard

### Conexión SSH
```bash
# Conectarse al servidor
gcloud compute ssh wg-server --zone=us-central1-a
```
**Descripción:** Establece conexión SSH con la instancia servidor

```bash
# Conectarse al cliente
gcloud compute ssh wg-client --zone=us-central1-a
```
**Descripción:** Establece conexión SSH con la instancia cliente

### Instalación
```bash
# Actualizar paquetes
sudo apt update
```
**Descripción:** Actualiza la lista de paquetes disponibles

```bash
# Instalar WireGuard
sudo apt install -y wireguard
```
**Descripción:** Instala WireGuard y sus dependencias

### Generación de Claves
```bash
# Cambiar al directorio de WireGuard
cd /etc/wireguard
```
**Descripción:** Navega al directorio de configuración de WireGuard

```bash
# Establecer permisos restrictivos
umask 077
```
**Descripción:** Configura permisos para que solo el owner pueda leer/escribir

```bash
# Generar clave privada del servidor
wg genkey | sudo tee server_private.key
```
**Descripción:** Genera y guarda la clave privada del servidor

```bash
# Generar clave pública del servidor
sudo cat server_private.key | wg pubkey | sudo tee server_public.key
```
**Descripción:** Genera la clave pública correspondiente a partir de la privada

```bash
# Guardar claves en variables
SERVER_PRIVATE_KEY=$(sudo cat server_private.key)
SERVER_PUBLIC_KEY=$(sudo cat server_public.key)
```
**Descripción:** Almacena las claves en variables de entorno para fácil acceso

## 📝 Configuración de Archivos

```bash
# Crear archivo de configuración del servidor
sudo nano /etc/wireguard/wg0.conf
```
**Descripción:** Crea y abre el archivo de configuración principal de WireGuard

```bash
# Establecer permisos del archivo de configuración
sudo chmod 600 /etc/wireguard/wg0.conf
```
**Descripción:** Restringe permisos del archivo de configuración solo al owner

## 🌐 Configuración de Red

```bash
# Habilitar IP forwarding temporalmente
sudo sysctl -w net.ipv4.ip_forward=1
```
**Descripción:** Habilita el reenvío de paquetes IP entre interfaces

```bash
# Habilitar IP forwarding permanentemente
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
```
**Descripción:** Hace persistente la configuración de IP forwarding

## ⚙️ Gestión del Servicio WireGuard

```bash
# Habilitar servicio para inicio automático
sudo systemctl enable wg-quick@wg0
```
**Descripción:** Configura WireGuard para iniciarse automáticamente al boot

```bash
# Iniciar servicio WireGuard
sudo systemctl start wg-quick@wg0
```
**Descripción:** Inicia el servicio WireGuard con configuración wg0

```bash
# Reiniciar servicio WireGuard
sudo systemctl restart wg-quick@wg0
```
**Descripción:** Reinicia el servicio para aplicar cambios de configuración

```bash
# Verificar estado del servicio
sudo systemctl status wg-quick@wg0
```
**Descripción:** Muestra el estado actual del servicio WireGuard

## 📊 Monitoreo y Diagnóstico

```bash
# Ver estado de WireGuard
sudo wg show
```
**Descripción:** Muestra información detallada de interfaces y peers activos

```bash
# Ver estadísticas detalladas
sudo wg show wg0 dump
```
**Descripción:** Muestra estadísticas completas de la interfaz wg0

```bash
# Ver logs del servicio
sudo journalctl -u wg-quick@wg0 -f
```
**Descripción:** Muestra logs en tiempo real del servicio WireGuard

```bash
# Ver logs específicos
sudo journalctl -u wg-quick@wg0 -n 50
```
**Descripción:** Muestra las últimas 50 líneas de logs del servicio

## 🔍 Comandos de Red y Diagnóstico

```bash
# Obtener IP pública
curl -s ifconfig.me
```
**Descripción:** Obtiene la dirección IP pública de la instancia

```bash
# Obtener IP pública via gcloud
gcloud compute instances describe wg-server \
 --zone=us-central1-a \
 --project=mi-proyecto-vpn-460916 \
 --format='get(networkInterfaces[0].accessConfigs[0].natIP)'
```
**Descripción:** Obtiene la IP pública de una instancia usando gcloud CLI

```bash
# Ver interfaces de red
ip addr show
```
**Descripción:** Muestra todas las interfaces de red y sus configuraciones

```bash
# Ver interfaz WireGuard específica
ip addr show wg0
```
**Descripción:** Muestra información específica de la interfaz WireGuard

```bash
# Ver tabla de rutas
ip route show
```
**Descripción:** Muestra la tabla de enrutamiento del sistema

## 🧪 Pruebas de Conectividad

```bash
# Ping básico al servidor VPN
ping -c 4 10.100.0.1
```
**Descripción:** Envía 4 paquetes ICMP para probar conectividad básica

```bash
# Ping desde servidor a cliente
ping -c 10 10.100.0.2
```
**Descripción:** Prueba conectividad desde el servidor hacia el cliente

```bash
# Monitor de tráfico en interfaz WireGuard
sudo tcpdump -i wg0 -n
```
**Descripción:** Captura y muestra tráfico de red en la interfaz WireGuard

## 📁 Pruebas de Transferencia

```bash
# Crear archivo de prueba
echo "Prueba de transferencia VPN" > /tmp/prueba.txt
```
**Descripción:** Crea un archivo de texto para pruebas de transferencia

```bash
# Iniciar servidor HTTP simple
cd /tmp && python3 -m http.server 8000
```
**Descripción:** Inicia un servidor HTTP básico para transferencia de archivos

```bash
# Descargar archivo via HTTP
wget http://10.100.0.1:8000/prueba.txt -O /tmp/prueba_recibida.txt
```
**Descripción:** Descarga archivo del servidor usando wget

```bash
# Verificar contenido del archivo
cat /tmp/prueba_recibida.txt
```
**Descripción:** Muestra el contenido del archivo descargado

## 📈 Pruebas de Rendimiento

```bash
# Instalar iperf3
sudo apt install -y iperf3
```
**Descripción:** Instala la herramienta de pruebas de rendimiento iperf3

```bash
# Iniciar servidor iperf3
iperf3 -s
```
**Descripción:** Inicia iperf3 en modo servidor para pruebas de rendimiento

```bash
# Ejecutar prueba de rendimiento
iperf3 -c 10.100.0.1 -t 30
```
**Descripción:** Ejecuta prueba de rendimiento durante 30 segundos

## 🛠️ Resolución de Problemas

```bash
# Instalar resolvconf (solución al error DNS)
sudo apt update
sudo apt install resolvconf
```
**Descripción:** Instala resolvconf para resolver problemas de DNS en WireGuard

```bash
# Ver información detallada del archivo de configuración
sudo cat /etc/wireguard/wg0.conf
```
**Descripción:** Muestra el contenido del archivo de configuración para debug

```bash
# Ver todos los servicios relacionados con WireGuard
sudo systemctl list-units | grep wireguard
```
**Descripción:** Lista todos los servicios de systemd relacionados con WireGuard

## 📋 Comandos de Información del Sistema

```bash
# Ver información del sistema
uname -a
```
**Descripción:** Muestra información completa del sistema operativo

```bash
# Ver uso de memoria
free -h
```
**Descripción:** Muestra el uso actual de memoria en formato legible

```bash
# Ver procesos relacionados con WireGuard
ps aux | grep wireguard
```
**Descripción:** Muestra procesos activos relacionados con WireGuard

```bash
# Ver puertos en escucha
sudo netstat -tulpn | grep 51820
```
**Descripción:** Verifica que WireGuard esté escuchando en el puerto 51820