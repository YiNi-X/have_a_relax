# Requirements

## V1 Must Haves

- Deployable with Docker Compose on a small Linux server.
- Expose a single HTTPS chat site through Caddy.
- Run Open WebUI connected to LiteLLM.
- Configure LiteLLM for DeepSeek official OpenAI-compatible API.
- Keep secrets in `.env`, never committed.
- Include operator documentation for first deployment and daily checks.

## V1 Nice To Haves

- Basic reverse-proxy access protection before the login page.
- Persistent volumes for Open WebUI and PostgreSQL.
- Clear placeholders for adding Alibaba Bailian/Qwen later.

## Out Of Scope

- Paid memberships.
- API resale.
- Custom invitation system.
- Web search.
- File knowledge base.
- Local GPU inference.

