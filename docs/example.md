# Worked Example — customer in → compliant offers out

This traces one real (synthetic) customer through the full pipeline. Data is taken verbatim from the repo's sample files; the LLM outputs below are representative of the flow's prompt schemas.

---

## Step 0 · The input customer

From `customer_records.sample.json`:

```json
{
  "customer_id": "cust_000001",
  "age": 39,
  "region": "Lagos",
  "employment_type": "formal_private",
  "monthly_income_estimate_naira": 850000,
  "dependants_count": 3,
  "marital_status": "married",
  "risk_tolerance": "medium",
  "financial_goals": ["children_education", "protect_family"],
  "data_sharing_marketing_consent": "explicit",
  "holdings": [
    { "product_id": "prod_005", "subsidiary_id": "sub_life", "status": "active",
      "opened_months_ago": 18, "premium_naira": 42000 }
  ]
}
```

They hold **prod_005 · Term Life Protection Plan** (subsidiary `sub_life`). Because offers are constrained to **same-subsidiary** moves, the only in-scope candidates are other `sub_life` products: `prod_006` (Education Endowment) and `prod_007` (Capital Builder).

---

## Step 1 · Recommendation Brain (commercial ranking)

The brain ranks every in-scope product by fit — it does **not** consider legality.

```json
{
  "customer_id": "cust_000001",
  "candidates": [
    {
      "rank": 1,
      "recommended_product_id": "prod_006",
      "recommended_product_name": "Education Endowment Plan",
      "subsidiary_id": "sub_life",
      "trigger_product_id": "prod_005",
      "type": "intra_cross_sell",
      "intended_action_level": "Recommend",
      "fit_bucket": "high",
      "reason_code": "goal_match_education",
      "fit_reason": "Married, 3 dependants, explicit goal 'children_education' — endowment directly funds school fees."
    },
    {
      "rank": 2,
      "recommended_product_id": "prod_007",
      "recommended_product_name": "Capital Builder Plan",
      "subsidiary_id": "sub_life",
      "trigger_product_id": "prod_005",
      "type": "intra_cross_sell",
      "intended_action_level": "Recommend",
      "fit_bucket": "medium",
      "reason_code": "surplus_income_savings",
      "fit_reason": "₦850k monthly income supports a savings-oriented life product, but no explicit wealth goal stated."
    }
  ]
}
```

---

## Step 2 · Rule-Drafting Agent (compliance, from RAG)

Independently, the agent retrieves law/policy passages and drafts rules. Life products stress **suitability and licensed advice**, so the drafted cap is below "Recommend":

```json
[
  {
    "rule_id": "rule_014",
    "type": "intra_cross_sell",
    "subsidiary_id": "sub_life",
    "trigger_product_id": "prod_005",
    "recommended_product_id": "prod_006",
    "readable": "A term-life holder may be INFORMED about the Education Endowment Plan, but a licensed adviser must confirm suitability before recommending.",
    "conditions": [
      { "field": "holdings", "operator": "holds_product", "value": "prod_005" },
      { "field": "data_sharing_marketing_consent", "operator": "equals", "value": "explicit" }
    ],
    "required_consent": ["data_sharing_marketing_consent"],
    "allowed_channels": ["app", "adviser"],
    "max_action_level": "Inform",
    "reason_code": "suitability_licensed_advice",
    "source": "Insurance suitability & disclosure guidance — advice on long-term life products requires a licensed adviser.",
    "status": "draft"
  }
]
```

---

## Step 3 · Gate Engine (rules ∧ recommendations)

The engine intersects the ranked offers with the drafted rules and **caps the action level**:

| Rank | Product | Brain wanted | Rule cap | Consent OK? | ✅ Final surfaced action |
|------|---------|--------------|----------|-------------|--------------------------|
| 1 | prod_006 Education Endowment | Recommend | **Inform** | ✔ explicit | **Inform** (adviser confirms suitability) |
| 2 | prod_007 Capital Builder | Recommend | *no signed rule* | ✔ | **Silent** (held back — no approved rule yet) |

**Result handed to the compliance officer:**

> For `cust_000001`: **Inform** about the *Education Endowment Plan* via app/adviser (goal match: children's education; consent on file). *Capital Builder* is suppressed pending a signed rule.

---

## What this demonstrates

- The commercial model wanted to **Recommend** both — the compliance layer **downgraded one to Inform and suppressed the other**. Neither model could override the other.
- Every decision is **traceable**: fit reason from the brain, source law from the rule, consent check from the gate.
- Nothing reaches the customer without a rule a human has (or will) sign.
