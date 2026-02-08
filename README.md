# Yana's Beauty Clinic

A responsive static website for a professional eyelash extension and eyebrow shaping business. Features a full DevOps stack: Docker/Nginx containerization, Prometheus + Grafana monitoring, CI/CD via GitHub Actions, Cloudflare Tunnel with automatic failover to Cloudflare Pages.

## Architecture

```
                         ┌──────────────────┐
                         │    Cloudflare     │
                         │   Health Check    │
                         └────────┬─────────┘
                                  │
                  ┌───────────────┼───────────────┐
                  ▼                               ▼
       ┌───────────────────┐           ┌───────────────────┐
       │  Primary Origin   │           │ Failover Origin   │
       │  Home Server      │           │ Cloudflare Pages  │
       │  (Tunnel)         │           │ (Auto-deployed)   │
       └────────┬──────────┘           └───────────────────┘
                │
    ┌───────────┼───────────┐
    ▼           ▼           ▼
┌────────┐ ┌────────┐ ┌──────────┐
│ Nginx  │ │Promethe│ │ Grafana  │
│        │ │  us    │ │          │
└────────┘ └────────┘ └──────────┘
```

```
├── index.html                          # Homepage (Hebrew RTL)
├── services.html                       # Services & pricing
├── gallery.html                        # Portfolio gallery
├── contact.html                        # Contact & booking
├── css/style.css                       # Styles (mobile-first, RTL)
├── js/main.js                          # Interactivity
├── nginx/nginx.conf                    # Nginx config (gzip, caching, security, metrics)
├── Dockerfile                          # Nginx Alpine container
├── docker-compose.yml                  # Full stack orchestration
├── monitoring/
│   ├── prometheus/prometheus.yml        # Prometheus scrape config
│   └── grafana/
│       ├── provisioning/               # Auto-configured datasources & dashboards
│       └── dashboards/nginx.json       # Pre-built Nginx dashboard
└── .github/workflows/
    └── ci-cd.yml                       # CI/CD: lint → build → deploy (primary + failover)
```

## Tech Stack

| Layer          | Technology                          |
|----------------|-------------------------------------|
| Frontend       | HTML5, CSS3, JavaScript (Hebrew RTL)|
| Web Server     | Nginx 1.27 (Alpine)                 |
| Containerization | Docker, Docker Compose            |
| Monitoring     | Prometheus + Grafana                |
| CI/CD          | GitHub Actions                      |
| Primary Host   | Cloudflare Tunnel (home server)     |
| Failover Host  | Cloudflare Pages (auto-deployed)    |
| DNS/CDN/WAF    | Cloudflare                          |

## Quick Start

### Run full stack
```bash
docker compose up -d
```
| Service  | URL                      |
|----------|--------------------------|
| Website  | http://localhost:8090     |
| Grafana  | http://localhost:3000     |
| Prometheus | http://localhost:9090   |

Grafana default login: `admin` / `admin`

### Run website only
```bash
docker build -t yanas-beauty-clinic .
docker run -p 8090:80 yanas-beauty-clinic
```

## CI/CD Pipeline

The GitHub Actions pipeline runs on every push to `main`:

```
Push to main
    │
    ▼
┌─────────┐     ┌─────────────┐     ┌──────────────────┐
│  Lint   │────▶│  Build &    │────▶│  Deploy Primary  │
│  HTML   │     │  Test       │     │  (SSH → Docker)  │
└─────────┘     └─────────────┘     └──────────────────┘
                                             │
                                    ┌──────────────────┐
                                    │ Deploy Failover  │
                                    │ (Cloudflare Pages)│
                                    └──────────────────┘
```

1. **Lint** - Validates HTML/CSS
2. **Build** - Builds Docker image with caching, runs smoke tests against all pages
3. **Deploy Primary** - SSHs into home server, pulls latest code, rebuilds containers
4. **Deploy Failover** - Deploys static site to Cloudflare Pages (parallel with primary)

## Monitoring

Prometheus scrapes Nginx metrics via `nginx-prometheus-exporter`. Grafana provides a pre-configured dashboard with:

- Active connections (real-time graph)
- Requests per second
- Connection states (reading/writing/waiting)
- Nginx UP/DOWN status indicator
- Total request count

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

## Failover Strategy

When the home server goes down (power outage), Cloudflare detects the tunnel is unhealthy and can route traffic to the Cloudflare Pages deployment, which is always up-to-date via CI/CD.

## Configuration

Update this placeholder across all HTML files:

| Placeholder | Replace With |
|-------------|--------------|
| `YOUR_BOOKING_PAGE` | Your Calmark booking URL |

### GitHub Secrets (for CI/CD)

| Secret | Description |
|--------|-------------|
| `SSH_HOST` | Home server IP or hostname |
| `SSH_USER` | SSH username |
| `SSH_PRIVATE_KEY` | SSH private key for deployment |
| `CLOUDFLARE_API_TOKEN` | Cloudflare API token (Pages deploy) |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID |
