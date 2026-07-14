<div align="center">
  <h1>🏓 ft_transcendence</h1>
  <p><b><i>We are Team Babylonians.</i></b></p>
</div>

<div align="center">

![Language](https://img.shields.io/badge/language-TypeScript-3178c6.svg)
![Backend](https://img.shields.io/badge/backend-Fastify-000000.svg)
![Frontend](https://img.shields.io/badge/graphics-Babylon.js-e0684b.svg)
![Database](https://img.shields.io/badge/database-SQLite-003b57.svg)
![Infra](https://img.shields.io/badge/infra-Docker-2496ed.svg)
![Monitoring](https://img.shields.io/badge/monitoring-Prometheus%20%2B%20Grafana-e6522c.svg)
![School](https://img.shields.io/badge/42-Heilbronn-black.svg)
![Grade](https://img.shields.io/badge/grade-125%25-brightgreen.svg)

</div>

---

## Table of Contents

- [About](#about)
- [Tech Stack](#tech-stack)
- [Modules](#modules)
- [Architecture](#architecture)
- [DevOps & Monitoring](#devops--monitoring)
- [Setup & Usage](#setup--usage)
- [Makefile Reference](#makefile-reference)
- [Team](#team)

---

## About

**ft_transcendence** is the final project of the 42 Common Core — a full-stack multiplayer Pong game built from scratch. The team implemented a faithful recreation of the original 1972 Pong with modern features: 3D rendering, online multiplayer, live chat, an AI opponent, a full user management system with 2FA, and a production-grade DevOps monitoring stack.

The project is scored via optional Major and Minor modules on top of the mandatory base. With **10 major + 3 minor modules**, the team achieved the **maximum grade of 125%**.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | TypeScript, Vite, Tailwind CSS, Babylon.js |
| Backend | Node.js, Fastify |
| Database | SQLite |
| Auth | JWT, Cookie sessions, Google Authenticator (TOTP) |
| Networking | WebSockets, Nginx reverse proxy, HTTPS (self-signed SSL) |
| Monitoring | Prometheus, Grafana, Alertmanager |
| Infra | Docker, Docker Compose |

---

## Modules

### Major Modules (×10)

| Module | Description |
|---|---|
| **Backend Framework** | REST API and route handling built with Fastify |
| **Remote Authentication** | Google Authenticator support as a third-party auth provider |
| **Remote Players** | Online multiplayer via WebSockets with real-time synchronization |
| **Live Chat** | In-app chat with messaging, game invites, and user stat previews |
| **User Dashboard & Statistics** | Player profiles, match history, friend requests, profile pictures, and a blocking system |
| **AI Opponent** | Play against an AI with three difficulty settings |
| **2FA & JWTs** | Session management via HTTP-only cookies and JWTs; optional 2FA via QR-linked Authenticator app |
| **3D Graphics** | Game rendered in 3D using Babylon.js (the origin of the team name) |
| **Server-Side Pong API** | Game state (ball physics, paddle positions) calculated server-side and broadcast to clients |
| **DevOps Monitoring** | Prometheus + Grafana + Alertmanager stack with exporters, custom dashboards, alert rules, and Slack integration |

### Minor Modules (×3)

| Module | Description |
|---|---|
| **Frontend Framework** | Tailwind CSS for styling, Vite for fast HMR and bundling |
| **Database** | SQLite for persistent user, game, and social data |
| **Multi-Browser** | Tested across multiple browsers (Firefox-native) |

---

## Architecture

```
                        ┌─────────────────────────────────────┐
                        │           Docker Network             │
                        │                                      │
  Browser ──HTTPS──▶  nginx (8443)                            │
                        │  ├──▶  frontend (5173)  [Vite/TS]   │
                        │  └──▶  backend  (3000)  [Fastify]   │
                        │           └──▶  SQLite              │
                        │                                      │
                        │  ── Monitoring Stack ──              │
                        │  prometheus    (9090)                │
                        │  grafana       (3001)                │
                        │  alertmanager  (9093)  ──▶  Slack    │
                        │  nginx-exporter(9113)                │
                        └─────────────────────────────────────┘
```

---

## DevOps & Monitoring

> Implemented by [bebuber](https://github.com/Bebuber)

The monitoring stack runs as four additional Docker containers fully integrated into the compose network. It observes the nginx, backend, Grafana, Alertmanager, and Prometheus services themselves.

### Prometheus

Scrapes metrics from all services on a 15-second interval. Data is retained for 15 days via persistent volume. Scrape targets:

| Job | Target | What it tracks |
|---|---|---|
| `prometheus` | localhost:9090 | Prometheus self-metrics |
| `nginx` | nginx-exporter:9113 | HTTP request rates, connections |
| `backend` | backend:3000 | Fastify route-level metrics (via `fastify-metrics`) |
| `grafana` | grafana:3000 | Grafana internals |
| `alertmanager` | alertmanager:9093 | Alert pipeline metrics |

### Grafana

Auto-provisioned via config files — no manual dashboard setup needed on first boot. Dashboards include:

- **Targets Up** — live service health across all jobs
- **Backend 5xx Errors (5m)** — rolling error rate per route
- **HTTP request throughput** per handler

### Alertmanager

Fires alerts to a **custom Slack channel** via webhook. The Slack webhook URL is injected at runtime via a custom entrypoint script that performs environment variable substitution before Alertmanager starts — avoiding secrets in committed config files.

Current alert rules:

| Alert | Condition | Severity |
|---|---|---|
| `BackendDown` | Backend job unreachable for 20s | critical |

### nginx-exporter

A dedicated `nginx/nginx-prometheus-exporter` sidecar container scrapes Nginx's `stub_status` endpoint and exposes it in Prometheus format, bridging Nginx's native metrics into the monitoring stack.

---

## Setup & Usage

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose (v2+)
- A Slack webhook URL (optional — only required for Alertmanager notifications)

### 1. Clone

```bash
git clone https://github.com/Bebuber/Transcendence.git
cd Transcendence
```

### 2. Create `.env`

Create a `.env` file in the project root:

```env
JWT_SECRET=your_jwt_secret_key

GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=your_grafana_password
GF_AUTH_ANONYMOUS_ENABLED=false
GF_USERS_ALLOW_SIGN_UP=true

SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...
```

### 3. Build & Run

```bash
make build
```

### 4. Open

Navigate to `https://localhost:8443/` and accept the self-signed certificate warning.

Monitoring dashboards are available at `http://localhost:3001/` (Grafana).

---

## Makefile Reference

| Command | Description |
|---|---|
| `make build` | Build all Docker images and start all services |
| `make up` | Start already-built services |
| `make down` | Stop all services |
| `make re` | Stop and rebuild all services |
| `make prune` | Full teardown including volumes and database |
| `make reset` | Prune then rebuild from scratch |
| `make help` | Show available commands |

---

## Team

| Name | GitHub | Contributions |
|---|---|---|
| Jacob Graf | [@jgraf](https://github.com/jgraf-42) | Game design, game logic, 3D rendering, AI opponent, gamemodes, playtesting |
| Tobias Keil | [@kixikCodes](https://github.com/Cimex404) | Backend architecture, WebSockets, remote players, database, live chat, dashboard, friend/block system |
| Noel Monzon | [@N03l-MG](https://github.com/N03l-MG) | Cybersecurity, auth & authorization, 2FA, JWT/cookie management, API protection, UI design, sound design, documentation |
| Betül Büber | [@bebuber](https://github.com/Bebuber) | DevOps monitoring stack (Prometheus, Grafana, Alertmanager, Slack integration, nginx-exporter, backend metrics) |

---

*42 Heilbronn — Group Project — 125%*
