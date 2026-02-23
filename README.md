# n8n AI Sales & Qualification Agent (WhatsApp)

> **Core Positioning:** A production-ready WhatsApp sales assistant that drives conversations via a strict state-machine script, handles objections using a Vector Knowledge Base (RAG), qualifies leads (Scoring), executes automated follow-ups, and seamlessly hands off to human managers. Engineered for high reliability using deduplication, exponential retries, and centralized error handling.

## Project Context & Status
- **Status:** MVP successfully delivered and tested (Inbound routing → AI Dialog & RAG → Handoff / Follow-up). Currently paused by the client due to internal budget restructuring.
- **Cross-Functional Collaboration:** Worked closely with the client's internal development team to design, negotiate, and establish the API integration contracts and Webhook payloads.
- **Deployment:** The provided [/docker/docker-compose.yml](/docker/docker-compose.yml) represents the planned "next step" for deploying the system onto the client's self-hosted infrastructure.

---

## System Architecture

The project is built as a highly modular, event-driven pipeline in **n8n**. 

![System Architecture] [./assets/00_system_architecture_diagram.jpg](./assets/00_system_architecture_diagram.jpg)
*(See the visual representation of the business logic and state machine).*

### Core Business Features
1. **State Machine Routing:** Leads are strictly guided through a sales funnel (`Discovery` -> `Education` -> `Closing`) based on their CRM `stage_id`.
2. **RAG Knowledge Base:** Integrated with **Supabase (pgvector)** to dynamically retrieve official company policies and overcome client objections without LLM hallucinations.
3. **Multimodal Processing:** Native handling of Audio (Speech-to-Text via Whisper), Images (Vision AI), and vCard contacts, normalizing everything into a single text stream.
4. **Smart Lead Scoring:** The LLM evaluates user intent, assigns a score (0-100), and classifies the lead as Cold/Warm/Hot in the CRM.
5. **Batch Analytics Worker:** A cron-triggered workflow aggregates daily chat logs and uses an LLM to generate insights (drop-off points, top objections, success triggers).
6. **Automated Follow-ups:** Automatically re-engages users who have been inactive for >24 hours to boost conversion rates.

---

## Engineering & Reliability Practices
This project implements Senior-level automation patterns to ensure it survives in a harsh production environment:

*   **Idempotency & Deduplication:** Webhooks are checked against a `processed_events` database using unique `messageId`s. Duplicate events are dropped instantly to prevent double-replies.
*   **Global Error Handling (DLQ):** Any unhandled exception across the system triggers a central Dead Letter Queue workflow, logging the exact node failure and alerting admins via Telegram.
*   **API Resilience:** All external API calls (LLM, Database) utilize Exponential Backoff (Retries) to survive rate limits (`HTTP 429`) and network blips.
*   **Security & PII Masking:** A dedicated `PII_Guard` code node intercepts the payload, masking phone numbers and redacting credit card formats before data reaches the LLM or logs.
*   **Human-in-the-Loop (Kill-Switch):** Managers can take over any chat by sending a `/stop` command, which updates the DB and physically severs the AI's ability to reply to that lead.
*   **Externalized Prompts:** System instructions are decoupled from logic nodes and versioned as Markdown files (see [/prompts](/prompts) ).

---

## Repository Structure

- [`/`](/) — Core n8n Workflow JSON exports.
- [`/docs/`](/docs/) — Deep-dive documentation (`ARCHITECTURE.md`, `RUNBOOK.md`, `integrations.md`, `security.md`, `TEST_SCENARIOS.md`).
- [`/prompts/`](/prompts/) — Version-controlled LLM System Instructions.
- [`/fixtures/`](/fixtures/) — Sample JSON payloads for testing different edge cases (Text, Voice, Contacts).
- [`/docker/`](/docker/) — Production-ready `docker-compose.yml` (n8n + Postgres/pgvector).
- [`/assets/`](/assets/) — Architecture diagrams and Dashboard screenshots.

---

## How to Evaluate / Test

Since the project is provider-agnostic, you can test the entire logic without a real WhatsApp connection:

1. Import the workflows into your n8n instance.
2. Configure credentials (or use mock data) based on the [.env.example](/.env.example).
3. Open the **[90_test_mock_provider](/90_test_mock_provider)** workflow.
4. Paste any payload from the `/fixtures` folder into the HTTP Request node and execute.
5. Monitor the execution in the **`01_core_ai_sales_agent`** workflow to observe State routing, RAG retrieval, and JSON output parsing.
