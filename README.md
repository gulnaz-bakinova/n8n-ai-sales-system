# n8n AI Sales & Qualification Agent (WhatsApp)

*🇷🇺 [Русская версия](README.ru.md)*

> A production-ready WhatsApp sales assistant that drives conversations via a strict state-machine script, handles objections using a Vector Knowledge Base (RAG), qualifies leads (Scoring), executes automated follow-ups, and seamlessly hands off to human managers. Engineered for high reliability using deduplication, exponential retries, and centralized error handling.

![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?style=flat-square&logo=n8n)
![Gemini](https://img.shields.io/badge/Google_Gemini-AI_Agent-4285F4?style=flat-square&logo=google)
![Supabase](https://img.shields.io/badge/Supabase-pgvector_RAG-3ECF8E?style=flat-square&logo=supabase)
![Status](https://img.shields.io/badge/Status-MVP_Delivered-success?style=flat-square)

## Project Context & Status
- **Status:** MVP successfully delivered and tested (Inbound routing → AI Dialog & RAG → Handoff / Follow-up). Currently paused by the client due to internal budget restructuring.
- **Cross-Functional Collaboration:** Worked closely with the client's internal development team to design, negotiate, and establish the API integration contracts and Webhook payloads.
- **Deployment:** The provided [/docker/docker-compose.yml](/docker/docker-compose.yml) represents the planned "next step" for deploying the system onto the client's self-hosted infrastructure.

---

## System Architecture

The project is built as a highly modular, event-driven pipeline in **n8n**. 

<img src="./assets/00_system_architecture_diagram.jpg" width="400" alt="System Architecture">

*See the visual representation of the business logic and state machine.*

**Actual n8n Pipeline:**

<img src="./assets/05_n8n_workflow_overview.png" width="1000" alt="n8n Workflow">

*Production-ready n8n workflow with multimodal processing, RAG, and isolated state machine stages.*

### Core Business Features
| Feature | Description |
| :--- | :--- |
| 1. **State Machine Routing** | Leads are strictly guided through a sales funnel (`Discovery` -> `Education` -> `Closing`) towards autonomous registration. If a user deviates, the AI handles the query and smoothly steers them back into the script. |
| 2. **RAG Knowledge Base** | Integrated with **Supabase (pgvector)** to dynamically retrieve official company policies and overcome client objections without LLM hallucinations. |
| 3. **Multimodal Processing** | Native handling of Audio (Speech-to-Text via Whisper), Images (Vision AI), and vCard contacts, normalizing everything into a single text stream. |
| 4. **Smart Lead Scoring** | The LLM evaluates user intent, assigns a score (0-100), and classifies the lead as Cold/Warm/Hot in the CRM.|
| 5. **Batch Analytics Worker** | A cron-triggered workflow aggregates daily chat logs and uses an LLM to generate insights (drop-off points, top objections, success triggers). |
| 6. **Automated Follow-ups** | Automatically re-engages users who have been inactive for >24 hours to boost conversion rates. |
| 7. **No-Code CMS** | Sales scripts matrices are decoupled into Google Sheets, allowing business owners to update the bot's copy in real-time without code changes. |
| 8. **Anti-Ban Protection** | Instantly detects negative sentiment or opt-outs, putting the lead in a Do Not Contact state to prevent WhatsApp account bans. |



**Business Analytics Dashboard:**

<img src="./assets/01_metrics_dashboard.png" width="800" alt="Metrics Dashboard">

*Business Analytics Dashboard with key funnel metrics: total leads, hot leads, handoff rate, and error rate. Data is aggregated daily by the AI Analyst worker.*

---

## Engineering & Reliability Practices
The architecture incorporates production-ready patterns to ensure fault tolerance and stable performance under load:

*   **Idempotency & Deduplication:** Webhooks are checked against a `processed_events` database using unique `messageId`s. Duplicate events are dropped instantly to prevent double-replies.
*   **Global Error Handling (DLQ):** Any unhandled exception across the system triggers a central Dead Letter Queue workflow, logging the exact node failure and alerting admins via Telegram.
*   **Security & PII Masking:** A dedicated `PII_Guard` code node intercepts the payload, masking phone numbers and redacting credit card formats before data reaches the LLM or logs.
*   **Externalized Prompts:** System instructions are decoupled from logic nodes and versioned as Markdown files (see [/prompts](/prompts) ).

---

## Repository Structure

### Workflows (n8n JSON)
| Workflow | Description |
| :--- | :--- |
| [00_rag_data_ingestion.json](00_rag_data_ingestion.json) | **Data Prep.** Vectorizes the Knowledge Base and syncs it to Supabase (pgvector). |
| [01_core_ai_sales_agent.json](01_core_ai_sales_agent.json) | **The Brain.** Core logic: State machine, LLM routing, RAG retrieval, and Handoffs. |
| [02_retention_followup_worker.json](02_retention_followup_worker.json) | **Cron Job.** Daily scan to automatically re-engage leads inactive for >24 hours. |
| [03_batch_insights_analyst.json](03_batch_insights_analyst.json) | **Analytics.** Fetches 100 recent logs and uses LLM to generate script conversion insights. |
| [90_test_mock_provider.json](90_test_mock_provider.json) | **QA Tool.** Simulates incoming WhatsApp payloads (Text, Voice, Contacts) for local testing. |
| [99_global_incident_handler.json](99_global_incident_handler.json) | **DLQ.** Global error catcher that writes to a Dead Letter Queue and sends Telegram alerts. |

### Documentation (`/docs`)
* 📄 [ARCHITECTURE.md](./docs/ARCHITECTURE.md) — System components, data flow, and State Machine logic.
* 🔌 [Integrations & API](./docs/integrations.md) — Provider-agnostic Webhook contracts and payload specs.
* 🔒 [Security](./docs/security.md) — Secrets management, PII masking, and Human-in-the-loop guardrails.
* 📖 [Runbook](./docs/RUNBOOK.md) — Troubleshooting guide and edge-case handling matrix.
* 🧪 [Test Scenarios](./docs/TEST_SCENARIOS.md) — Golden set of test cases (RAG, Objections, Escalations).
* 🚀 [Deployment](./docs/DEPLOYMENT.md) — Planned infrastructure and Docker setup.

### Assets (`/assets`)
* 🗺️ [00_system_architecture_diagram.jpg](./assets/00_system_architecture_diagram.jpg) — Miro flow diagram.
* 📊 [01_metrics_dashboard.png](./assets/01_metrics_dashboard.png) — Business analytics dashboard.
* 🗄️ [02_supabase_pgvector.png](./assets/02_supabase_pgvector.png) — RAG vector database structure.
* 📝 [03_scripts_db.png](./assets/03_scripts_db.png) — Prompt variables.
* 📝 [04_crm_db.png](./assets/04_crm_db.png) — Google Sheets CRM.

---

## How to Evaluate / Test

Since the project is provider-agnostic, you can test the entire logic without a real WhatsApp connection:

1. Import the workflows into your n8n instance.
2. Configure credentials (or use mock data) based on the [.env.example](/.env.example).
3. Open the **[90_test_mock_provider.json](/90_test_mock_provider.json)** workflow.
4. Paste any payload from the [`/fixtures/`](/fixtures/) folder into the HTTP Request node and execute.
5. Monitor the execution in the **[01_core_ai_sales_agent.json](01_core_ai_sales_agent.json)** workflow to observe State routing, RAG retrieval, and JSON output parsing.

---

### 👤 Author

**Gulnaz Bakinova**

*AI Automation & Applied AI Engineer · End-to-end automation for sales / support / ops*

Let's connect!
[LinkedIn](https://www.linkedin.com/in/gulnaz-bakinova/) 

*This repository is provided for portfolio and demonstration purposes only*
