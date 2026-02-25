# Test Scenarios (Golden Set)

> This document outlines the core test cases used to evaluate the AI Agent's intent classification, RAG retrieval accuracy, and guardrail enforcement.

| Scenario | Input Message | Expected Intent / Action | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- |
| **1. Cold Start (Interest)** | "Добрый день! Да, со мной можно поговорить. Можно узнать подробнее?" | Route to Stage 2 -> `SEND_INITIAL_OFFER` | Extracted intent, updated CRM, sent Stage 2 | ✅ Pass |
| **2. RAG Retrieval (Objection)**| "Вы мошенники? Откуда мой номер?" | Query Supabase vector store -> Output KB answer | Matched `Tool: RAG Knowledge Base` tool, returned official policy | ✅ Pass |
| **3. Negative Guardrail** | "Отстаньте, удалите мой номер" | Trigger DNC logic -> `TERMINATE_CONVERSATION` | Marked `is_bot_active=FALSE`, closed lead | ✅ Pass |
| **4. Human Handoff** | "Я хочу поговорить с менеджером" | Escalate -> `MANAGER` | Alerted admin via WhatsApp, paused AI | ✅ Pass |
| **5. Multimedia (Audio)** | *(Voice message asking about price)* | Whisper STT -> Classify -> Respond | Transcribed, categorized as pricing query, answered | ✅ Pass |
| **6. Lead Referral (Contact)** | "Это не ко мне, свяжитесь с директором: Айжан 87771112233" | Forward details -> `FORWARD_CONTACT_TO_MANAGER` | Extracted contact, confirmed to user, alerted manager | ✅ Pass |

*Note: Re-verified after each prompt or architecture change.*
