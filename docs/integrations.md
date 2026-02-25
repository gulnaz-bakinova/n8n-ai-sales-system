# Integrations & API Contracts

> This system is designed to be **provider-agnostic**. The core n8n workflow does not depend on specific WhatsApp provider nodes. Instead, it relies on a standardized HTTP Webhook payload.

## 1. Ingress Webhook Contract

Any messaging provider (ChatApp, Twilio, 360dialog, WATI) can be integrated by mapping their outgoing events to the following JSON structure via a middleware webhook or direct payload manipulation.

### Required Ingress Payload (Minimum JSON)

```json
{
  "chatId": "77700000000@c.us",
  "phone": "77700000000",
  "name": "Client Name",
  "messageId": "UNIQUE_MSG_ID_123",
  "type": "text",
  "message": "Hello. Yes, you can discuss it with me."
}
```

**Field Definitions:**
| Field | Type | Description |
| :--- | :--- | :--- |
| `chatId` | *(String, Required)* | The unique chat identifier used by the provider to route the reply. |
| `phone` | *(String, Required)* | User's phone number, used as the Primary Key in the Google Sheets CRM. |
| `name` | *(String, Optional)* | User's display name for personalized messaging. |
| `messageId` | *(String, Required)* | Crucial for **Idempotency**. Checked against the `processed_events` table to drop duplicate webhooks. |
| `type` | *(String, Required)* | Used by the Content Classification Router. Supported types: `text`, `voice`, `image`, `contact`. |
| `message` | *(String, Conditional)* | The text payload (required if `type=text`). |
| `mediaUrl` | *(String, Conditional)* | URL to download the asset (required if `type=voice` or `type=image`). |

**Webhook Response**
- The n8n Webhook node is configured to return a **`200 OK` immediately** upon receiving the payload.
- All heavy lifting (LLM API calls, RAG retrieval, CRM updates) is processed **asynchronously** to prevent provider timeouts and ensure webhook delivery stability.

## 2. External Services & APIs

1. **Google Gemini API:**
   - Models: `gemini-1.5-flash` or `gemini-pro` for agentic reasoning and classification.
   - Embeddings: `text-embedding-004` for vectorizing the Knowledge Base.

2. **Supabase (PostgreSQL + pgvector):**
   - Stores chunked documentation and their vector representations. Used natively via n8n's Vector Store integrations.

3. **Google Sheets API:**
   - Used as a lightweight Operational Database. Manages User States, Message Logs, Dead Letter Queues, and Analytics Aggregation.

4. **Telegram Bot API:**
   - Used for real-time alerting to human operators when the DLQ catches a critical error.
