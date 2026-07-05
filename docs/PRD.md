# Product Brief — AI Broker Agent

*A one-page PRD-style overview. This is a portfolio prototype, not a shipped product; metrics below are illustrative targets to frame the thinking.*

---

## 1. Problem statement

Regulated financial groups leave cross-sell/upsell revenue on the table because the safe path (manual compliance review) is slow and inconsistent, and the fast path (a naïve recommendation model) risks offering products a customer isn't legally allowed to be offered. There is no cheap way to get **ranked, compliant, ready-to-approve** offers at customer scale.

## 2. Target users

| User | Job-to-be-done | What they get |
|------|----------------|---------------|
| **Compliance officer** | Approve offers without hand-checking every rule | Draft rules citing the source law + a gated offer list to sign |
| **Relationship manager / broker** | Know the best next product for a customer, safely | Ranked offers with plain-English reasons and an allowed action level |
| **Product / growth lead** | Grow cross-sell without regulatory risk | A scalable, auditable pipeline instead of spreadsheets |

## 3. Value proposition

> Turn "which offers are both smart *and* allowed?" from a manual, per-customer judgement call into a reviewable, auditable pipeline — ranked by fit, capped by law.

## 4. Scope (what it does / doesn't)

**Does**
- Rank every in-scope next product per customer (upsell / add-on / intra-cross-sell).
- Draft compliance rules from the group's legal corpus via RAG.
- Gate recommendations against rules and cap the action level.
- Constrain offers to **same-subsidiary** moves only.

**Doesn't (by design)**
- Approve offers autonomously — a human signs.
- Contact customers or execute sales.
- Recommend across subsidiaries the customer has no holding in.
- Invent customer fields or product IDs (hard prompt guardrails).

## 5. Key design decisions

1. **Two models, two jobs.** Commercial ranking ≠ compliance. Separating them prevents "the model wanted the sale" from overriding the rules.
2. **Draft, don't decide.** Every LLM step produces a *proposal* for human sign-off — the compliance-safe default in a regulated domain.
3. **Everything is reason-coded and sourced.** Rules cite the law passage; offers carry a fit reason. Auditable end to end.
4. **Conservative by default.** Where law stresses suitability/affordability/licensed advice, cap below "Recommend".

## 6. Success metrics (illustrative)

| Metric | Why it matters |
|--------|----------------|
| **Compliance precision** — % of surfaced offers a human approves unchanged | Trust; measures whether the gate actually works |
| **Officer review time per 100 offers** | The efficiency win vs. manual review |
| **Cross-sell conversion on surfaced offers** | Commercial value of the ranking quality |
| **Zero out-of-policy offers reaching customers** | The non-negotiable guardrail |
| **Coverage** — % of customers with ≥1 valid ranked offer | Reach of the engine |

## 7. Risks & mitigations

| Risk | Mitigation |
|------|-----------|
| LLM invents a field/product/value | Hard prompt constraints: fields from schema, values from allow-lists, IDs checked against catalogue |
| Over-eager offers create regulatory exposure | Conservative action-level caps + mandatory human sign-off |
| Rules drift from current law | Legal corpus is the single source; re-run drafting when policy changes |
| Hallucinated justification | Every rule must cite a specific source passage |

## 8. Roadmap (next iterations)

- **v2** — Feedback loop: capture officer edits to rules and offers to fine-tune ranking.
- **v2** — Explainability UI for the compliance officer (rule → source passage → affected offers).
- **v3** — Multi-channel action routing (which allowed channel to use per offer).
- **v3** — A/B test action-level caps against conversion, within compliance limits.
- **v3** — Extend beyond same-subsidiary once cross-subsidiary consent logic is modelled.
