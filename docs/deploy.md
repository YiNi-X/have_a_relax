# Deploy The Internal Test Version

This guide deploys a free internal AI chat site for a small development team.

## Architecture

```text
browser
  -> Caddy HTTPS reverse proxy
  -> Open WebUI
  -> LiteLLM Proxy
  -> OpenAI GPT-5.5 API

LiteLLM -> PostgreSQL for logs and state
Open WebUI -> Docker volume for users and chats
```

## Server Requirements

- Linux server with 2 CPU cores and 4 GB RAM
- Docker and Docker Compose plugin
- Domain name pointed to the server
- Firewall allows only `80/tcp`, `443/tcp`, and SSH
- OpenAI API key with access to the model ID configured in `deploy/litellm-config.yaml`

Do not run local models on this server.

## Image Pulling In Mainland China

The compose file uses alternate official image sources where possible:

- Open WebUI: `openwebui/open-webui:main`
- LiteLLM: `docker.litellm.ai/berriai/litellm:main-latest`

Open WebUI publishes identical images to GHCR and Docker Hub. LiteLLM's own docs use `docker.litellm.ai/berriai/litellm:main-latest` for Docker deployment. If Docker Hub is still slow on your server, configure an Alibaba Cloud ACR Docker Hub accelerator on the server.

## DNS Setup For `huihang.icu`

In Alibaba Cloud DNS, add this record before starting Caddy:

```text
Record type: A
Host record: chat
Record value: your server public IPv4 address
TTL: default
```

After the record takes effect, `chat.huihang.icu` should resolve to the server. Caddy needs this DNS record plus open `80/tcp` and `443/tcp` ports to issue HTTPS certificates.

## First Deployment

1. Upload this repository to the server.

2. Enter the deploy directory.

   ```bash
   cd deploy
   ```

3. Create your environment file.

   ```bash
   cp .env.example .env
   ```

4. Edit `.env`.

   Minimum required values:

   ```env
   SITE_DOMAIN=chat.huihang.icu
   ACME_EMAIL=3361836542@qq.com
   WEBUI_SECRET_KEY=replace-with-a-long-random-string
   WEBUI_ADMIN_EMAIL=3361836542@qq.com
   WEBUI_ADMIN_PASSWORD=replace-with-a-temporary-admin-password
   LITELLM_MASTER_KEY=sk-replace-with-a-long-random-string
   LITELLM_SALT_KEY=replace-with-a-long-random-string
   POSTGRES_PASSWORD=replace-with-a-long-random-string
   OPENAI_API_KEY=sk-your-openai-key
   ```

   Generate random strings with:

   ```bash
   openssl rand -hex 32
   ```

5. Start services.

   ```bash
   docker compose config
   docker compose up -d
   ```

6. Check service health.

   ```bash
   docker compose ps
   docker compose logs -f caddy open-webui litellm
   ```

   Optional Caddy config validation:

   ```bash
   docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
   ```

7. Open your site.

   ```text
   https://chat.huihang.icu
   ```

8. Log in with the admin account from `.env`.

   Open WebUI can create the initial admin automatically when `WEBUI_ADMIN_EMAIL` and `WEBUI_ADMIN_PASSWORD` are set on a fresh install. After the first login, change the admin password in the UI.

9. Optional: open the LiteLLM admin UI through an SSH tunnel.

   LiteLLM is bound to `127.0.0.1:4000` on the server, not exposed on the public internet. From your local machine:

   ```bash
   ssh -L 4000:127.0.0.1:4000 root@your-server-ip
   ```

   Then open:

   ```text
   http://127.0.0.1:4000/ui
   ```

   Log in with `LITELLM_MASTER_KEY`. For V1, Open WebUI can use the master key directly; for a harder setup later, create a LiteLLM virtual key for Open WebUI and replace `OPENAI_API_KEY`.

## Team Account Options

Recommended V1 mode:

```env
ENABLE_SIGNUP=false
DEFAULT_USER_ROLE=pending
```

The admin creates accounts for team members manually.

Alternative self-registration mode:

```env
ENABLE_SIGNUP=true
DEFAULT_USER_ROLE=pending
```

Members can register, but the admin must approve them before use.

After changing `.env`, restart:

```bash
docker compose up -d
```

## Optional Access Gate

Open WebUI login is enough for many internal tests. If you also want a shared password before the login page, enable Caddy basic auth.

1. Generate a password hash.

   ```bash
   docker run --rm caddy:2.8 caddy hash-password --plaintext "your-access-password"
   ```

2. Put the hash in `.env`.

   ```env
   CADDY_BASIC_AUTH_HASH=$2a$14$...
   ```

3. Uncomment the `basic_auth` block in `Caddyfile`.

4. Reload Caddy.

   ```bash
   docker compose exec caddy caddy reload --config /etc/caddy/Caddyfile
   ```

## Model Configuration

The default LiteLLM config exposes:

- `gpt-5.5`

This name is configured in `deploy/litellm-config.yaml` and routed through LiteLLM as `openai/gpt-5.5` because this internal test deployment is using the model ID available to the current API key. If your OpenAI account exposes a different model alias, update this file and restart LiteLLM:

```bash
docker compose restart litellm
```

## Open WebUI Connection

Open WebUI is configured with:

```env
OPENAI_API_BASE_URL=http://litellm:4000/v1
OPENAI_API_KEY=${LITELLM_MASTER_KEY}
ENABLE_FORWARD_USER_INFO_HEADERS=true
```

That means users chat in Open WebUI, but all model requests pass through LiteLLM for routing and logging.

LiteLLM uses the Open WebUI user ID header for per-user attribution:

```yaml
general_settings:
  user_header_name: X-OpenWebUI-User-Id
```

## Updating

Pull new images and restart:

```bash
docker compose pull
docker compose up -d
```
