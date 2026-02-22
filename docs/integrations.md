# Integrations & API Contracts

This system is designed to be **provider-agnostic**. The core n8n workflow does not depend on specific WhatsApp provider nodes. Instead, it relies on a standardized HTTP Webhook payload.

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
