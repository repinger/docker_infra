# docker_infra

Self-hosted server infrastructure, managed with Docker Compose. A single compose file includes per-service compose projects and wires them together on shared IPv4/IPv6 bridge networks behind Traefik.

## Services

| Directory | Service | Purpose |
|---|---|---|
| `proxy/` | Traefik | Reverse proxy, TLS termination, OIDC, crowdsec & captcha middleware |
| `acme.sh/` | acme.sh | ACME certificates |
| `backrest/` | Backrest | Backup UI (restic frontend) |
| `crowdsec/` | CrowdSec | Intrusion prevention / detection, LAPI |
| `dns/` | AdGuard Home | DNS filtering, OIDC-protected |
| `email/` | Stalwart | Mail server |
| `monitoring/` | Grafana + Prometheus | Metrics collection and dashboards |
| `pocket-id/` | Pocket-ID | OIDC identity provider used for auth across services |
| `endlessh/` | endlessh | SSH tarpit |
| `web/` | web-blackhole | HTTP/3 nginx sink for unrouted traffic |

## Layout

```
docker-compose.yml      # Root compose: includes all service projects, defines networks
.env.example            # Template for .env (per-service config values)
.env                    # Actual configuration (gitignored)
<service>/              # Each service has its own docker-compose.yml + config
proxy/config/           # Traefik static config and dynamic conf.d/ middleware
```

The root `docker-compose.yml` uses compose `include` to pull in every service project, so the whole stack is managed from one file:

```bash
docker compose up -d
```

## Networking

Two shared bridge networks, defined at the root and consumed by all services:

- `infra-1` — IPv4 (`DOCKER_NET_IPV4_SUBNET`), NAT via bridge
- `infra_v6-1` — public IPv6 (`DOCKER_NET_IPV6_SUBNET`), **no NAT** (masquerade disabled)

Subnets and public/private addresses are set in `.env` via `DOCKER_NET_IPV4_SUBNET`, `DOCKER_NET_IPV6_SUBNET`, `PUBLIC_IP_*`, and `PRIVATE_IPV4*` / `PRIVATE_IPV6*`.

## Configuration

1. Copy the template and fill in values:

   ```bash
   cp .env.example .env
   ```

2. Per-service settings live in `.env` under `# --- <Service> ---` sections: domains, OIDC client IDs/secrets, cert paths, and tokens (e.g. `TRAEFIK_CROWDSEC_LAPI_KEY`, `ADGUARDHOME_OIDC_*`, `POCKET_ID_ENC_KEY`).

3. Traefik dynamic config lives in `proxy/config/conf.d/` (TLS options, access lists, crowdsec/captcha/OIDC middleware).

`USER_ID` / `GROUP_ID` in `.env` set the UID/GID used by containers.

## License

Apache License 2.0 — see [LICENSE.md](LICENSE.md).