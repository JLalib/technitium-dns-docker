# 🌐 Technitium DNS Server Docker

[![GitHub](https://img.shields.io/badge/GitHub-TechnitiumSoftware%2FDnsServer-blue?logo=github)](https://github.com/TechnitiumSoftware/DnsServer)
[![Docker Hub](https://img.shields.io/badge/Docker%20Hub-technitium%2Fdns--server-blue?logo=docker)](https://hub.docker.com/r/technitium/dns-server)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

## 📋 Descripción general

**Technitium DNS Server** es un servidor DNS autohospedado profesional y open-source que funciona como servidor **autoritativo** y **recursivo** simultáneamente, bloqueando anuncios y malware a nivel DNS para toda tu red. Soporta protocolos modernos como **DNS-over-TLS (DoT)**, **DNS-over-HTTPS (DoH)**, **DNS-over-QUIC (DoQ)**, validación **DNSSEC**, clustering de múltiples instancias, **SSO OIDC**, y cuenta con una web console moderna y API HTTP REST. Es la alternativa completa y privada a confiar tu DNS en ISPs o servicios cloud.

Enterprise-grade pero fácil de usar: **100K+ requests/second** en hardware commodity (i7-8700), setup zero-config out-of-the-box, configuración vía variables de entorno, licencia MIT.

## ✨ Características principales

- 🎯 **DNS Autoritativo + Recursivo dual-mode** — Hosting zonas DNS custom + resolver queries clientes
- 🛡️ **Bloqueo anuncios integrado** — Múltiples blocklists simultáneos (AdGuard, Pi-hole, EasyList, uBlock Origin), granular allow/deny, stats real-time
- 🦠 **Bloqueo malware/phishing/PUP** — Bases de datos de amenazas, protección red-wide
- 🔐 **Protocolos DNS encriptados nativos** — DoT (RFC 7858), DoH (RFC 8484, HTTP/1.1/2/3), DoQ (RFC 9250)
- ✅ **DNSSEC validation** — Algoritmos RSA + ECDSA, RFC-compliant
- ⚡ **Forwarders modernos** — Cloudflare, Google, Quad9, AdGuard vía DoT/DoH/DoQ + conditional forwarders
- 🧠 **Caché inteligente** — Serve stale, prefetch, auto-prefetch, performance optimizado
- 🖥️ **Web console moderna** — UI browser completa, configuración gráfica, logs históricos
- 🔌 **API HTTP REST** — Totalmente automatable, usada por la propia UI
- 🔗 **Clustering multi-instancia** — Gestión 2+ servidores desde panel admin central, sync automático
- 🏢 **SSO OIDC enterprise** — OpenID Connect, rate-limiting, firewall rules, hotspot portal
- 🚀 **Performance ultra** — 100K+ req/sec, async I/O, commodity hardware
- 🐳 **Zero-config Docker** — Out-of-box setup, environment variables, MIT open source

## 📋 Requisitos del sistema

- Docker & Docker Compose v2+
- **100 MB – 500 MB RAM** mínimo (lightweight)
- **1 GB espacio disco** (logs, DB, caché)
- **Puerto 53 UDP/TCP** — DNS queries
- **Puerto 80/443** — Web console + DoH/DoQ
- **Puerto 5353 TCP** (opcional) — DoT
- Opcional: Reverse proxy (Caddy/nginx) para HTTPS

## 🐳 Instalación

### Opción 1: Docker Compose embebido simple (recomendado)

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
      # DoT (DNS-over-TLS, optional)
      - "5353:5353/tcp"
      # DoH/DoQ (DNS-over-HTTPS/QUIC)
      - "443:443/tcp"
    volumes:
      - dns-server-data:/etc/dns
      - dns-server-config:/config
    environment:
      - TZ=Europe/Madrid
      # Admin password (primera ejecución)
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

**Acceso:**
- `http://localhost` — Web console local (usuario: `admin`, contraseña: del `.env`)
- `https://dns.tudominio.com` — HTTPS remoto (si Caddy + dominio)

## ⚙️ Configuración

1. **Variables de entorno clave**
   - `TZ` — Zona horaria (ej: `Europe/Madrid`)
   - `DNS_SERVER_ADMIN_PASSWORD` — Contraseña admin inicial (cambiar obligatoriamente)
   - `DNS_SERVER_DOMAIN` — Dominio para DoH/DoQ (opcional)
   - `DNS_SERVER_HTTP_PORT` — Puerto HTTP web console (default: 80)
   - `DNS_SERVER_HTTPS_PORT` — Puerto HTTPS DoH/DoQ (default: 443)

2. **Volúmenes persistentes**
   - `/etc/dns` — Datos DNS (zonas, caché, DB)
   - `/config` — Configuración, certificados, settings.json

3. **Puertos expuestos**
   - `53/udp,tcp` — DNS estándar (obligatorio)
   - `80/tcp` — Web console HTTP / DoH
   - `443/tcp` — DoH/DoQ HTTPS
   - `5353/tcp` — DoT (opcional)

## 🚀 Primeros pasos

1. **Acceder a la web console**  
   Abre `http://localhost` → Login: usuario `admin`, contraseña del `.env` → Dashboard con resumen stats DNS

2. **Cambiar contraseña admin (IMPORTANTE)**  
   Dashboard → Settings → Admin → "Change Password" → Nueva contraseña fuerte → Save

3. **Configurar como servidor DNS de red**  
   En router/DHCP: DNS primario = IP servidor (ej: `192.168.1.100`)  
   O en cada cliente: DNS manual = IP servidor  
   Prueba: `nslookup google.com 192.168.1.100`

4. **Agregar blocklists (anuncios + malware)**  
   Dashboard → Blocklists → "+ Add" → URL blocklist → Enable → Auto-descarga/actualiza  
   Ejemplos:  
   - AdGuard DNS: `https://adguardteam.github.io/AdGuardHome/Blocklists/index.html`  
   - EasyList: `https://easylist.to/`  
   - uBlock Origin: `https://github.com/uBlockOrigin/uAssets/tree/master/filters`

5. **Configurar forwarders (upstream DNS)**  
   Dashboard → Forwarders → "+ Add" → Ejemplos:  
   - Cloudflare DoH: `https://1.1.1.1/dns-query`  
   - Google DoT: `tls://8.8.8.8:853`  
   Enable → Usan forwarder para recursive queries

6. **Configurar DoH/DoQ (DNS encriptado)**  
   Dashboard → Settings → Protocols → Enable "DNS-over-HTTPS" (puerto 443) → Enable "DNS-over-QUIC" (puerto 443)  
   Certificados auto (Let's Encrypt si Caddy)  
   Clients usan: `https://dns.tudominio.com/dns-query` (DoH) o `quic://dns.tudominio.com` (DoQ)

7. **Ver query logs y stats**  
   Dashboard → Query Logs → Lista todas queries (original + resultado) → Filter por dominio/IP/tipo  
   Dashboard → Statistics → Gráficos queries/bloques/performance

8. **Crear zona DNS autoritativa custom (opcional)**  
   Dashboard → Zones → "+ Add Zone" → Zone name: ej. `internal.local` → Tipo: Primary → Crea registros (A, CNAME, MX, etc)  
   Queries a `internal.local` → Resuelven localmente

9. **Configurar clustering (multi-servidor, opcional)**  
   En segundo servidor: Settings → Clustering → Enable Cluster Mode → Primary server: IP primer servidor → Cluster authentication token: generar

10. **Monitorear performance**  
    Dashboard → Gráficos real-time (queries/sec, latencia)  
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
- Certificados TLS válidos y renovación automática
- DoH/DoQ funcionan sobre puerto 443 estándar
- Acceso web console seguro desde cualquier red
- Requiere dominio válido apuntando a tu IP pública + puertos 80/443 abiertos

Alternativa: VPN (WireGuard/Tailscale) + acceso solo LAN → Máxima seguridad sin exposición pública.

## 🛠️ Gestión y mantenimiento

```bash
# Ver logs en tiempo real
docker logs -f technitium-dns

# Reiniciar container
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

> 📖 **Guía completa en el blog:** [Cómo instalar Technitium DNS Server en Docker - Servidor DNS autohospedado, bloqueador anuncios, privacidad](https://genbyte.blogspot.com/2026/08/como-instalar-technitium-dns-server-en.html)  
> 🎥 **Canal YouTube:** [Genbyte](https://www.youtube.com/@genbyte) | 📧 **Newsletter:** [Suscríbete gratis](https://genbyte.blogspot.com/) | ☕ **Apoya:** [Ko-fi](https://ko-fi.com/genbyte)