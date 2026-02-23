# Security & Data Privacy

This project was built with strict security guidelines, making it suitable for production environments, particularly in FinTech or data-sensitive sectors.

## 1. Secrets Management
| Security Measure  | Implementation |
| :--- | :--- |
| **No Hardcoded Credentials** | All API keys (Gemini, Supabase, Telegram), database passwords, and provider tokens are managed via **n8n Credentials Vault**. |
| **Environment Variables** | The `.env` file is heavily `.gitignore`d. Only a sanitized `.env.example` is provided in this repository to illustrate the required configuration without exposing actual secrets. |
| **Workflow Export** | All exported JSON workflows in this repository have been stripped of credential bindings. |
| **Token Lifecycle, Rotation & Least Privilege** | All third-party connections utilize n8n's abstracted credentials vault. For OAuth2 flows (e.g., Google Sheets), n8n handles automatic token refreshes to prevent `401 Unauthorized` errors. Both OAuth scopes and static API keys (e.g., Supabase, Telegram) strictly follow the Principle of Least Privilege. Key rotation is managed externally without requiring application downtime. |

## 2. PII Data Masking (Data Privacy)
Before any user message is sent to external LLM providers (e.g., Google Gemini) or logged into the analytics database, the system executes a **PII Guard Node (Regex/Code)**.
- **Phone Numbers:** Masked to show only the last 4 digits (e.g., `*******5678`) in plain-text logs. The original number is preserved *only* for the execution context to route the reply.
- **Credit Card / Sensitive Numbers:** Regex patterns detect 13-16 digit strings and redact them to `[CARD_REDACTED]` before sending the payload to the AI agent.

## 3. Human-in-the-Loop & Guardrails
- **Kill-Switch:** Authorized human agents can issue a `/stop` command via WhatsApp. This flips the `is_bot_active` flag to `FALSE` in the CRM, physically severing the AI's ability to reply to that specific user.
- **System Guardrails:** The LLM is strictly prompted **never** to provide financial advice, make guarantees, or invent pricing. If a query falls outside the provided RAG context, it is forced to return a `MANAGER` escalation code.

## 4. Webhook Security (Recommended for Prod)
While the mock provider uses open webhooks for demonstration, the production setup implies:
- **IP Whitelisting:** n8n ingress points should only accept requests from the specific WhatsApp provider's IP range.
- **Authentication:** Bearer tokens or Basic Auth should be enforced on the main Webhook node.
