# Operations

## Daily Checks During The Trial

Run from the `deploy` directory:

```bash
docker compose ps
docker compose logs --tail=100 litellm
docker compose logs --tail=100 open-webui
```

Watch for:

- Repeated upstream API failures
- Very long responses
- Unexpected high usage by one user
- Server memory pressure

## Cost Review

LiteLLM stores usage data in PostgreSQL. For quick checks, open the LiteLLM UI/API if enabled in your deployment, or inspect logs first.

V1 review questions:

- How many requests per user per day?
- Which model is used most?
- What is the estimated cost per active user?
- Are reasoning-model requests materially more expensive?
- Are users asking for files, search, or team knowledge features?

## Backups

Create a backup directory on the server:

```bash
mkdir -p ../backups
```

Back up PostgreSQL:

```bash
docker compose exec -T postgres pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB" > ../backups/litellm-$(date +%F).sql
```

Back up Docker volumes before major changes:

```bash
docker run --rm \
  -v deploy_open_webui_data:/data:ro \
  -v "$(pwd)/../backups:/backup" \
  alpine tar czf /backup/open-webui-$(date +%F).tgz -C /data .
```

## Restarting Services

```bash
docker compose restart open-webui
docker compose restart litellm
docker compose restart caddy
```

## Troubleshooting

### Site Does Not Open

Check DNS and Caddy logs:

```bash
dig +short chat.huihang.icu
docker compose logs --tail=100 caddy
```

### Login Works But Models Fail

Check LiteLLM logs and your OpenAI key:

```bash
docker compose logs --tail=200 litellm
```

Common causes:

- `OPENAI_API_KEY` is wrong or empty.
- The upstream account has no balance.
- The model name in `litellm-config.yaml` is no longer supported.
- Open WebUI is not using the LiteLLM base URL.

### Server Feels Slow

For a 2-core/4-GB server:

- Keep concurrent testers small.
- Avoid file knowledge base and local models.
- Use the normal chat model as default.
- Restrict reasoning model access during the first week.

## End Of Month Review

Export these findings into a short note:

- Active users
- Total requests
- Total estimated API cost
- Highest-cost users
- Most common use cases
- Top missing features
- Decision: keep config-only deployment, customize frontend, or build product backend
