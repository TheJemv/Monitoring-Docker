# Monitoring — Server Stack

This is the whole server side of the
[Monitoring](https://github.com/TheJemv/Monitoring) app: a docker-compose
stack with Prometheus, node-exporter, cAdvisor, and Portainer, each exposed
on your network so you can reach them directly (through your own router
port-forwarding and VPN setup).

| File | What it's for |
| --- | --- |
| [`docker-compose.yml`](docker-compose.yml) | Prometheus, node-exporter, cAdvisor, and Portainer |
| [`prometheus.yml`](prometheus.yml) | Prometheus scrape config — already points at the other containers by their compose service name |

## Requirements

- A machine with [Docker](https://docs.docker.com/engine/install/) and the
  `docker compose` plugin — a home server, a mini PC, a VPS, anywhere Docker
  runs.

## 1. Get this onto your server

```bash
git clone https://github.com/TheJemv/Monitoring-Docker.git
cd Monitoring-Docker
```

## 2. Start the stack

```bash
docker compose up -d
```

This brings up, each bound to the host on the port shown:

- **Prometheus** on port `9090` — collects and stores the metrics.
- **node-exporter** on port `9100` — host-level metrics (CPU, RAM, disks, temperature).
- **cAdvisor** on port `8080` — per-container CPU/RAM metrics.
- **Portainer** on ports `9000` (HTTP) and `9443` (HTTPS) — the Docker API the app talks to for the Docker tab.

Check everything's up with:

```bash
docker compose ps
```

### Matching it to your hardware

Adjust [`prometheus.yml`](prometheus.yml) if you rename services or add more
scrape targets. In the [Monitoring app's source](https://github.com/TheJemv/Monitoring),
check `SSD_MOUNTPOINT`/`HDD_MOUNTPOINT` in
[`src/screens/prometheus/hooks/use-prometheus-overview.ts`](https://github.com/TheJemv/Monitoring/blob/main/src/screens/prometheus/hooks/use-prometheus-overview.ts)
and `MEMORY_TOTAL_GIB` in
[`src/screens/prometheus/prometheus.constants.ts`](https://github.com/TheJemv/Monitoring/blob/main/src/screens/prometheus/prometheus.constants.ts)
to match your server.

## Exposing it to the internet

Each service above binds directly to the host on its own port (see the list
above). To reach them while you're away from home, forward those ports on
your router to this machine and connect in over your own VPN — no tunnel or
reverse proxy container is needed for that.

## Setting up Portainer's access token

The app's Docker tab talks to the real Docker Engine API through Portainer,
so it needs an access token:

1. Open Portainer → your user (top right) → **Access tokens**.
2. Create a new token and copy it — Portainer only shows it once.
3. Confirm your Portainer *environment* id with `GET /api/endpoints`
   (Portainer usually calls the local one `1` on a fresh install, but check
   — it isn't always). If yours isn't `1`, update `PORTAINER_ENDPOINT_ID` in
   [`src/api/portainer/constants.ts`](https://github.com/TheJemv/Monitoring/blob/main/src/api/portainer/constants.ts)
   in the app's source.

## Next: configure the app

Once the stack is running, open the Monitoring app → **Configuration** tab,
and enter your server host, service URLs, and the Portainer URL/token from
above. See the [main repo](https://github.com/TheJemv/Monitoring) for more
about the app itself.
