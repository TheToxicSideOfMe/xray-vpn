# xray-vpn

A self-hosted censorship circumvention server using Xray + Nginx + Docker Compose.
Supports VLESS over WebSocket and XHTTP with TLS via Let's Encrypt.

Built to help people in heavily censored regions (Iran, China, Russia, etc.)
access the open internet.

## Stack

- **Xray** — VLESS protocol with WebSocket/XHTTP transport
- **Nginx** — Reverse proxy + TLS termination
- **Certbot** — Automatic Let's Encrypt certificates
- **Docker Compose** — Container orchestration

## Project Structure
xray-vpn/
├── docker-compose.yml
├── config/
│   └── config.json        # Xray config
├── nginx/
│   └── conf.d/
│       └── xray.conf      # Nginx reverse proxy + TLS
└── certbot/
├── conf/              # Let's Encrypt certs (auto-generated)
└── www/               # ACME challenge webroot (auto-generated)

## Prerequisites

- A VPS or server with a public IP
- A domain name pointing to your server's IP (A record)
- Docker + Docker Compose installed
- Ports 80 and 443 open

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/yourusername/xray-vpn.git
cd xray-vpn
```

### 2. Generate a UUID

```bash
cat /proc/sys/kernel/random/uuid
```

### 3. Configure Xray

Edit `config/config.json` and replace `YOUR-UUID-HERE` with your UUID.

### 4. Configure Nginx

Edit `nginx/conf.d/xray.conf` and replace `your-domain.com` with your domain.

### 5. Get TLS certificate

```bash
# Start Nginx temporarily for ACME challenge
docker compose up -d nginx

# Issue certificate
docker compose run --rm certbot certonly \
  --webroot \
  --webroot-path=/var/www/certbot \
  --email your@email.com \
  --agree-tos \
  --no-eff-email \
  -d your-domain.com
```

### 6. Restore full Nginx config and start everything

```bash
docker compose up -d
```

### 7. Verify

```bash
docker compose ps        # all containers should be Up
docker compose logs xray # should show "Xray started"
curl https://your-domain.com  # should return 404
```

## Client Setup

Generate your VLESS connection link:
vless://YOUR-UUID-HERE@your-domain.com:443?encryption=none&security=tls&type=xhttp&path=%2Fray&sni=your-domain.com#My-VPN

### Android
Install **v2rayNG** from [GitHub releases](https://github.com/2dust/v2rayNG/releases),
tap `+` → Import from clipboard → paste the link → connect.

### iOS
Use **FoXray** or **Shadowrocket** from the App Store.

### Windows
Use **v2rayN** from [GitHub releases](https://github.com/2dust/v2rayN/releases).

## Notes

- Your UUID is your password — keep it private
- Let's Encrypt certs expire after 90 days — renew with:
```bash
  docker compose run --rm certbot renew
  docker compose restart nginx
```
- For dynamic home IPs, consider a DDNS service like DuckDNS
- For maximum stealth against DPI, consider upgrading to VLESS+REALITY on a VPS

## Why

Iran, Russia, China and other countries actively block standard VPN protocols
using Deep Packet Inspection (DPI). VLESS over XHTTP disguises traffic as
normal HTTPS, making it significantly harder to detect and block.

## License

MIT