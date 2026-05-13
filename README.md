# 🐳 WireGuard Easy VPN + Pi-hole en Docker

Docker Compose para instalar WireGuard Easy VPN y Pi-hole en Raspberry Pi, PC o servidores NAS.
WireGuard Easy es una VPN que permite acceder de forma remota a tu red local y navegar de forma segura incluso cuando utilizas redes Wi-Fi públicas, mientras que Pi-hole bloquea anuncios y rastreadores en todos los dispositivos de tu red local o conectados mediante WireGuard.

**📺 [Tutorial completo en YouTube](https://youtu.be/C043K2Pie9Q?si=7RE5iNxDx3r4-ogH)**


## 🛒 Hardware recomendado

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
Si estás empezando en el mundo del self-hosting, estos vídeos te servirán como base para usar una Raspberry Pi, un PC o un NAS como servidor doméstico con servicios autoalojados. Te recomiendo verlos antes de continuar con esta guía.
- 📺 [Raspberry Pi 5: Configuración de cero](https://youtu.be/xRsxs5eBpmI?si=E7SvINDe1LTBV80S)
- 📺 [Instalar Docker y Portainer en Raspberry Pi 5 / PC](https://youtu.be/-7vvELophxU?si=jD1oQdPo2f9jWDQN)
- 📺 [Instalar Docker y Portainer en servidor NAS](https://youtu.be/hOiNrQXN-VE?si=ekD4vuqoADXxLphR)


## ⚙️ Instalación

Usa el `docker-compose.yml` correspondiente según el dispositivo donde lo vayas a instalar. Antes de desplegar los contenedores, modifica las líneas que contengan `# Comentario`

### Docker Compose para Raspberry Pi / PC

```yaml
networks:
  wireguard_net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 10.42.42.0/24
        
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy:15
    container_name: wg-easy
    restart: unless-stopped
    networks:
      - wireguard_net
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    ports:
     # Comprobar puertos libres con comando: sudo ss -tulpn | grep :puerto 
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
      
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    hostname: raspberrypi # Nombre de tu dispositivo.
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Europe/Madrid # Zona horaria. Comprobar aquí: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
      - FTLCONF_webserver_api_password=tupassword # Password de acceso a Pi-hole.
     # - FTLCONF_webserver_port=8080,8443s # Opcional: cambiar puertos 80:80 (HTTP) y 443:443 (HTTPS) para acceso web.
      - FTLCONF_dns_listeningMode=ALL
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
      - TZ=Europe/Madrid # Zona horaria. Comprobar aquí: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
      - SUBDOMAINS=tusubdominio # Nombre del subdominio creado en duckdns.org
      - TOKEN=tutoken # Token de Duck DNS.
      - UPDATE_IP=ipv4
      - LOG_FILE=false
    volumes:
      - ./duckdns:/config # Directorio para archivos persistentes.
```

### Docker Compose para servidor NAS (UGREEN, Synology, QNAP, etc)

```yaml
networks:
  wireguard_net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 10.42.42.0/24
        
  pihole_macvlan:
    driver: macvlan
    driver_opts:
      parent: eth0 # Interfaz de red de tu máquina. Comprobar con comando: ip route | grep default
    ipam:
      config:
        - subnet: 192.168.1.0/24 # Rango de IPs de tu red local.
          gateway: 192.168.1.1 # IP de tu router.
        
services:
  wg-easy:
    image: ghcr.io/wg-easy/wg-easy:15
    container_name: wg-easy
    restart: unless-stopped
    networks:
      - wireguard_net
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    ports:
     # Comprobar puertos libres con comando: sudo ss -tulpn | grep :puerto 
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

  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    hostname: ugreennas # Nombre de tu dispositivo.
    restart: unless-stopped
    networks:
      pihole_macvlan:
        ipv4_address: 192.168.1.5 # IP libre en tu LAN para Pi-hole.
    environment:
      - TZ=Europe/Madrid #Zona horaria. Comprobar aquí: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
      - FTLCONF_webserver_api_password=tupassword # Password de acceso a Pi-hole.
      - FTLCONF_dns_listeningMode=ALL
     # - FTLCONF_dns_specialDomains_iCloudPrivateRelay=true # Opcional: relay privado de Apple.
      - PIHOLE_UID=1000 # UID de tu máquina. Comprobar con comando: id -u
      - PIHOLE_GID=10 # GID de tu máquina. Comprobar con comando: id -g
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
      - TZ=Europe/Madrid # Zona horaria. Comprobar aquí: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
      - SUBDOMAINS=tusubdominio # Nombre del subdominio creado en duckdns.org
      - TOKEN=tutoken # Token de Duck DNS.
      - UPDATE_IP=ipv4
      - LOG_FILE=false
    volumes:
      - ./duckdns:/config # Directorio para archivos persistentes.
```


### 🌐 Acceso web

#### WireGuard Easy:
- Panel web WireGuard Easy: `http://IP-DE-TU-SERVIDOR:51821`

#### Pi-hole (Raspberry Pi / PC):
- Panel web Pi-hole por HTTP: `http://IP-DE-TU-SERVIDOR:80/admin`
- Panel web Pi-hole por HTTPS: `https://IP-DE-TU-SERVIDOR:443/admin`

#### Pi-hole (Servidor NAS):
- Panel web Pi-hole: `http://IP-DE-TU-PIHOLE/admin`


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
