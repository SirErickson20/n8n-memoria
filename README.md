# 🧠 n8n Long-Term Memory & Agentic Summarization Circuit

[![n8n](https://img.shields.io/badge/Orchestrator-n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io/)
[![OpenAI](https://img.shields.io/badge/LLM-GPT--4o--mini-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com/)
[![Airtable](https://img.shields.io/badge/Database-Airtable-18BFFF?style=for-the-badge&logo=airtable&logoColor=white)](https://airtable.com/)

Implementación de una arquitectura híbrida de **Persistencia Externa (Memoria de Largo Plazo)** y **Limpieza de Contexto (Agentic Summarization)** en n8n, diseñada para erradicar la amnesia entre ejecuciones efímeras de flujos y mitigar la saturación de tokens en sesiones extendidas.

---

## 📌 Arquitectura del Circuito

El flujo intercepta las interacciones entrantes a través de un Webhook, realiza una consulta determinista en una base de datos externa relacional (Airtable) filtrando por `Session_ID`, e inyecta semánticamente el perfil y la última síntesis en el `System Prompt` del **AI Agent**. Cuando el historial supera el umbral de 5 mensajes, se dispara una rutina de condensación ejecutiva mediante `gpt-4o-mini` y un `Structured Output Parser`.

```text
               [Webhook Inbound Trigger] (POST /agent-memory-entry)
                                   │
                                   ▼
                [Airtable: Buscar Registro por Session_ID]
                                   │
                                   ▼
                       [IF: ¿Usuario Existe en DB?]
                         ├── NO (False) ──► [Airtable: Registrar Usuario] ──► [Set: Contexto Inicial]
                         │                                                             │
                         └── SÍ (True)  ──► [Set: Formatear Contexto] ─────────────────┤
                                                                                       ▼
                                                                        [AI Agent (Tools Agent)]
                                                                        (OpenAI GPT-4o-mini + Buffer)
                                                                                       │
                                                                                       ▼
                                                                          [IF: ¿Supera 5 Mensajes?]
                                                                           ├── SÍ (>5) ──► [LLM Summarizer] ──► [Airtable: Update]
                                                                           │                                            │
                                                                           └── NO (<=5) ────────────────────────────────┤
                                                                                                                        ▼
                                                                                                             [Respond to Webhook]
