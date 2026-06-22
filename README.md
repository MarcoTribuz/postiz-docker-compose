
## Watch the Tutorial for docker-compose install:
[https://m.youtube.com/watch?v=A6CjAmJOWvA&t=5s](https://m.youtube.com/watch?v=A6CjAmJOWvA&t=5s)

## Warning
If you are upgrading from Postiz old version, please make sure you update your docker compose, you can read more here:
https://docs.postiz.com/installation/migration

### Configuration uses environment variables

The docker containers for Postiz are entirely configured with environment variables.

- **Option A** - environment variables in your `docker-compose.yml` file
- **Option B** - environment variables in a `postiz.env` file mounted in `/config` for the Postiz container only
- **Option C** - environment variables in a `.env` file next to your `docker-compose.yml` file (not recommended).

... or a mixture of the above options!

There is a [configuration reference](https://docs.postiz.com/configuration/reference) page with a list
of configuration settings.

Setup:
```
git clone https://github.com/gitroomhq/postiz-docker-compose
```

Then run:
```
docker compose --env-file stack.env up
```

Wait for it to load:

Open your website on https://localhost:4007

## Deploy con Portainer (Git repository)

Lo stack è parametrizzato con variabili d'ambiente (vedi `stack.env`). Per deployarlo in Portainer:

1. **Stacks → Add stack → Repository**.
2. Repository URL: questo repo. Compose path: `docker-compose.yaml`.
3. Nella sezione **Environment variables** incolla le variabili di `stack.env` e compila i valori reali:
   - `MAIN_URL` / `FRONTEND_URL` / `NEXT_PUBLIC_BACKEND_URL` → il tuo dominio pubblico (NO slash finale; `NEXT_PUBLIC_BACKEND_URL` finisce con `/api`).
   - `JWT_SECRET` → stringa random unica (`openssl rand -hex 32`).
   - `POSTIZ_DB_USER` / `POSTIZ_DB_PASSWORD` / `POSTIZ_DB_NAME` → credenziali DB Postiz.
   - `TEMPORAL_DB_USER` / `TEMPORAL_DB_PASSWORD` → credenziali DB Temporal.
4. **Deploy the stack**.

> **Nota:** con un dominio HTTPS serve un reverse proxy (Traefik / Nginx Proxy Manager) davanti alla porta `4007`, altrimenti login e OAuth non funzionano. Lo stack è pesante (Elasticsearch + 2 Postgres + Temporal): assicurati di avere RAM sufficiente.
