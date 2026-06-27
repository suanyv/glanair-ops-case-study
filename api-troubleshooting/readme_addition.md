## API Troubleshooting Fundamentals

Demonstrating the ability to authenticate, query, and diagnose API failures using Postman against the GlanAir Salesforce org.

- **OAuth 2.0 authentication** (Authorization Code with PKCE) — the same handshake used by Shopify, HubSpot, Stripe, and most B2B SaaS integrations
- **Full CRUD cycle** — GET, POST, PATCH, DELETE against live case and contact data
- **Error diagnosis** — deliberately broken requests reproducing 401, 400, and 404 failures with documented root causes and fixes
- **Rate limit observability** — reading `Sforce-Limit-Info` headers to monitor API quota

Full walkthrough with screenshots: [`api-troubleshooting/api-troubleshooting.md`](api-troubleshooting/api-troubleshooting.md)
