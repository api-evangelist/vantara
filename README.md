# Milana (formerly Vantara)

Milana — https://getmilana.ai (vantara.ai redirects here) — is "the AI product engineer," a Homebrew-backed startup that records user sessions with a lightweight open-source browser SDK ([milana-js](https://www.npmjs.com/package/milana-js)) and uses vision AI to watch the replays, understand what users came to do and whether they succeeded, and surface friction traditional analytics miss.

Programmatic surfaces captured in this repo:

- **MCP server** — official hosted remote server at `https://app.getmilana.ai/mcp` (streamable HTTP, OAuth via Clerk) — see `mcp/`
- **Browser SDK** — `milana-js` npm package + CDN script tag, MIT, open source at [VantaraAI/milana-sdk](https://github.com/VantaraAI/milana-sdk) — see `packages/`, `changelog/`, `skills/`
- **llms.txt** — published on both getmilana.ai and docs.getmilana.ai — see `llms/`
- **Well-known** — security.txt + RFC 8414 OAuth authorization-server metadata on app.getmilana.ai — see `well-known/`, `scopes/`, `authentication/`, `security/`

No public REST API is documented; there is no OpenAPI to harvest.
