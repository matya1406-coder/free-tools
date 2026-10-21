# First automation offer: a practical starter kit

This is the complete text edition of a beginner portfolio and pitching kit. It contains a short guide, three worked fictional case studies, an editable service profile, five messages and three exercises with model answers. It is not an executable workflow or a PDF/DOCX/ZIP package.

## Assumptions and honest use

- You understand basic n8n nodes, expressions and branching, but have no client delivery experience to claim.
- All businesses, people, records and scenarios below are invented for learning. No workflow has been run or tested as part of this document. All outputs are expected outputs, not measured results.
- Use synthetic records in a private practice environment. Do not connect real customer accounts, send messages or use credentials for these exercises.
- Suggested n8n node sequences are design sketches. Node options and field handling must be checked in your installed version.
- You are initially offering a written process review or demo specification, not claiming readiness to deploy production systems. Expand your service only after you can demonstrate the necessary skills.

## 1. Short guide: move from learning to a specific offer

### Pick one problem you can explain

Use enquiry triage as a starting point: “Enquiries arrive in a shared inbox, and someone manually decides who should handle them.” Ask whether this actually happens before proposing automation. Do not assume the business needs n8n.

Your first offer can be: “I will map one enquiry process and write a small automation brief using fictional data. You receive the input fields, routing rules, expected output and situations requiring a human decision.”

Bound it: one intake source, at most three enquiry types, one output table and one review. Exclude live email, integrations, deployment and ongoing monitoring. The first useful outcome is a clear decision about what to automate, not a sale of a large system.

### Make an evidence ladder

1. Written design: label it “Fictional design exercise; not implemented.”
2. Local demo: build it with synthetic data; record the version, date, actual outputs and failures.
3. Tested demo: publish only the checks you actually ran, including limitations.
4. Client work: describe it only with permission and only after it exists.

Never copy a model answer and present it as a completed implementation. Screenshots and test logs should come from your own work. Avoid “saved hours,” “increased revenue” or “production-ready” unless you have evidence and a defensible measurement.

### Ask five discovery questions

1. Where does the input arrive, and who owns the process?
2. What fields are required, and what makes a record invalid?
3. What rules determine the next action?
4. What happens when information is missing or the process fails?
5. What observable output would make a small demo useful?

### Publish a compact portfolio entry

Use this order: status label; fictional problem; input; rules; expected output; actual test evidence if any; limitations; next learning step. A useful case study explains decisions and exceptions, not just the names of nodes.

### Begin a conversation

Look for someone who has described the specific process problem. Reply with a useful mini checklist or example first. Ask one relevant question. Use only claims you can support and follow the community's posting rules. A non-response is not an invitation to keep sending messages.

## 2. Case study A — enquiry routing

**Fictional educational design; not implemented or tested.**

Scenario: Cedar Demo Studio receives three enquiry types. Its coordinator needs a routing table, not automatic outgoing messages.

Input schema: `enquiry_id` (required string), `email` (required string), `category` (string), `message` (string). Trim whitespace and lowercase email/category. For this exercise only, a basic email check requires one @, non-empty local/domain parts and a dot in the domain; this is not full email validation or proof of deliverability.

Rules in order:

1. Missing ID or failed email check goes to manual review with its reason; do not add it to processed IDs.
2. A valid ID already in the processed set is a duplicate; do not route it again.
3. `sales` goes to Sales; `support` to Support; `billing` to Accounts. Anything else goes to manual review.
4. After storing a valid record's routed or review output, add its ID to the processed set. In a real system, durable storage and an atomic uniqueness check are needed to handle restarts and concurrent executions.

Fixture, starting with an empty processed set:

| Row | enquiry_id | email | category | message |
|---|---|---|---|---|
| 1 | E101 | ada@example.com | sales | Need a quote |
| 2 | E102 | ben@example.com | support | Need help |
| 3 | E101 | ada@example.com | sales | Need a quote |
| 5 | E104 | cam@example.com | partnership | Collaboration |

Expected output:

| Row | status | owner | reason |
|---|---|---|---|
| 1 | routed | Sales | matched category |
| 2 | routed | Support | matched category |
| 3 | duplicate | none | ID already processed |
| 4 | review | Coordinator | invalid email |
| 5 | review | Coordinator | unknown category |

Expected processed IDs: E101, E102, E104. Output record fields: `enquiry_id`, `status`, `owner`, `reason`. Keep the original row number in the demo log to distinguish duplicates.

Suggested design: Manual Trigger → synthetic input → normalization/validation → duplicate decision → category Switch → output table. This is a conceptual sequence; implement state explicitly rather than assuming a node remembers earlier records.

Acceptance specification: the five rows match the table; a second run against the retained processed set marks rows 1, 2, 3 and 5 duplicate, while row 4 still needs review. A storage failure must stop processing before an ID is marked complete. No outbound message is sent.

Portfolio copy: “I designed a fictional enquiry-routing demo with validation, duplicate handling and manual-review rules. The expected behaviour is documented; implementation and testing are still pending.”


**Fictional educational design; not implemented or tested.**



Fixture (all contacts are `accounts@example.com`):

| ID | status | due_date | outstanding_AED | last_reminder |
|---|---|---|---:|---|
| I201 | unpaid | 2026-10-10 | 250 | blank |
| I202 | paid | 2026-10-01 | 0 | blank |
| I203 | unpaid | 2026-10-19 | 100 | blank |
| I204 | unpaid | 2026-10-11 | 180 | 2026-10-16 |
| I205 | unpaid | invalid | 80 | blank |
| I206 | unpaid | 2026-10-01 | 300 | 2026-10-12 |

Expected decisions: I201 eligible; I202 skipped (paid); I203 skipped (not overdue); I204 skipped (recent reminder); I205 review (invalid due date); I206 eligible (seven-day boundary).



Suggested design: Manual Trigger → fixture → validate → date/status filters → candidate table with draft text. Keep credentials and email-send nodes out of the demo. Do not change `last_reminder` merely because a draft was generated; a future live system would update it only after confirmed sending and needs separate approval and failure handling.

Acceptance specification: exactly two candidates, three skipped rows and one review row; amounts and IDs remain unchanged; no sends occur. Additional checks to run when built: missing contact → review; negative amount → review; last reminder tomorrow → review; unpaid with zero balance → skipped; duplicate ID → review for all occurrences.

Portfolio copy: “This fictional design prepares reminder candidates for human review. It specifies date boundaries and invalid-data handling. It does not send emails or demonstrate any reduction in overdue payments.”

## 4. Case study C — weekly order summary

**Fictional educational design; not implemented or tested.**

Scenario: Birch Demo Shop wants a weekly paid-order summary. Use the interval `2026-10-12T00:00:00Z` inclusive through `2026-10-19T00:00:00Z` exclusive. All amounts are AED; mixed currencies require review and must not be summed together. “Paid total” is a sum of input order amounts, not accounting revenue or profit.

Input fields: `order_id`, `timestamp`, `status`, `amount_AED`. Validate required fields, UTC timestamps, non-negative finite amounts and status in `paid`, `cancelled`, `pending`. Exclude invalid rows to review. Collapse identical valid duplicates by ID; conflicting duplicates require review for every row of that ID and are excluded from totals.

| Row | order_id | timestamp | status | amount_AED |
|---|---|---|---|---:|
| 1 | O301 | 2026-10-12T00:00:00Z | paid | 120 |
| 2 | O302 | 2026-10-14T09:00:00Z | paid | 80 |
| 3 | O302 | 2026-10-14T09:00:00Z | paid | 80 |
| 4 | O303 | 2026-10-15T11:00:00Z | cancelled | 50 |
| 5 | O304 | 2026-10-19T00:00:00Z | paid | 90 |
| 6 | O305 | 2026-10-18T10:00:00Z | paid | -10 |

Expected summary:

```json
{
  "period_start_inclusive": "2026-10-12T00:00:00Z",
  "period_end_exclusive": "2026-10-19T00:00:00Z",
  "currency": "AED",
  "input_rows": 6,
  "paid_order_count": 2,
  "paid_total": 200,
  "average_paid_order": 100,
  "excluded_identical_duplicate_rows": 1,
  "excluded_non_paid_rows": 1,
  "excluded_outside_period_rows": 1,
  "review_rows": 1
}
```

Classify each row once in this order: invalid/review, duplicate, outside period, non-paid, included. Row 6 is review, row 3 is duplicate, row 5 is outside, row 4 is non-paid, rows 1 and 2 are included. This reconciles all six input rows.

Suggested design: Manual Trigger → fixture → validation → duplicate checks → period/status filtering → aggregate → report text. Use integer minor units for money in an implementation. If there are no paid orders, output count 0, total 0 and average null; do not divide by zero. Review rows must appear in the report warning rather than disappearing silently.


Portfolio copy: “I specified a fictional weekly reporting workflow with explicit time boundaries and duplicate reconciliation. The totals are worked examples from synthetic data, not client results.”

## 5. Editable service profile

Replace bracketed fields before sharing. Keep the demo labels.

**[Your name] — beginner automation process documentation**

I am learning n8n and documenting small automation ideas with synthetic data. I help make one repetitive process easier to understand before anyone decides whether to automate it.

**Starter service:** a written process map and demo specification for one enquiry intake process.

**You receive:** one input-field list, up to three routing rules, an output table, five example records, exception rules and an acceptance checklist. One wording revision is included.

**What I need:** an anonymised description of the current steps, the desired outcome and the person who handles exceptions. Please do not send passwords, API keys or personal customer records.

**Scope:** one input source and one output format. Delivery is documentation only. Live integrations, deployment, security assessment, technical support and maintenance are excluded.

**Evidence:** [Describe only your own completed work and actual checks.] My fictional design exercises are labelled as educational examples. I do not yet have client outcomes to report.

**Timing:** [Choose a realistic delivery date after reviewing the scope.] **Price:** [Agree before starting; do not advertise paid work without confirming applicable licensing requirements.] **Contact:** [Your contact].

**Completion:** the document covers the agreed input, rules, output and exception handling; you can request one clarification or wording revision within seven days of delivery.

## 6. Five editable outreach messages

These are templates for the learner's own use, separate from the kit provider's pilot invitation. Replace every bracket and remove anything untrue before sending. Do not imply you have built an unbuilt demo.

### A. Helpful public reply

For [specific process], a useful first step is to list the required input fields, the routing rule and the exception that needs a person. I am learning n8n and can share a short written outline using fictional data. Which step currently needs the most manual checking?

### B. Permission-based direct introduction

Hi [name], you mentioned [specific process problem] in [context]. I am developing a small documentation service: one process map, a demo input/output table and exception rules. My examples are fictional learning exercises. Would a short outline for that process be useful?

### C. Response to an invitation to share work

Thanks for inviting me to share. My [enquiry routing] example is a fictional [design exercise/local demo—choose the true status]. It covers [specific rules] and [exceptions]. [State actual tests, or say testing is pending.] Which part would you want clarified for your own process?

### D. Scoped proposal after discovery

Based on your description, I suggest documenting [one process] with [input], [up to three rules] and [output]. I would deliver the field list, synthetic examples, exception rules and acceptance checklist by [agreed date], with one wording revision. This excludes implementation and account access. The agreed fee is [amount, or free if agreed]. Does that match the written scope we discussed?

### E. One respectful follow-up

Hi [name], following up once on the outline for [process]. One question that may help either way: who should handle records that fail validation? If the timing is not right, no reply is needed and I will leave it here.

## 7. Three documentation exercises and model answers

### Exercise 1: write a bounded offer

Task: Using case A, write a problem statement, a one-sentence offer, deliverables, exclusions and one acceptance criterion. Do not claim time savings.

Model answer: “In a fictional studio, a coordinator assigns enquiries manually. I will document a three-category enquiry routing design using synthetic records. Deliverables are the field list, rules, five-record expected-output table and review policy. Live inbox connections, sending messages and deployment are excluded. Acceptance means each sample row has an unambiguous route, duplicate status or review reason.”

Self-check: Can someone tell exactly what they receive and what you will not implement? Is every claimed result observable in the document?

### Exercise 2: create a test log without inventing tests

Task: For case B, record expected behaviour, actual behaviour and status for a normal case, date boundary and invalid input. Fill actual behaviour only after running your own demo.

Model answer before execution:

| Check | Input | Expected | Actual | Status |
|---|---|---|---|---|
| Overdue unpaid | I201 fixture | eligible, draft only | not observed | not run |
| Seven-day boundary | I206 fixture | eligible | not observed | not run |
| Bad date | I205 fixture | review, no draft | not observed | not run |

Execution record template: date [ ]; n8n version [ ]; workflow revision [ ]; synthetic fixture [ ]; observed output [ ]; discrepancy [ ]; fix and rerun [ ]. Mark a check passed only when the actual output matches its expected behaviour. Retain failed attempts where they explain a limitation.

### Exercise 3: write a truthful portfolio entry

Task: Turn case C into a short portfolio entry and distinguish expected arithmetic from measured business impact.


Self-check: Does the opening label the work? Are expected and observed outcomes distinct? Can the reader see the next validation step?

## 8. Ready-to-use first-client preparation checklist

- Choose one of the three designs and keep its fictional label.
- Adapt the service profile to your real ability and remove unsupported claims.
- Write a one-paragraph portfolio entry; add real execution evidence only if available.
- Identify a relevant process discussion and write one useful, specific reply.
- Ask discovery questions before quoting implementation work.
- Agree scope, timing and completion criteria in writing before beginning any order.
- Record feedback: was the offer understandable, what was missing, and did the reader want the next step? Do not treat interest as willingness to pay.

---

**Need it customised, or a full version for your work?** [Message us on Telegram](https://t.me/mostafa_ideas_2050_bot?start=r_page_6), no sign-up.

**محتاج نسخة مخصصة أو كاملة لشغلك؟** [ابعتلنا على تليجرام](https://t.me/mostafa_ideas_2050_bot?start=r_page_6) من غير تسجيل.
