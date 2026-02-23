# System Runbook & Incident Response

> This document outlines troubleshooting steps for the most common failure scenarios in the AI Sales Agent workflow. 

All unhandled exceptions are caught by the [99_global_incident_handler.json](..99_global_incident_handler.json) workflow, which logs the error to the Google Sheets DLQ (Dead Letter Queue) and sends a Telegram alert.

---

## Incident: LLM Provider Timeout / API Error
**Symptom:** AI nodes (Gemini) fail with HTTP 5xx or timeout errors.

**Impact:** Bot cannot generate responses or classify intents.

**Automatic Mitigation:**
- The nodes are configured with **Exponential Backoff (Retry on Fail)**: Max 3 tries, waiting 5 seconds between attempts.
- If all retries fail, the system triggers the Global Error Handler.


**Manual Resolution:**
1. Check the provider's status page (e.g., Google AI Studio, OpenAI).
2. If it's a hard outage, temporarily route the `Code: JSON Parser & Logic Extractor` fallback output to `MANAGER` to bypass the AI and alert human agents.
3. Check `lead_status` in the CRM; failed executions are marked as `Error`.

---

## Incident: Google Sheets API Rate Limit (429)
**Symptom:** CRM operations (Read/Update) fail due to Google API quota limits.

**Impact:** State management fails, potentially breaking the routing logic.

**Automatic Mitigation:** Critical read nodes have built-in n8n retries.

**Manual Resolution:**
1. Check Google Cloud Console for quota exhaustion.
2. If rate limits are consistently hit during mass broadcasts, implement a batching queue before CRM updates or migrate the state table to Supabase/PostgreSQL.

---

## Incident: Webhook Provider Disconnect
**Symptom:** No messages arriving in n8n; `processed_events` table is not updating.

**Manual Resolution:**
1. Check the n8n execution log to verify if the webhook is active.
2. Ensure the ChatApp/Provider URL matches the current n8n webhook URL.
3. If IDs are missing from the provider, fallback to generating a message_id hash dynamically in n8n: hash(body.phone + $now + body.message) to maintain strict idempotency.

---

## How to Debug Executions
1. Obtain the `execution_id` from the Telegram alert or the DLQ spreadsheet.
2. Go to the n8n dashboard -> Executions.
3. Paste the ID in the search bar.
4. Pin the input data on the failed node to rerun and test fixes safely.

## Edge Cases Handling Matrix

| Edge Case | System Behavior (Mitigation) |
|---|---|
| **Duplicate Webhook** | Intercepted at Layer 1 via `messageId` DB lookup. Returns 200 OK, halts execution. |
| **External API Timeout** | Exponential backoff (3 retries, 5s delay). If failed, triggers DLQ. |
| **LLM Invalid JSON Output** | Caught by `Parse JSON` custom `try/catch`. Defaults to `MANAGER` intent to force human handoff. |
| **Network Outage / Node Crash** | Caught by Global Error Handler. State saved to `dead_letters` table for manual replay. |
| **Manager Interrupts Bot** | Admin sends `/stop`. Updates `is_bot_active=FALSE`. Bot physically ignores future webhooks. |
