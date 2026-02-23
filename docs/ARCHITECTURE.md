# System Architecture

This document outlines the high-level architecture and data flow of the n8n AI Sales Agent. The system is designed as an event-driven, multimodal pipeline with built-in reliability and safety guardrails.

## High-Level Components

1. **Automation Engine:** n8n (Node-based workflow execution).
2. **AI Provider:** Google Gemini (Reasoning, Language Detection, Embeddings).
3. **Knowledge Base (RAG):** Supabase with `pgvector` for semantic search.
4. **CRM & Logging:** Google Sheets (Operational DB, State Management, DLQ).
5. **Messaging Channel:** WhatsApp API Provider (via Webhook).

---

## Pipeline Layers (Main Workflow)

### Layer 1: Ingress & Security (Gateway)
- **Idempotency & Correlation:** The incoming messageId acts as both the idempotency key (dropping duplicates in the processed_events table) and the **Correlation ID**. It is preserved across the entire multimodal pipeline (voice/text/images) allowing full end-to-end distributed tracing.
- **Command Listener:** Scans for `/stop` or `/start` to allow human agents to take over (Kill-Switch).
- **PII Guard:** Masks phone numbers and redacts credit card patterns before data proceeds to logs or LLM inputs.

### Layer 2: Multimodal Normalization
- **Voice to Text:** Downloads audio files and uses Whisper for STT (Speech-to-Text).
- **Vision:** Passes images to Vision AI to extract text/context.
- **Contacts:** Parses vCard payloads into readable text.
- *Result:* All media types are unified into a single text stream for the AI.

### Layer 3: State Machine & Context
- **Language Detection:** Uses a fast LLM call to classify the language (RU/KZ) and adjust prompt templates accordingly.
- **State Management:** Fetches the lead's current stage_id (e.g., 1, 2, 3) and corresponding funnel code (like stage_offer, offer_accepted_part1) from the Google Sheets CRM to route the conversation accurately.


### Layer 4: AI Brain & RAG Engine
- **Agentic Routing:** The AI decides whether to reply directly, use the RAG tool, or escalate to a human.
- **RAG Integration:** If the user asks a specific policy/pricing question or raises an objection, the AI queries the Supabase vector store (`objections` tool).
- **Structured Output & Intent Routing:** The LLM is strictly prompted to return a specific JSON schema (output, lead_score, lead_status, reason). The output field acts as an execution trigger—returning either natural conversational text or strict system codes (e.g., SEND_INITIAL_OFFER, MANAGER, TERMINATE_CONVERSATION, FORWARD_CONTACT_TO_MANAGER) to drive the deterministic n8n state machine.


---

## Reliability & Observability Features

1. **Dead Letter Queue (DLQ):** A global error trigger catches any node failure, logs the `execution_id` and payload to a dedicated Google Sheet, and sends a Telegram alert to the admin.
2. **Retry Mechanism:** Exponential backoff (3 retries, 5s wait) is configured on all external API calls (LLM, Database) to handle temporary network/rate-limit issues.
3. **Batch Analytics:** A standalone worker runs daily to aggregate chat logs, analyze drop-off points, and generate optimization reports.
4. **Automated Follow-up:** A cron-triggered workflow checks for leads inactive for >24 hours and sends a re-engagement message.

## State Machine & Handoff Policy

The conversation flow is strictly governed by a centralized State Machine, visually mapped in [/assets/00_system_architecture_diagram.jpg](/assets/00_system_architecture_diagram.jpg).
- **Transitions:** `Stage 1 (Discovery)` → `Stage 2 (Education)` → `Stage 3 (Closing)`.
- **Escalation Policy:** Handoff to a human manager is triggered automatically if:
1) The user explicitly requests a human.
2) The user expresses negative sentiment (triggers DNC).
3) The RAG tool fails to find a confident answer.
4) The AI outputs a JSON parsing error.
