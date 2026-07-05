# AI Broker Agent (Langflow)

A compliance-aware **product recommendation agent** for a financial-services group, built as a [Langflow](https://www.langflow.org/) flow. It ingests a group's legal corpus, product catalogue, and customer profiles, then drafts **next-best-product recommendations** for each customer — while keeping a human compliance officer in the loop.

> **Domain:** prototype catalogue for *Custodian Investment Plc* (Nigeria) — an insurance-led financial holding group (general insurance, life assurance, pensions, trusteeship).

---

## Why this is interesting

Naïve recommendation bots happily suggest products a customer isn't legally allowed to be offered. This agent separates **"what would sell"** from **"what is allowed"**, mirroring how a regulated broker actually operates:

```
                ┌─────────────────────────┐
   Legal corpus │  1. Rule-Drafting Agent │  → machine-readable
   Products     │     (RAG over policy)   │    compliance rules (JSON)
   Cust. schema └────────────┬────────────┘
                             │
   Customer      ┌───────────▼─────────────┐
   Products      │  2. Recommendation Brain│  → ranked product offers
   Prod. KB      │     (upsell / add-on /  │    (best → worst)
                 │      cross-sell)        │
                 └───────────┬─────────────┘
                             │
                 ┌───────────▼─────────────┐
                 │  3. Gate Engine         │  → only compliant offers
                 │  (rules ∧ recommendations)│   survive
                 └─────────────────────────┘
```

1. **Rule-Drafting Agent** — retrieves law & policy passages from a vector store (AstraDB), then drafts recommendation rules as a strict JSON array for an automated compliance engine. It *drafts*; it never *approves*.
2. **Recommendation Brain** — for one customer, ranks every sensible next product, constrained to **same-subsidiary** moves (upsell, add-on, intra-cross-sell) using a product knowledge base.
3. **Gate Engine** — evaluates the drafted rules against each recommendation so only compliant offers pass through.

## Architecture

The flow (`Broker Agent--Version B1.json`) is 19 nodes:

| Stage | Components |
|-------|-----------|
| **Ingestion / RAG** | 5× `File` loaders, `SplitText`, `AstraDB` vector store (collection `legal_corpus`) |
| **Data shaping** | `CorpusToText`, `CustomerToText`, `ChunksToText`, `CustomerSchema`, 2× `TypeConverter` |
| **Reasoning** | 2× `Prompt Template`, 2× `LanguageModelComponent` (OpenAI) |
| **Compliance** | `FilterCustomer`, `GateEngine` |

## Repository contents

| File | Description |
|------|-------------|
| `Broker Agent--Version B1.json` | The Langflow flow (secrets replaced with `${ENV_VAR}` placeholders) |
| `products.json` | Product catalogue (10 products across 4 subsidiaries) |
| `legal_corpus.json` | Policy/legal passages used for RAG |
| `product_knowledge_qa.json` | Knowledge base: who each product suits, when to recommend |
| `subsidiary_and_surfaces.json` | Group + subsidiary structure |
| `customer_records.sample.json` | 50 **synthetic** customer profiles (no real PII) |

> The full 1,011-record synthetic dataset is kept local; a 50-record sample is included so the flow is runnable.

## Running it

1. Install Langflow: `pip install langflow` then `langflow run`.
2. In the UI, **Import** `Broker Agent--Version B1.json`.
3. Provide credentials (the flow ships with placeholders — nothing sensitive is committed):
   - `OPENAI_API_KEY`
   - `ASTRA_DB_APPLICATION_TOKEN`
   - `ASTRA_DB_API_ENDPOINT`
4. Point the `File` loaders at the JSON data files and run the flow.

## Data & safety notes

- All customer data is **synthetic** — pseudonymous IDs (`cust_000001`), financial-profile attributes only. No names, emails, or phone numbers.
- No live credentials are committed; the flow uses environment-variable placeholders.

---

*Built as a demonstration of agentic RAG, prompt engineering, and compliance-gated LLM reasoning.*
