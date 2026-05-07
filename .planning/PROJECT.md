# Internal AI Chat Testbed

## Purpose

Build a free internal testing deployment for the development team to evaluate real usage, model quality, and API cost before deciding whether to commercialize.

## V1 Scope

- Open WebUI provides the chat interface, login, conversation history, and basic user management.
- LiteLLM provides an OpenAI-compatible model gateway and usage/cost logging.
- PostgreSQL stores LiteLLM state and logs.
- Caddy provides HTTPS reverse proxy and optional basic access protection.
- OpenAI GPT-5.5 is the first upstream model provider.

## Constraints

- Target server: China mainland lightweight server, 2 CPU cores, 4 GB RAM.
- No local model hosting.
- No payment, membership, invite-code product logic, or custom frontend in V1.
- Official API keys only.
