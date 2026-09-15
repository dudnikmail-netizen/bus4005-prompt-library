# Prompt Library — BuildRange Materials

**Assessment 1 report (BUS4005-T5-W)** — the full 10-prompt library. See [README.md](./README.md) for iteration evidence and test results.

| # | Prompt / Workflow | Problem | Prompt text | Automation potential | Risks & limitations |
|---|---|---|---|---|---|
| 1 | Delivery Status Alert | Dispatch updates often omit a firm date or reason. | "Given order ID, status, reason (stock shortage, freight delay, or backorder) and time window, write a plain-language update under 60 words; or say date unknown." | High — structured data maps to template. | Relies on accurate input; wrong fields still mislead. |
| 2 | Sales Quote Generator | Manual quoting is slow, inconsistent. | "Given product codes, quantities, prices, postcode, generate an itemised quote with GST; flag unknown codes." | High — rules-based formatting. | Outdated pricing; needs rep sign-off. |
| 3 | Inquiry Triage | Manual reading misses edge cases needing escalation. | "Classify this message; return JSON with category, order_number and human_review_required (true if details missing or policy exception applies)." | High — structured JSON feeds downstream systems. | Category field isn't fixed-list; still needs review. |
| 4 | Product Spec Summariser | Reps rewrite dense technical documents. | "Convert this spec sheet into a one-page plain-language summary: dimensions, load ratings, finishes, uses." | Medium — needs accuracy review. | Oversimplified safety ratings; needs technical sign-off (NIST, 2024). |
| 5 | Credit Application Summary | Manual application review slows onboarding. | "Summarise this application: business name, ABN, history, limit, flagged risks. Do not recommend approval." | Medium — summary only. | Sensitive data needs access controls; verify. |
| 6 | Supplier RFQ Comparison | Manual comparison is slow, inconsistent. | "Compare these quotes on price, lead time and terms as a table; note best value with rationale." | High — criteria-based. | Misses reliability; decision stays human. |
| 7 | Inventory Exception Report | Manual reviews miss shortages too late. | "List SKUs below reorder threshold; draft a one-line purchasing recommendation with supplier and lead time." | High — data-triggered. | Only as accurate as stock data; needs override. |
| 8 | Complaint Response Draft | Manual drafting delays resolution, invents unapproved promises. | "Draft an empathetic reply using only case-record facts; never invent timeframes or commitments; under 100 words." | Medium — draft only. | Hallucinated commitments; human review before sending (NIST, 2024). |
| 9 | Product Marketing Copy | Ad hoc content creation is slow. | "Write a 150-word description: material, finishes, ideal applications, warm professional tone." | Medium-high — needs brand review. | Overstated claims; needs marketing sign-off. |
| 10 | CRM Follow-Up Email | Reps lack time to chase dormant accounts. | "Draft a friendly follow-up referencing the customer's last order and a current promotion." | High — personalised at scale. | Stale CRM data misfires; review first. |
