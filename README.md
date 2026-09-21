# 🌐 Technitium DNS Server Docker

[![GitHub](https://img.shields.io/badge/GitHub-technitium%2Fdns--server-blue?logo=github)](https://github.com/TechnitiumSoftware/DnsServer)
[![Docker](https://img.shields.io/badge/Docker-technitium%2Fdns--server-blue?logo=docker)](https://hub.docker.com/r/technitium/dns-server)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

## 📋 Descripción general

**Technitium DNS Server** es un servidor DNS autohospedado profesional y open-source que funciona como servidor **autoritativo y recursivo simultáneamente**, bloqueando anuncios y malware a nivel DNS para toda tu red. Soporta protocolos modernos como **DNS-over-TLS/HTTPS/QUIC**, validación **DNSSEC**, clustering de múltiples instancias, **SSO OIDC**, y cuenta con una web console moderna y amigable. Es la alternativa completa y privada a confiar tu DNS en ISPs o servicios cloud.

Enterprise-grade pero fácil de usar. **100K+ requests/second** en hardware commodity (i7-8700). MIT open source.

## ✨ Características principales

- 🎯 **DNS Autoritativo + Recursivo dual-mode** — Hosting zonas custom + resolver queries clientes
- 🛡️ **Bloqueo anuncios** — Múltiples blocklists simultáneos (AdGuard, Pi-hole, EasyList, uBlock), granular allow/deny, stats real-time
- 🦠 **Bloqueo malware** — Bases de datos virus/phishing/PUP, protección red-wide
- 🔐 **DoT/DoH/DoQ nativos** — DNS-over-TLS (RFC 7858), DNS-over-HTTPS (RFC 8484, HTTP/1.1/2/3), DNS-over-QUIC (RFC 9250)
- ✅ **DNSSEC validation** — Algoritmos RSA + ECDSA, RFC-compliant
- ⚡ **Forwarders modernos** — Cloudflare, Google, Quad9, AdGuard vía DoT/DoH/DoQ + conditional forwarders
- 🧠 **Caché inteligente** — Serve stale, prefetch, auto-prefetch, performance optimizado
- 🖥️ **Web console moderna** — UI browser, configuración gráfica, logs históricos
- 🔌 **API HTTP REST** — Automatizable, integrable, usada por la propia UI
- 🔗 **Clustering** — Gestión 2+ instancias desde panel admin central, sync automático
- 🏢 **SSO OIDC** — OpenID Connect enterprise auth
- 📊 **Logging & Firewall** — Query logs históricos, firewall rules, rate-limiting, hotspot portal
- 🚀 **Performance ultra** — 100K+ req/sec, async I/O, commodity hardware
- 🐳 **Zero-config Docker** — Out-of-box setup, environment variables, MIT open source

## 📋 Requisitos del sistema

- Docker & Docker Compose v2+
- 100 MB – 500 MB RAM mínimo (lightweight)
- 1 GB espacio disco (logs, DB, caché)
- Puerto **53 UDP/TCP** (DNS queries)
- Puerto **80/443** (web console + DoH/DoQ)
- Puerto **5353** opcional (DoT)
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
   - `DNS_SERVER_ADMIN_PASSWORD` — Contraseña inicial admin (cambiar en primer login)
2. **Volúmenes persistentes**
   - `/etc/dns` — Datos DNS (zonas, caché, DB)
   - `/config` — Configuración (settings.json, certificados)
3. **Puertos expuestos**
   - `53/udp,tcp` — DNS estándar (obligatorio)
   - `80` — Web console HTTP / DoH
   - `443` — DoH/DoQ + HTTPS web console
   - `5353/tcp` — DoT (opcional)
4. **Healthcheck** — Verifica resolución DNS interna cada 30s
5. **Reverse proxy (Caddy)** — Termina TLS, obtiene certificados Let's Encrypt automáticos

## 🚀 Primeros pasos

1. **Acceder web console**  
   Abre `http://localhost` → Login: usuario `admin`, contraseña del `.env` → Dashboard aparece (resumen stats DNS)

2. **Cambiar contraseña admin (IMPORTANTE)**  
   Dashboard → Settings → Admin → Click "Change Password" → Ingresa nueva contraseña fuerte → Save

3. **Configurar como servidor DNS de red**  
   En router/DHCP: configura DNS primario = IP servidor (ej: `192.168.1.100`)  
   O: En cada cliente, configura DNS manual = IP servidor  
   Prueba: `nslookup google.com 192.168.1.100`

4. **Agregar blocklists (anuncios + malware)**  
   Dashboard → Blocklists → Click "+ Add" → URL blocklist (ej: AdGuard, Pi-hole, EasyList) → Enable → automático descarga + actualiza  
   Ejemplo URLs:
   - AdGuard DNS: `https://adguardteam.github.io/AdGuardHome/Blocklists/index.html`
   - EasyList: `https://easylist.to/`
   - uBlock Origin: `https://github.com/uBlockOrigin/uAssets/tree/master/filters`

5. **Configurar forwarders (upstream DNS)**  
   Dashboard → Forwarders → Click "+ Add" forwarder  
   Ej: Cloudflare DoH: `https://1.1.1.1/dns-query`  
   O Google DoT: `tls://8.8.8.8:853` → Enable → usan forwarder para recursive queries

6. **Configurar DoH/DoQ (DNS encriptado)**  
   Dashboard → Settings → Protocols → Enable "DNS-over-HTTPS" (puerto 443) → Enable "DNS-over-QUIC" (puerto 443)  
   Certificados auto (Let's Encrypt si Caddy)  
   Clients usan: `https://dns.tudominio.com/dns-query` (DoH) o `quic://dns.tudominio.com` (DoQ)

7. **Ver query logs y stats**  
   Dashboard → Query Logs → Lista todas queries DNS (original + resultado) → Filter por dominio/IP/tipo  
   Dashboard → Statistics → Gráficos queries/bloques/performance

8. **Crear zona DNS autoritativa custom (opcional)**  
   Dashboard → Zones → Click "+ Add Zone" → Zone name: ej. `internal.local` → Tipo: Primary (autoridad) → Crea registros (A, CNAME, MX, etc) → Queries a `internal.local` resuelven localmente

9. **Configurar clustering (multi-servidor, opcional)**  
   En segundo servidor DNS: Settings → Clustering → Enable Cluster Mode → Primary server: IP primer servidor → Cluster authentication token: generar

10. **Monitorear performance**  
    Dashboard → gráficos real-time (queries/sec, latencia)  
    Ver logs: `docker logs -f technitium-dns`  
    Stats consumo: `docker stats technitium-dns`

## 💡 Casos de uso

- 🏠 **Homelabs** — DNS privado, bloquea anuncios red-wide, cero dependencia ISP
- 🏢 **SMB/Empresa** — DNS corporativo, control granular, clustering multi-sitio
- 🔒 **Privacidad** — No expone queries a ISP, forwarders DoT/DoH/DoQ encriptados
- 🌍 **Hosting DNS** — Zonas autoritativas custom, hosting dominio privado
- 🛡️ **Seguridad** — Bloquea malware/phishing red-wide, filtrado DNS-level
- 🧪 **Lab/desarrollo** — Zonas test (`internal.local`, `dev.local`), conditional forwarders

## 🔒 Acceso remoto seguro

La opción recomendada es **Opción 2 con Caddy**:
- Caddy obtiene certificados TLS válidos (Let's Encrypt) automáticamente
- Termina HTTPS en puerto 443, proxy inverso a puerto 80 del contenedor DNS
- DoH/DoQ funcionan nativamente en `https://dns.tudominio.com/dns-query`
- Web console accesible en `https://dns.tudominio.com`
- Alternativa: nginx + certbot, Traefik, o Cloudflare Tunnel

## 🛠️ Gestión y mantenimiento

```bash
# Ver logs en tiempo real
docker logs -f technitium-dns

# Restart container
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

MIT License — Ver [LICENSE](https://github.com/TechnitiumSoftware/DnsServer/blob/master/LICENSE) en el repositorio oficial.

---

> 📖 **Artículo original:** [Cómo instalar Technitium DNS Server en Docker - Servidor DNS autohospedado, bloqueador anuncios, privacidad](https://genbyte.blogspot.com/2026/08/como-instalar-technitium-dns-server-en.html)  
> 🎥 **Canal YouTube:** [Genbyte](https://www.youtube.com/@genbyte) | 📧 **Newsletter:** [Suscríbete gratis](https://genbyte.blogspot.com/) | ☕ **Ko-fi:** [Invítame a un café](https://ko-fi.com/genbyte)