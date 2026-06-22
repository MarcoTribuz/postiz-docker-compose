# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Docker Compose deployment for [Postiz](https://postiz.com) (social media scheduling app). No application source code — only orchestration config. The Postiz app itself ships as a prebuilt image (`ghcr.io/gitroomhq/postiz-app:latest`). Work here is editing Compose files, environment variables, and Temporal dynamic config.

## Commands

```bash
docker compose up                      # start full prod stack (docker-compose.yaml)
docker compose up -d                   # detached
docker compose --profile debug up      # include the spotlight (Sentry) service
docker compose -f docker-compose.dev.yaml up   # dev infra only (no Postiz app)
docker compose down                    # stop
docker compose logs -f postiz          # tail app logs
```

App served at http://localhost:4007 (host `4007` → container `5000`).

## Architecture

Two stacks on two Docker networks (`postiz-network`, `temporal-network`); the `postiz` service joins both to reach Temporal.

**Postiz stack** (`postiz-network`):
- `postiz` — the app (single image runs frontend + backend + workers). Healthcheck must pass before considered up; `start_period` 120s.
- `postiz-postgres` (postgres:17) — app database, volume `postgres-volume`.
- `postiz-redis` (redis:7.2) — queue/cache, volume `postiz-redis-data`.

**Temporal stack** (`temporal-network`) — workflow engine for scheduled posts:
- `temporal` (auto-setup) — depends on its own dedicated `temporal-postgresql` (separate from app DB) and `temporal-elasticsearch` for visibility.
- `temporal-ui` at http://localhost:8080, `temporal-admin-tools` (tctl/CLI).

`depends_on` uses `condition: service_healthy` throughout — startup is gated on healthchecks, so a broken healthcheck blocks dependents.

## Key files

- `docker-compose.yaml` — **production stack**. The file to edit for real deployments.
- `docker-compose.dev.yaml` — dev infra only (Postgres, Redis, pgAdmin :8081, RedisInsight :5540, Temporal). Does **not** run the Postiz app; you run the app separately against this. Header warns it is not kept up to date — don't use for prod.
- `dynamicconfig/development-sql.yaml` — Temporal dynamic config mounted into the `temporal` container (SQL variant is the one wired up).

## Configuration

All config is environment variables on the `postiz` service (see README for the three options: inline, `postiz.env` in `/config`, or `.env`). The full reference is at https://docs.postiz.com/configuration/reference.

When changing real deployments:
- `JWT_SECRET` must be a unique random string per install (placeholder ships in the file).
- `MAIN_URL` / `FRONTEND_URL` / `NEXT_PUBLIC_BACKEND_URL` must match the public address; defaults assume `localhost:4007`.
- Social-platform and Stripe/OpenAI keys are blank by default — fill only what's used.
- `STORAGE_PROVIDER` defaults to `local` (volume `postiz-uploads`); Cloudflare R2 block is commented as the alternative.

## Notes

- Upgrading from an old Postiz version: follow the migration guide (https://docs.postiz.com/installation/migration) before swapping the Compose file.
- `.idea/` (JetBrains) is untracked; `.gitignore` only excludes `.vscode`.
