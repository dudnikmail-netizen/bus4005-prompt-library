# Prompt Iteration Log — BuildRange Materials

**Assessment 1 portfolio evidence (BUS4005-T5-W)**
Supports Criterion 1 (Prompt design and iteration). Linked from the submitted report.

See [PROMPT-LIBRARY.md](./PROMPT-LIBRARY.md) for the full 10-prompt library this log tests.

This log documents the design-test-refine cycle for the prompts in the library where meaningful iteration occurred. Every prompt is built on the RACE framework (Role, Action, Context, Expected output; adapted from AIPromptsX, 2025). Each entry below shows the actual prompt tested, what went wrong, and the lesson that shaped the next version.

---

## Prompt 1: Delivery Status Alert

**Grounded in real dispatch practice:** delays were communicated by phone or email from the dispatch team, but updates often left out a firm new date, the actual reason (stock shortage, freight delay, mill backorder), or any indication of what happens next — leaving customers to chase the answer themselves.

**Testing method:** each version tested in a fresh chat (no shared context between versions) on Claude Sonnet 5 (Anthropic), 15 September 2026, against the same case: order BM-7734, mill backorder, no confirmed new date.

| Version | Change made | Prompt text | Observed effect | Lesson learned |
|---|---|---|---|---|
| v1 | Baseline, no structure | "Write a message telling the customer their delivery is late." | Correctly did **not** invent a date. But returned two alternative drafts (A/B options), formatted as a full email with a subject line and `[Customer Name]`/`[Your Name]` placeholders — needs a person to pick one and finish it. | An unscoped prompt can be accurate and still be unusable for automation: it hands back a menu of polished choices instead of one finished, sendable artifact. |
| v2 | Added role + required fields | "You are a logistics assistant. Write a status update including order ID, status, and new time." | Correctly stated the date as unconfirmed and included all required fields cleanly. But it ended by asking which format the user wanted (email / internal note / casual message) — still not a finished output. | Requiring fields fixes content completeness, but without an explicit output-format constraint the model treats the task as a menu of options rather than a deliverable. |
| v3 (final) | Added single-output format constraint + word limit + uncertainty fallback | "You are a logistics/dispatch assistant. Given order ID, status, reason (stock shortage, freight delay, or mill backorder), and new time window, write a plain-language update under 60 words. If no new date is confirmed yet, say so explicitly rather than omitting it." | Produced exactly one complete, ready-to-send message — no placeholders, no options, no follow-up question — and correctly stated the new date was unconfirmed. | The decisive fix wasn't preventing hallucination — the model never fabricated a date, at any version. It was forcing a single, deterministic, complete output. Even a well-behaved model defaults to offering humans choices unless the prompt explicitly rules that out, which defeats the purpose of automation. |

*See `/evidence` folder for screenshots of all three test runs.*

---

## Prompt 3: Inquiry Triage

**Testing method:** each version tested in a fresh chat on Claude Sonnet 5 (Anthropic), 15 September 2026.

| Version | Change made | Prompt text | Observed effect | Lesson learned |
|---|---|---|---|---|
| v1 | Baseline, open-ended | "What is this message about?" | The model did not attempt to classify the message at all — it explained what the message was about in prose and offered to help draft a reply if asked. | An open-ended prompt doesn't just produce inconsistent labels — it may not produce a categorisation at all. The task needs to be explicitly framed as classification, not asked as a question. |
| v2 | Added closed category list | "Classify this message as Pricing, Stock, Delivery, or Complaint." | Tested on a genuine multi-issue message (delivery delay + stock question). The model identified both issues but hedged: proposed Delivery as primary and Stock as secondary, offering to apply both if multi-label tagging was supported — rather than committing to one label. | A closed category list narrows the vocabulary, but without a "pick exactly one" instruction the model reasons about ambiguity out loud instead of resolving it. |
| v3 | Constrained output format + order number extraction | "Classify this message as Pricing, Stock Availability, Delivery Status, Complaint or Other; extract order number; respond with category and number only." | Produced exactly the required output — "Delivery Status, BM-5521" — with no hedging or extra commentary. | Requiring a single fixed-format response, not just a fixed category list, is what actually forces a committed, one-line answer suitable for automated routing. |
| v4 (final) | Added structured JSON output + `human_review_required` escalation flag | "Classify this message; return JSON with category, order_number and human_review_required (true if a detail is missing or a policy exception applies)." | Hard test case (damaged goods, urgent, no resolution stated): correctly returned `human_review_required: true` with a sound reason (no confirmed resolution path or damage evidence on file). The category field came back as free text ("damaged_order_complaint") rather than one of v3's fixed labels. | The escalation flag worked exactly as intended on a genuine edge case. But it came at a cost: the fixed-category constraint from v3 was lost — richer per-case detail versus a consistent, closed taxonomy downstream systems could reliably route on. |
| Robustness test (v4) | No prompt change — tested on a calm, routine case | Same v4 prompt, run against: "My order #BM-6210 is due tomorrow. Can you guarantee it'll arrive on time?" | Did **not** return false as expected. The model set `human_review_required: true` again, reasoning that it has no access to real-time carrier data and so cannot confirm a delivery guarantee itself. | This wasn't the confirmation expected — it revealed something more useful: the model escalates any question it cannot verify against live data, regardless of urgency. Arguably the right conservative default for a tool with no system integration, but it means this prompt's automation value is reliable triage-and-log, not autonomous resolution, for any question requiring a real commitment. |

*See `/evidence` folder for screenshots of all five test runs.*

---

## Prompt 8: Complaint Response Draft

**Testing method:** each version tested in a fresh chat on Claude Sonnet 5 (Anthropic), 15 September 2026.

| Version | Change made | Prompt text | Observed effect | Lesson learned |
|---|---|---|---|---|
| v1 | Baseline, tone-only constraints | "Draft an empathetic reply: acknowledge the issue, avoid admitting liability, offer next steps, under 100 words." | Tested with a hard case (delayed order, urgent deadline, open refund request). Did not invent hard numbers, but used bracket placeholders for unknowns (`[X hours]` for a follow-up window, `[Customer Name]`, `[Your Name]`), asserted an action not on the case record ("I've escalated your order for urgent tracking"), and ended by asking whether a firmer commitment or a different version was wanted — not a finished, sendable message. | The same pattern seen in Prompts 1 and 3's v1: an unconstrained prompt doesn't necessarily fabricate facts — it produces a polished draft with placeholders and open questions instead of one deployable output. |
| v2 (final) | Added explicit grounding rule | "Draft an empathetic reply using only case-record facts; never invent timeframes, compensation or commitments; under 100 words." | Same hard case: produced one complete, ready-to-send message with no placeholders and no follow-up question. Stated only recorded facts (delayed, no confirmed date, refund requested, escalating for human review) and made no promise about timing. | The grounding rule fixed two things at once — no invented specifics, and no leftover choices for a human to resolve — confirming the single-output-format lesson from Prompt 1 generalises across the library, not just one prompt. |

*See `/evidence` folder for screenshots of both test runs.*

---

## Cross-Cutting Finding

The same failure pattern appeared independently in the v1 (unconstrained) version of all three tested prompts: Delivery Status Alert v1 returned two draft options with placeholders; Inquiry Triage v1 didn't attempt a category at all and offered further help instead; Complaint Response Draft v1 used bracket placeholders and asked which version was preferred. **None of the three fabricated hard facts.** In every case, the fix was the same: an explicit single-output-format constraint, not a request for "accuracy." This suggests the constraint that matters most for turning a capable model into an automation-ready one isn't primarily about truthfulness — it's about eliminating the model's default habit of leaving a human to finish or choose.

---

## Responsible AI Considerations

This library was designed against five risk categories, drawing on NIST's Generative AI Risk Management Profile (NIST, 2024):

- **Hallucination / confabulation** — the primary risk found through testing (see Cross-Cutting Finding above). Mitigated through explicit "state uncertainty" and "use only case-record facts" instructions, verified by manual testing on three prompts.
- **Bias** — prompts that touch subjective or consequential judgement (e.g. Credit Application Summary) are deliberately restricted to factual summarisation only — they never make or recommend a decision, so bias in the underlying model cannot directly drive a real outcome. This follows the general caution in Bender et al. (2021) that large language models can reproduce biases present in their training data when used for consequential judgements.
- **Privacy** — prompts handling customer or financial data (e.g. Credit Application Summary, Complaint Response Draft) are scoped to use only data already held in the business's own records, and do not request or output data beyond what each task needs.
- **Security** — no prompt stores, logs, or transmits data outside the immediate task; every output is a draft reviewed by staff before use, limiting the impact of any single incorrect or manipulated output.
- **Governance** — every prompt keeps a human decision-maker at the point of consequence (sending, approving, deciding), consistent with NIST's recommendation that human oversight sit at the highest-risk points of a GenAI-enabled workflow.

## Note

Prompts 2, 4–7, 9–10 followed the same design-test-refine cycle at a smaller scale (1–2 revisions each, converging on explicit roles, required fields, and output constraints). Full test transcripts available on request.

**References**

- AIPromptsX. (2025). *RACE framework: Role-aligned contextual expertise*. https://aipromptsx.com/prompts/frameworks/race
- Bender, E. M., Gebru, T., McMillan-Major, A., & Shmitchell, S. (2021). On the dangers of stochastic parrots: Can language models be too big? *ACM Conference on Fairness, Accountability, and Transparency* (pp. 610–623).
- National Institute of Standards and Technology. (2024). *Artificial intelligence risk management framework: Generative artificial intelligence profile* (NIST-AI-600-1). https://doi.org/10.6028/NIST.AI.600-1
