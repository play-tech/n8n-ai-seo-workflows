# Engineering Architecture & Production Design Principles

**Author:** Mohammad Haider Ali (`amhaider442@gmail.com`)

---

## 1. Idempotency & State Management

In production marketing and revenue automation, webhooks from Meta Lead Ads, Shopify Customer Events, Typeform, or Stripe frequently deliver **at-least-once** payloads. Without idempotency, a single form submission can trigger duplicate CRM contacts, duplicate WhatsApp outreach, and skewed ROAS reporting.

### Implementation Pattern
1. **Normalization Layer:** Strip formatting from phone numbers (`E.164`) and lowercase/trim email addresses prior to hashing.
2. **Composite Idempotency Key:** Compute `SHA256(normalized_email + ":" + normalized_phone + ":" + event_type)`.
3. **Database Upsert:** Use `ON CONFLICT (idempotency_key) DO UPDATE` in Supabase/PostgreSQL or BigQuery staging tables before invoking downstream LLM or CRM nodes.

---

## 2. Deterministic AI Agent Nodes

When embedding LLMs (`Claude 3.7 Sonnet` / `GPT-4o`) inside automated pipelines:
- **Strict Schema Enforcement:** Every LLM call enforces `response_format: { type: "json_object" }` or structured tool calling.
- **Context Window Hygiene:** Raw HTML scraped via Firecrawl is converted to clean Markdown (`onlyMainContent: true`) and truncated to relevant semantic sections before prompt injection.
- **Fallback Routing:** If an LLM node returns malformed JSON or times out after 3 retries, the execution branches to a deterministic rule-based fallback rather than dropping the lead.

---

## 3. Server-Side Tracking & Attribution (Meta CAPI + GA4)

To solve signal loss from iOS privacy changes and ad-blockers:
- Browser pixel events (`fbp`, `fbc`, `client_user_agent`) are captured alongside a deterministic `event_id`.
- When a lead converts in Close CRM, HubSpot, or Shopify, n8n dispatches a server-side **Meta Conversions API (CAPI)** event with the matching `event_id` so Meta deduplicates browser vs. server events cleanly.
