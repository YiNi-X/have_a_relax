# Requirements

## V1 Must Haves

- Deployable with Docker Compose on a small Linux server.
- Expose a single HTTPS chat site through Caddy.
- Run Open WebUI connected to LiteLLM.
- Configure LiteLLM for OpenAI GPT-5.5 API access.
- Keep secrets in `.env`, never committed.
- Include operator documentation for first deployment and daily checks.

## V1 Nice To Haves

- Basic reverse-proxy access protection before the login page.
- Persistent volumes for Open WebUI and PostgreSQL.
- Clear provider configuration that can be extended to other models later.

## Out Of Scope

- Paid memberships.
- API resale.
- Custom invitation system.
- Web search.
- File knowledge base.
- Local GPU inference.
