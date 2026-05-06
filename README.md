# 🐳 WireGuard Easy VPN + Pi-hole en Docker

Docker Compose para instalar WireGuard Easy VPN y Pi-hole en Raspberry Pi, PC o servidores NAS.
WireGuard Easy es una VPN que permite acceder de forma remota a tu red local y navegar de forma segura incluso cuando utilizas redes Wi-Fi públicas, mientras que Pi-hole bloquea anuncios y rastreadores en todos los dispositivos de tu red local o conectados mediante WireGuard.

### 📺 [Tutorial completo en YouTube](https://youtu.be/C043K2Pie9Q?si=7RE5iNxDx3r4-ogH)

## 🛒 Hardware recomendado (links de afiliado)

### Raspberry Pi
- [Raspberry Pi Zero](https://amzlink.to/az0HhtYcw1gmc)
- [Raspberry Pi Zero Kit de Inicio](https://amzlink.to/az0W4uWgDihGD)
- [Raspberry Pi 5 (4GB)](https://amzlink.to/az0kKjY684Uj9)
- [Raspberry Pi 5 (8GB)](https://amzlink.to/az08MBpttxAE0)
- [Raspberry Pi 5 (Kit de Inicio)](https://amzlink.to/az0aU2Htv0MxJ)
- [Carcasa oficial para Raspberry Pi 5](https://amzlink.to/az0IhBFDFvJxC)
- [Ventilador oficial para Raspberry Pi 5](https://amzlink.to/az0OSK6YxlGEr)
- [Fuente de alimentación para Raspberry Pi](https://amzlink.to/az0mr1rElpTPl)
- [Tarjeta MicroSD para Raspberry Pi](https://amzlink.to/az0XrnBu87lwI)
- [Almacenamiento NVMe para Raspberry Pi](https://amzlink.to/az0NebFbPO6YI)
- [Carcasa para Raspberry Pi 5 + HAT NVMe](https://amzlink.to/az0Gzz3rMqaFK)

### Servidores NAS

- [UGREEN NAS DX2800](https://amzlink.to/az0qwiIp6ZNSl)
- [UGREEN NAS DH4300 Plus](https://amzlink.to/az0Oyzom5kXuu)
- [UGREEN NAS DXP4800](https://amzlink.to/az0BZ6iK7V3LS)
- [UGREEN NAS DXP4800 Plus](https://amzlink.to/az04N89RnbyGU)
- [Disco duro HDD para NAS](https://amzlink.to/az0ouuFKDlp18)

## ✅ Requisitos previos
Si estás empezando en el mundo del self-hosting y los servidores caseros, estos vídeos te servirán como base para utilizar una Raspberry Pi, un PC o un NAS como servidor doméstico con servicios autoalojados. Te recomiendo verlos antes de continuar con esta guía.
- [Raspberry Pi | Configuración Inicial](https://youtu.be/xRsxs5eBpmI?si=E7SvINDe1LTBV80S)
- [Raspberry Pi / PC | Instalar Docker y Portainer](https://youtu.be/-7vvELophxU?si=jD1oQdPo2f9jWDQN)
- [Servidor NAS | Instalar Docker y Portainer](https://youtu.be/hOiNrQXN-VE?si=ekD4vuqoADXxLphR)

## ⚙️ Instalación

Usa el `docker-compose.yml` correspondiente según el dispositivo donde lo vayas a instalar. Antes de desplegar los contenedores, modifica las líneas que contengan `# Comentario`

### Docker Compose para Raspberry Pi / PC

```yaml
networks:
  wireguard:
    driver: bridge
    enable_ipv6: true 
    ipam:
      driver: default
      config:
        - subnet: 10.42.42.0/24
        - subnet: fdcc:ad94:bacf:61a3::/64
        
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy:15
    container_name: wg-easy
    restart: unless-stopped
    networks:
      - wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    environment:
      - INSECURE=true
    volumes:
      - ./wireguard:/etc/wireguard # Directorio para archivos persistentes.
      - /lib/modules:/lib/modules:ro
    sysctls:
      - net.ipv4.ip_forward=1
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv6.conf.all.disable_ipv6=0
      - net.ipv6.conf.all.forwarding=1
      - net.ipv6.conf.default.forwarding=1
      
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    hostname: raspberrypi # Nombre de tu dispositivo.
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Europe/Madrid # Zona horaria.
      - FTLCONF_webserver_api_password=tupassword # Password de acceso a Pi-hole.
     # - FTLCONF_webserver_port=8080,8443s # Opcional: cambiar puertos 80:80 (HTTP) y 443:443 (HTTPS) para acceso web.
      - FTLCONF_dns_listeningMode=LOCAL
     # - FTLCONF_dns_specialDomains_iCloudPrivateRelay=true # Opcional: relay privado de Apple.
      - PIHOLE_UID=1000 # UID de tu máquina. Comprobar con comando: id -u
      - PIHOLE_GID=1000 # UID de tu máquina. Comprobar con comando: id -g
    volumes:
      - ./pihole:/etc/pihole # Directorio para archivos persistentes.
    cap_add:
     # - NET_ADMIN # Opcional: usar Pi-hole como servidor DHCP.
     # - SYS_TIME # Opcional: usar Pi-hole como cliente NTP.
      - SYS_NICE
 
  duckdns:
    image: lscr.io/linuxserver/duckdns:latest
    container_name: duckdns
    restart: unless-stopped
    network_mode: host
    environment:
      - PUID=1000 # UID de tu máquina. Comprobar con comando: id -u
      - PGID=1000 # UID de tu máquina. Comprobar con comando: id -g
      - TZ=Europe/Madrid # Zona horaria.
      - SUBDOMAINS=tusubdominio # Nombre del subdominio creado en duckdns.org
      - TOKEN=tutoken # Token de Duck DNS.
      - UPDATE_IP=both
      - LOG_FILE=false
    volumes:
      - ./duckdns:/config # Directorio para archivos persistentes.
```

### Docker Compose para servidor NAS (UGREEN, Synology, QNAP, etc)

```yaml
*** EN CONSTRUCCIÓN ***
```

### 🌐 Acceso web

#### WireGuard Easy:
`http://IP-DE-TU-SERVIDOR:51821`

#### Pi-hole (Raspberry Pi / PC):
`http://IP-DE-TU-SERVIDOR:80/admin` o `https://IP-DE-TU-SERVIDOR:443/admin`

#### Pi-hole (Servidor NAS):
`http://IP-DE-TU-PIHOLE/admin`

## 📄 Documentación oficial

### WireGuard Easy
- [WireGuard Easy GitHub](https://github.com/wg-easy/wg-easy)

### Pi-hole
- [Pi-hole en Docker](https://docs.pi-hole.net/docker/)
- [Pi-hole GitHub](https://github.com/pi-hole/docker-pi-hole)
- [Pi-hole Docker Hub](https://hub.docker.com/r/pihole/pihole)

### Duck DNS
- [Duck DNS en Docker](https://docs.linuxserver.io/images/docker-duckdns/)
- [Crear dominio en Duck DNS](https://www.duckdns.org)
