# Yana's Beauty Clinic

A responsive static website for a professional eyelash extension and eyebrow shaping business, containerized with Docker and served via Nginx. Deployed through a CI/CD pipeline with GitHub Actions and Cloudflare Tunnel.

## Architecture

```
Client → Cloudflare Tunnel → Docker (Nginx) → Static HTML/CSS/JS
```

```
├── index.html              # Homepage
├── services.html           # Services & pricing
├── gallery.html            # Portfolio gallery
├── contact.html            # Contact & booking
├── css/style.css           # Styles (mobile-first, responsive)
├── js/main.js              # Interactivity
├── nginx/nginx.conf        # Nginx config (gzip, caching, security headers)
├── Dockerfile              # Multi-stage Nginx Alpine container
├── docker-compose.yml      # Container orchestration
└── .github/workflows/
    └── ci-cd.yml           # CI/CD pipeline (lint → build → deploy)
```

## Tech Stack

| Layer          | Technology                     |
|----------------|--------------------------------|
| Frontend       | HTML5, CSS3, JavaScript        |
| Web Server     | Nginx 1.27 (Alpine)            |
| Containerization | Docker, Docker Compose       |
| CI/CD          | GitHub Actions                 |
| Reverse Proxy  | Cloudflare Tunnel              |
| DNS/CDN        | Cloudflare                     |

## Quick Start

### Run locally
```bash
docker compose up -d
```
Site available at `http://localhost:8090`

### Build only
```bash
docker build -t yanas-beauty-clinic .
docker run -p 8090:80 yanas-beauty-clinic
```

## CI/CD Pipeline

The GitHub Actions pipeline runs on every push to `main`:

1. **Lint** - Validates HTML/CSS
2. **Build** - Builds Docker image with caching, runs smoke tests against all pages
3. **Deploy** - SSHs into the server, pulls latest code, rebuilds and restarts the container

## Cloudflare Tunnel Setup

Add the service to your existing `config.yml`:

```yaml
ingress:
  - hostname: beauty.yourdomain.com
    service: http://localhost:8090
  # ... existing services
  - service: http_status:404
```

Then create a CNAME record in Cloudflare DNS pointing `beauty` to your tunnel.

## Nginx Features

- Gzip compression for text assets
- Static asset caching (7 days CSS/JS, 30 days images)
- Security headers (X-Frame-Options, X-Content-Type-Options, XSS Protection)
- Clean URLs (`.html` extension optional)
- Health check endpoint

## Configuration

Before deploying, update these placeholders:

| Placeholder | File(s) | Replace With |
|-------------|---------|--------------|
| `YOUR_BOOKING_PAGE` | All HTML files | Your Calmark booking URL |
| `+972-XX-XXX-XXXX` | contact.html | Business phone number |
| `info@yanasbeauty.com` | contact.html | Business email |
| `Your address here` | contact.html | Business address |

### GitHub Secrets (for CI/CD)

| Secret | Description |
|--------|-------------|
| `SSH_HOST` | Server IP or hostname |
| `SSH_USER` | SSH username |
| `SSH_PRIVATE_KEY` | SSH private key for deployment |
