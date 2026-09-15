# Prompt Iteration Log — BuildRange Materials

**Assessment 1 portfolio evidence (BUS4005-T5-W)**
Supports Criterion 1 (Prompt design and iteration). Linked from the submitted report.

This log documents the design-test-refine cycle for the prompts in the library where meaningful iteration occurred. Each entry shows the actual prompt tested, what went wrong, and the lesson that shaped the next version.

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

*Design and rationale below; manual model verification pending — will be updated once tested.*

| Version | Change made | Prompt text | Observed effect | Lesson learned |
|---|---|---|---|---|
| v1 | Baseline, open-ended | "What is this message about?" | Free-text answers with inconsistent category labels (e.g. "shipping" vs "delivery"). | Needs a closed, defined category list to support downstream routing. |
| v2 | Added closed category list | "Classify this message as Pricing, Stock, Delivery, or Complaint." | More consistent labels, but multi-issue messages returned two categories in free text. | Needs a strict single-output format for automated routing. |
| v3 | Constrained output format + order number extraction | "Classify this message as Pricing, Stock Availability, Delivery Status, Complaint or Other; extract order number; respond with category and number only." | Consistent single-category output across test messages; order numbers correctly extracted. | Constrained output format is essential when a prompt feeds an automated routing step. |
| v4 (final) | Added structured JSON output + `human_review_required` escalation flag | "Classify this message; return JSON with category, order_number and human_review_required (true if a detail is missing or a policy exception applies)." | Hard test case: a damaged-goods complaint with no resolution stated and an urgent tone. v3 correctly assigned "Complaint" but gave no signal that the case needed staff attention. v4 correctly set `human_review_required: true` and did not attempt to resolve the request itself. | A category label alone isn't enough for edge cases — an explicit escalation flag is what actually stops the system from silently mishandling ambiguous requests. |
| Negative test (v4) | No prompt change — robustness check | Same v4 prompt, run against: "My order is due tomorrow. Can you guarantee delivery?" (a routine, on-time, non-urgent inquiry). | Correctly returned `human_review_required: false`, with review_reason "Order is currently on time and the customer is only inquiring." | An escalation mechanism needs testing on calm cases as well as hard ones — a flag that fires on everything is as unusable as one that never fires. |

---

## Prompt 8: Complaint Response Draft

*Design and rationale below; manual model verification pending — will be updated once tested.*

| Version | Change made | Prompt text | Observed effect | Lesson learned |
|---|---|---|---|---|
| v1 | Baseline, tone-only constraints | "Draft an empathetic reply: acknowledge the issue, avoid admitting liability, offer next steps, under 100 words." | Hard test case: a delayed order with an urgent deadline and an open refund request. The draft invented specifics nowhere in the case record — a "4-hour specialist callback," a named refund window of "3–7 business days," and self-service action codes like "EXPEDITE." | Tone and liability constraints alone don't stop fabrication — an ungrounded prompt will invent plausible-sounding commitments to sound helpful. |
| v2 (final) | Added explicit grounding rule | "Draft an empathetic reply using only case-record facts; never invent timeframes, compensation or commitments; under 100 words." | Same hard case: reply now states only what's recorded (delayed, no confirmed date, refund not yet approved, human review required) and explicitly avoids promising a timeframe or process not on file. | Naming the failure mode directly ("never invent timeframes, compensation or commitments") is more reliable than general instructions like "be accurate" — consistent with NIST's confabulation guidance (NIST, 2024). |

---

## Note

Prompts 2, 4–7, 9–10 followed the same design-test-refine cycle at a smaller scale (1–2 revisions each, converging on explicit roles, required fields, and output constraints). Full test transcripts available on request.

**References**

- National Institute of Standards and Technology. (2024). *Artificial intelligence risk management framework: Generative artificial intelligence profile* (NIST-AI-600-1). https://doi.org/10.6028/NIST.AI.600-1
