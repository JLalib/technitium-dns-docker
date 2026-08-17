# 🌐 Technitium DNS Server Docker

[![GitHub](https://img.shields.io/badge/GitHub-technitium%2Fdns--server-blue?logo=github)](https://github.com/TechnitiumSoftware/DnsServer)
[![Docker](https://img.shields.io/badge/Docker-technitium%2Fdns--server-blue?logo=docker)](https://hub.docker.com/r/technitium/dns-server)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

## 📋 Descripción general

**Technitium DNS Server** es un servidor DNS autohospedado profesional y open-source que funciona como servidor **autoritativo** y **recursivo** simultáneamente, bloqueando anuncios y malware a nivel DNS para toda tu red. Soporta protocolos modernos como **DNS-over-TLS/HTTPS/QUIC**, validación **DNSSEC**, **clustering** de múltiples instancias, **SSO OIDC**, y cuenta con una **web console moderna**, siendo la alternativa completa y privada a confiar tu DNS en ISPs o servicios cloud. Es enterprise-grade pero fácil de usar.

## ✨ Características principales

- 🎯 **DNS Autoritativo + Recursivo dual-mode** — Hosting zonas DNS custom + resolver queries clientes
- 🛡️ **Bloqueador anuncios integrado** — Múltiples blocklists simultáneos (AdGuard, Pi-hole, EasyList, uBlock Origin), granular allow/deny, stats real-time
- 🦠 **Bloqueo malware/phishing/PUP** — Bases de datos de amenazas, protección red-wide
- 🔐 **Protocolos DNS encriptados nativos** — DoT (RFC 7858), DoH (RFC 8484, HTTP/1.1/2/3), DoQ (RFC 9250)
- ✅ **DNSSEC validation** — Algoritmos RSA + ECDSA, RFC-compliant
- ⚡ **Forwarders modernos** — Cloudflare, Google, Quad9, AdGuard vía DoT/DoH/DoQ + conditional forwarders
- 🧠 **Caché inteligente** — Serve stale, prefetch, auto-prefetch, optimizado para performance
- 🖥️ **Web console moderna** — UI gráfica en navegador, configuración visual, logs históricos
- 🔌 **API HTTP REST** — Totalmente automatable, usada por la propia UI
- 🔗 **Clustering multi-instancia** — Gestión centralizada de 2+ servidores, sync automático
- 🏢 **SSO OIDC + Enterprise** — OpenID Connect, rate-limiting, firewall rules, hotspot portal
- 🚀 **Performance ultra** — 100K+ req/sec en hardware commodity (i7-8700), async I/O
- 🐳 **Zero-config Docker** — Out-of-box setup, configuración por environment variables
- 📜 **MIT Open Source** — Licencia permisiva, comunidad activa

## 📋 Requisitos del sistema

- Docker & Docker Compose v2+
- **RAM**: 100 MB – 500 MB mínimo (lightweight)
- **Disco**: 1 GB espacio (logs, DB, caché)
- **Puertos obligatorios**:
  - `53/udp` + `53/tcp` — DNS queries
  - `80/tcp` — Web console HTTP
  - `443/tcp` — DoH/DoQ + Web console HTTPS
- **Puertos opcionales**:
  - `5353/tcp` — DoT (DNS-over-TLS)
- **Opcional**: Reverse proxy (Caddy/nginx) para HTTPS automático

## 🐳 Instalación

### Opción 1: Docker Compose simple (recomendado)

```bash
mkdir -p dns-server && cd dns-server
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  dns-server:
    image: technitium/dns-server:latest
    container_name: technitium-dns
    restart: unless-stopped
    ports:
      # DNS UDP/TCP
      - "53:53/udp"
      - "53:53/tcp"
      # Web console HTTP
      - "80:80"
      # DoT (DNS-over-TLS, opcional)
      - "5353:5353/tcp"
      # DoH/DoQ (DNS-over-HTTPS/QUIC)
      - "443:443/tcp"
    volumes:
      - dns-server-data:/etc/dns
      - dns-server-config:/config
    environment:
      - TZ=Europe/Madrid
      # Admin password (primera ejecución) — CAMBIAR DESPUÉS
      - DNS_SERVER_ADMIN_PASSWORD=cambiar_despues_fuerte
    healthcheck:
      test: ["CMD", "dig", "@localhost", "technitium.com"]
      interval: 30s
      timeout: 5s
      retries: 3

volumes:
  dns-server-data:
  dns-server-config:
EOF

docker compose up -d
```

### Opción 2: Con reverse proxy Caddy (HTTPS automático)

```bash
mkdir -p dns-server && cd dns-server
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  dns-server:
    image: technitium/dns-server:latest
    container_name: technitium-dns
    restart: unless-stopped
    ports:
      - "53:53/udp"
      - "53:53/tcp"
    volumes:
      - dns-server-data:/etc/dns
      - dns-server-config:/config
    environment:
      - TZ=Europe/Madrid
      - DNS_SERVER_ADMIN_PASSWORD=cambiar_despues

  caddy:
    image: caddy:latest
    container_name: dns-caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy-data:/data
      - caddy-config:/config
    depends_on:
      - dns-server

volumes:
  dns-server-data:
  dns-server-config:
  caddy-data:
  caddy-config:
EOF

cat > Caddyfile << 'EOF'
dns.tudominio.com {
    reverse_proxy dns-server:80
}
EOF

docker compose up -d
```

### Acceso a la interfaz

| Entorno | URL | Credenciales |
|---------|-----|--------------|
| Local | `http://localhost` | `admin` / password del `.env` |
| Remoto (Caddy + dominio) | `https://dns.tudominio.com` | `admin` / password del `.env` |

## ⚙️ Configuración

1. **Zona horaria** — Variable `TZ` (ej: `Europe/Madrid`)
2. **Contraseña admin inicial** — Variable `DNS_SERVER_ADMIN_PASSWORD` (cambiar obligatoriamente en primer login)
3. **Persistencia** — Dos volúmenes: `/etc/dns` (datos DNS) y `/config` (configuración)
4. **Healthcheck** — Verifica resolución DNS cada 30s con `dig @localhost technitium.com`
5. **Puertos expuestos** — 53 (UDP/TCP), 80, 443, 5353 (opcional DoT)

## 🚀 Primeros pasos

1. **Acceder a la web console**  
   Abre `http://localhost` → Login: `admin` / contraseña del `.env` → Dashboard con stats DNS

2. **Cambiar contraseña admin (IMPORTANTE)**  
   `Dashboard → Settings → Admin` → Click "Change Password" → Nueva contraseña fuerte → Save

3. **Configurar como servidor DNS de red**  
   - En router/DHCP: DNS primario = IP del servidor (ej: `192.168.1.100`)  
   - O en cada cliente: DNS manual = IP del servidor  
   - Probar: `nslookup google.com 192.168.1.100`

4. **Agregar blocklists (anuncios + malware)**  
   `Dashboard → Blocklists` → Click "+ Add" → URL blocklist → Enable → Auto-descarga/actualiza  
   **Ejemplos URLs**:  
   - AdGuard DNS: `https://adguardteam.github.io/AdGuardHome/Blocklists/index.html`  
   - EasyList: `https://easylist.to/`  
   - uBlock Origin: `https://github.com/uBlockOrigin/uAssets/tree/master/filters`

5. **Configurar forwarders (upstream DNS)**  
   `Dashboard → Forwarders` → "+ Add" → Ejemplos:  
   - Cloudflare DoH: `https://1.1.1.1/dns-query`  
   - Google DoT: `tls://8.8.8.8:853`  
   Enable → Usan forwarder para queries recursivas

6. **Configurar DoH/DoQ (DNS encriptado)**  
   `Dashboard → Settings → Protocols` → Enable "DNS-over-HTTPS" (puerto 443) + "DNS-over-QUIC" (puerto 443)  
   Certificados auto (Let's Encrypt si Caddy)  
   Clients usan: `https://dns.tudominio.com/dns-query` (DoH) o `quic://dns.tudominio.com` (DoQ)

7. **Ver query logs y stats**  
   `Dashboard → Query Logs` → Lista queries (original + resultado), filtros por dominio/IP/tipo  
   `Dashboard → Statistics` → Gráficos queries/bloques/performance

8. **Crear zona DNS autoritativa custom (opcional)**  
   `Dashboard → Zones` → "+ Add Zone" → Zone name: `internal.local` → Tipo: Primary  
   Crea registros A, CNAME, MX, etc → Queries a `internal.local` resuelven localmente

9. **Configurar clustering (multi-servidor, opcional)**  
   En segundo servidor: `Settings → Clustering` → Enable Cluster Mode  
   Primary server: IP primer servidor → Cluster authentication token: generar

10. **Monitorear performance**  
    `Dashboard → gráficos real-time` (queries/sec, latencia)  
    Logs: `docker logs -f technitium-dns`  
    Stats: `docker stats technitium-dns`

## 💡 Casos de uso

- 🏠 **Homelabs** — DNS privado, bloqueo anuncios red-wide, cero dependencia ISP
- 🏢 **SMB/Empresa** — DNS corporativo, control granular, clustering multi-sitio
- 🔒 **Privacidad total** — No expone queries a ISP, forwarders DoT/DoH/DoQ encriptados
- 🌍 **Hosting DNS** — Zonas autoritativas custom, hosting dominio privado
- 🛡️ **Seguridad** — Bloquea malware/phishing red-wide, filtrado DNS-level
- 🧪 **Lab/desarrollo** — Zonas test (`internal.local`, `dev.local`), conditional forwarders

## 🔒 Acceso remoto seguro

La **Opción 2 (Caddy)** proporciona HTTPS automático con Let's Encrypt:
- Configura `dns.tudominio.com` en tu DNS público → IP pública
- Caddy obtiene/renueva certificados TLS automáticamente
- DoH/DoQ funcionan sobre puerto 443 estándar (traversable firewalls)
- Web console accesible vía `https://dns.tudominio.com`

**Alternativa**: VPN (WireGuard/Tailscale) + acceso solo LAN para máxima privacidad.

## 🛠️ Gestión y mantenimiento

```bash
# Ver logs en tiempo real
docker logs -f technitium-dns

# Reiniciar contenedor
docker compose restart dns-server

# Actualizar imagen
docker pull technitium/dns-server:latest
docker compose up -d

# Monitorear consumo (ultra-ligero)
docker stats technitium-dns
# Típicamente: 50-200MB RAM, <1% CPU

# Backup configuración
docker cp technitium-dns:/config ./dns-config-backup-$(date +%Y%m%d)

# Reset a valores por defecto
docker exec technitium-dns rm -rf /config/settings.json
docker compose restart dns-server

# Consulta manual DNS (test)
dig @localhost google.com
nslookup google.com 127.0.0.1
```

## 📝 Licencia

**MIT License** — Código abierto, uso comercial permitido, sin garantía.  
Basado en [TechnitiumSoftware/DnsServer](https://github.com/TechnitiumSoftware/DnsServer) (MIT).

---

> 📖 **Guía completa en el blog**: [Cómo instalar Technitium DNS Server en Docker - Servidor DNS autohospedado, bloqueador anuncios, privacidad](https://genbyte.blogspot.com/2026/08/como-instalar-technitium-dns-server-en.html)  
> 🎥 **Canal YouTube**: [Genbyte](https://www.youtube.com/@Genbyte) | 💬 **Comunidad**: [Telegram](https://t.me/genbyte) | ☕ **Apoya**: [Ko-fi](https://ko-fi.com/genbyte)