# monitoring-stack

A production-friendly Docker Compose monitoring stack with **Grafana** and **Loki**. This repository provides secure-by-default configuration, persistent storage, and automated Grafana datasource provisioning.

## Overview

This stack is designed for quickly deploying log monitoring with minimal manual setup:

- **Loki** ingests and stores logs on local filesystem storage.
- **Grafana** provides dashboards and visualization.
- **Grafana provisioning** automatically registers Loki as a datasource.
- **GitHub OAuth** controls Grafana access through your GitHub organization.

## Architecture

- `grafana` service
  - Exposed on `http://localhost:3000`
  - Uses mounted `grafana/grafana.ini` for configuration
  - Uses mounted provisioning directory for datasource auto-loading
  - Persists data in Docker volume `grafana_data`
- `loki` service
  - Exposed on `http://localhost:3100`
  - Uses mounted `loki/config.yml`
  - Persists data in Docker volume `loki_data`
- Shared custom Docker network: `monitoring`

## Repository structure

```text
monitoring-stack/
├── docker-compose.yml
├── .env.example
├── README.md
├── grafana/
│   ├── grafana.ini
│   └── provisioning/
│       └── datasources/
│           └── datasource.yml
├── loki/
│   └── config.yml
└── scripts/
    ├── start.sh
    └── stop.sh
```

## Setup

1. Copy the environment template:

   ```bash
   cp .env.example .env
   ```

2. Edit `.env` with secure credentials and GitHub OAuth values.

3. Start the stack:

   ```bash
   ./scripts/start.sh
   ```

4. Open Grafana:

   - URL: `http://localhost:3000`

## GitHub OAuth setup

1. In GitHub, create an OAuth App.
2. Set:
   - **Homepage URL**: your Grafana URL (for example `https://grafana.example.com`)
   - **Authorization callback URL**: `<GRAFANA_ROOT_URL>/login/github`
3. Put credentials in `.env`:
   - `GITHUB_CLIENT_ID`
   - `GITHUB_CLIENT_SECRET`
4. Restrict access with `GITHUB_ALLOWED_ORG`.
5. Restart Grafana:

   ```bash
   docker compose restart grafana
   ```

## Docker commands

- Start services:

  ```bash
  docker compose up -d
  ```

- Stop services:

  ```bash
  docker compose down
  ```

- Check status:

  ```bash
  docker compose ps
  ```

- View logs:

  ```bash
  docker compose logs -f
  ```

## Example `.env`

```env
GRAFANA_ADMIN_USER=admin
GRAFANA_ADMIN_PASSWORD=change-me
GRAFANA_ROOT_URL=http://localhost:3000
GITHUB_CLIENT_ID=replace-with-client-id
GITHUB_CLIENT_SECRET=replace-with-client-secret
GITHUB_ALLOWED_ORG=your-github-org
```

## Troubleshooting

- **Grafana OAuth login fails**
  - Confirm callback URL is exactly `<GRAFANA_ROOT_URL>/login/github`.
  - Confirm `GITHUB_ALLOWED_ORG` matches your organization.
- **Datasource not visible in Grafana**
  - Check provisioning files are mounted:
    ```bash
    docker compose exec grafana ls -R /etc/grafana/provisioning
    ```
  - Restart Grafana after provisioning changes.
- **Containers unhealthy**
  - Inspect health and logs:
    ```bash
    docker compose ps
    docker compose logs loki grafana
    ```

## Customize Grafana

- Update `grafana/grafana.ini` for server/security/auth settings.
- Add dashboards/provisioning under `grafana/provisioning/`.
- Restart Grafana after config changes.

## Customize Loki

- Edit `loki/config.yml` for retention, limits, and ingestion settings.
- Keep filesystem paths under `/loki` aligned with mounted volume `loki_data`.
- Restart Loki after config updates:

  ```bash
  docker compose restart loki
  ```
