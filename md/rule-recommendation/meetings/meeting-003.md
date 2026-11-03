# Meeting 3 Analysis — AI Rule Recommendation Project

**Meeting 3 is a turning point.** The discussion expanded from an AI-assisted rule authoring feature into a broader recommendation platform supporting DQDL and SQL.

Three major challenges emerged:

1. **Technical scope is expanding faster than requirements are being finalized.** The team is estimating features while learning how ProDQ works.
2. **Product and engineering are not fully aligned on priorities.** Engineering wants to leverage the existing DQDL foundation, while product leadership appears increasingly interested in SQL.
3. **Frontend responsibilities are expanding.** The UI may need to support generating, reviewing, selecting, and saving dozens of recommendations, including asynchronous profiling workflows.

A positive signal: **your earlier UI proposal was explicitly acknowledged as simple, non-intrusive, and easy to use.** The group is now looking to you to help define the next interaction model.

## 1. Executive summary

| Area              | Assessment                                                    |
| ----------------- | ------------------------------------------------------------- |
| Architecture      | Direction emerging; major integration details unresolved      |
| DQDL generation   | Existing proof of concept and validation provide a head start |
| SQL generation    | Rule semantics and execution contract need investigation      |
| AI recommendation | Batch-oriented UX and profiling dependencies identified       |
| Frontend          | Concept validated; new design and implementation scope        |
| Product alignment | Unresolved DQDL-versus-SQL prioritization                     |

The meeting identified four use cases:

| Use case                            | Input                            | Output                                        |
| ----------------------------------- | -------------------------------- | --------------------------------------------- |
| Natural language to DQDL            | User-written description         | One DQDL rule                                 |
| Natural language to SQL             | User-written description         | One SQL rule condition                        |
| Metadata-driven DQDL recommendation | Column profile and logical model | Multiple DQDL recommendations                 |
| Metadata-driven SQL recommendation  | Column profile and logical model | Multiple ProDQ-compatible SQL recommendations |

The first two are **user-directed generation**. The last two are **system-directed recommendation** based on metadata. They require different backend orchestration, UX behavior, and product expectations.

## 2. Major technical decisions

### A. DQDL and ProDQ SQL have different semantics

ProDQ SQL rules are not necessarily complete SQL statements. A user provides a condition, and ProDQ generates the executable SQL around it.

Requirement: **Every customer email must be populated.**

**DQDL — passing condition:**

```text
IsComplete "email"
```

**ProDQ SQL — failure condition:**

```sql
email IS NULL
```

The team described ProDQ conditions as matching failures, while DQDL checks generally express passing expectations. System prompts cannot simply interchange the two syntaxes. A rule might look valid but reverse the business requirement.

**Recommendation:** Define a canonical representation of rule intent and an explicit mapping of pass/fail semantics for each engine. Also verify the ProDQ grammar: the team found examples involving `GROUP BY` and `HAVING`, so describing every SQL rule as a WHERE predicate is too narrow.

### B. Separate recommendation pipelines, shared platform capabilities

The team leaned toward separate retrieval and generation logic for DQDL and SQL because the systems have different sources, execution behavior, validation, and profiling APIs.

```text
                    Shared AI recommendation foundation
             User intent · Metadata · Lifecycle · Audit events
                                  |
                    +-------------+-------------+
                    |                           |
                DQDL pipeline               SQL pipeline
                - DQDL knowledge base       - SQL knowledge base
                - DQDL system prompt        - SQL system prompt
                - DQDL parser               - SQL validation
                - DQA profiling             - ProDQ profiling
                    |                           |
                    +-------------+-------------+
                                  |
                      Shared application experience
                       Review · Accept/Discard
                            Save · Metrics
```

Separate pipelines are reasonable, but they **do not require two entirely separate frontends or completely duplicated backend infrastructure**. Reuse shared operations and isolate engine-specific adapters.

### C. Validation and retries belong in the backend

The proposed flow is:

1. Receive a generation request.
2. Invoke the LLM.
3. Validate the generated rule.
4. If invalid, retry up to the configured limit (three attempts were discussed).
5. Return a validated rule or a clear failure.

The group supported keeping this orchestration in the API/backend layer, rather than in the LLM or React UI.

**Important qualification:** Syntactic validity does not establish semantic correctness. A generated rule can parse yet reference the wrong column, implement reversed failure semantics, or fail the intended quality check. Evaluation must test those risks.

## 3. Biggest frontend change: generation versus recommendation

The earlier concept generated **one rule** within the existing Rule Authoring modal. Meeting 3 introduced an experience that may recommend **20–30 or more rules** for a data subset.

These are distinct interactions:

| Experience           | User interaction                                                                            | Recommended location                                     |
| -------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| **Generate with AI** | Describe requirement, generate one rule, review, use or discard                             | Existing Rule Authoring/Editing modal                    |
| **Recommend rules**  | Analyze selected data subset, review groups of suggestions, select multiple rules, add them | Dedicated recommendation workspace within Rule Authoring |

Do **not** force batch recommendations into the existing single-rule modal. Grouped review, multi-selection, detailed previews, and asynchronous status demand more space.

### Recommended batch review UX

An illustrative view:

```text
AI Rule Recommendations
Data subset: Customer Transactions

2 of 6 rules selected                 [Select all] [Clear]

v customer_email (2 recommendations)
  [x] Email completeness              [Preview]
  [ ] Email format                    [Preview]

v account_id (2 recommendations)
  [x] Account ID completeness         [Preview]
  [ ] Account ID uniqueness           [Preview]

v transaction_amount (2 recommendations)
  [ ] Amount completeness             [Preview]
  [ ] Nonnegative amount              [Preview]

Rule preview:
  IsComplete "customer_email"

                  [Continue with 2 selected rules]
```

Recommended behavior:

- Group recommendations by column, with optional rule-type filters.
- Preview full expressions and supporting explanation without leaving the workspace.
- Support selecting, clearing, and reviewing multiple rules.
- Stage selected rules into existing rule authoring state before saving.
- Prevent duplicates and preserve user edits.
- Use an accessible grouped list/accordion rather than a complex tree unless true hierarchy is required.
- Plan for filtering and pagination/virtualization if recommendations grow substantially.

## 4. Asynchronous profiling is a substantial dependency

Recommendations depend on profiling metadata, which may take minutes or hours to produce. The team debated automatic profiling versus directing users to the existing profiling screen.

Suggested lifecycle:

```text
User requests recommendations
             |
Check profiling availability
             |
       +-----+------+
       |            |
   Available      Missing
       |            |
 Generate         Trigger/request profiling
 recommendations       |
       |          Job completes
       |            |
       |          Generate recommendations
       +-----+------+
             |
   Recommendations ready
             |
    Review, select, save
```

The group had not finalized Step Functions versus another persistent background workflow. **Do not expose that implementation choice through the frontend contract.** Give React a stable operation identifier and lifecycle states.

```ts
type RecommendationStatus =
  | "PENDING_PROFILING"
  | "PROFILING_IN_PROGRESS"
  | "GENERATING"
  | "READY"
  | "FAILED";

interface RecommendationJob {
  id: string;
  dataSubsetId: string;
  status: RecommendationStatus;
  createdAt: string;
  completedAt?: string;
  errorMessage?: string;
}
```

Also support resuming a job after a user leaves or a session expires. A two-hour profiling operation should not require keeping a modal open.

## 5. AI audit logging

The group debated logging only saved rules versus capturing both accepted and discarded generations. The final direction favored collecting user interactions and prompt/output information for product metrics.

| Event       | Meaning                                |
| ----------- | -------------------------------------- |
| `GENERATED` | Candidate produced                     |
| `ACCEPTED`  | User chose to use the candidate        |
| `DISCARDED` | User explicitly rejected the candidate |
| `SAVED`     | Rule persisted to the rule set         |

**Acceptance is not saving.** A user may accept a generated rule, modify it, and then cancel without saving.

Hypothetical funnel:

| Stage     | Candidates |
| --------- | ---------: |
| Generated |        100 |
| Accepted  |         70 |
| Saved     |         52 |

This would yield a **70% candidate acceptance rate** and **74.3% accepted-to-saved conversion**. These are illustrations, not actual project data.

Recommendations:

- Give each candidate a stable ID and link it to the saved rule, where applicable.
- Distinguish generation attempts, generated candidates, explicit discards, session abandonment, acceptance, and persistence.
- Capture retries and rule edits without inflating the acceptance metric.
- Do not interpret every discard as model failure; users may abandon tasks for other reasons.
- Agree on access controls, data retention, and permitted prompt/SQL logging due to sensitive business metadata.
- A Boolean `aiGenerated` flag can help, but is insufficient alone for complete analytics.

## 6. Unresolved dependencies

| Dependency                      | Current understanding                                                    | Risk   |
| ------------------------------- | ------------------------------------------------------------------------ | ------ |
| Data lake permissions           | Consumer account cannot yet access all required environments             | High   |
| DataHub URN/environment mapping | Environment identification exists; schema registration/access must align | High   |
| Sage integration                | Viewed as product differentiator; November 30 readiness was discussed    | High   |
| ProDQ SQL rule format           | Some examples understood, full grammar unresolved                        | High   |
| ProDQ profiling APIs            | Availability and ad hoc trigger support still need verification          | High   |
| SQL golden dataset              | Converted rules may not be end-to-end validated until December           | High   |
| Batch recommendation UI         | Grouping and multi-select discussed; API contract not finalized          | Medium |
| Retry/evaluation                | Direction established; acceptance criteria incomplete                    | Medium |

### Data lake access and environment mapping

The proposed direction uses DataHub URNs and environment-specific names such as `test_dqp_application` and `cert_dqp_application` to distinguish otherwise similar assets.

**Metadata visibility is not physical data access.** A URN identifying a CERT table does not grant the consuming AWS account permission to query it. Both registration conventions and cross-environment access must be resolved.

### SQL golden dataset

Rules reportedly converted from DQDL are not automatically a trusted golden dataset. Confirm that examples execute, behave correctly, and match expected pass/fail outcomes before using them to evaluate generation quality.

## 7. Team dynamics and delivery risks

### Product versus engineering priorities

Some engineers are concerned that separate DQDL and SQL estimates will lead product leadership to select only SQL, despite the existing DQDL foundation. Others point out—correctly—that the work is genuinely distinct.

**Recommendation:** Present a shared foundation plus two engine-specific delivery tracks, with a dependency/reuse matrix. This communicates the broader vision without hiding real differences in effort.

### Product-definition responsibility is shifting to engineers

One participant said product should provide more guidance on the UI rather than asking engineering to define everything. Engineering should influence UX, but product must own prioritization and core requirements—especially whether DQDL/SQL are user-facing choices.

### Estimates precede discovery

Participants discussed approximately **10–12 points for parts of the frontend work** and provisional broader figures around **50 points for a DQDL-related scope** and **63 for a SQL-related scope**. The transcript does not provide a reconciled total.

These should be planning placeholders, **not commitments**, given uncertainty about ProDQ profiling, SQL grammar, integration, and golden data.

## 8. Your contribution and position

The transcript is not fully speaker-labeled, so individual technical statements cannot all be attributed confidently. However, Mike was directly identified when the discussion turned to UX.

Positive signals:

1. **Your concept was recognized.** It was described as simple, non-intrusive, and easy to use.
2. **Your input was requested for SQL UX.** The team explicitly asked whether SQL generation would require separate UI changes.
3. **You were asked to decompose the frontend work.** The group wants a defensible set of stories rather than an unexplained single estimate.

These indicate the team is relying on your judgment beyond component implementation.

The risk is that **you become responsible for designing around unresolved backend and product decisions**. Make those decisions explicit before you accept implementation commitments.

## 9. Five questions to raise next

1. **Are DQDL and SQL user-selected rule types or implementation details hidden from users?** This controls navigation, authoring, validation, and persistence UX.
2. **Can a rule set contain both DQDL and SQL rules?** If so, the table and lifecycle must support mixed rule types; if not, selection occurs earlier.
3. **Is a recommendation only a draft until explicitly saved?** Selection should not necessarily mean persistence.
4. **What happens if profiling is missing or stale?** Automatic trigger, manual profiling workflow, notification, and error paths need clear requirements.
5. **What is the batch recommendation API contract?** At minimum, the UI needs candidate IDs, columns, rule types/engines, definitions, validation status, job status, and audit correlation.

The first question is especially important because the meeting debated whether showing a SQL option would steer users away from DQDL. That is fundamentally a **product strategy** decision.

## 10. Recommended frontend work breakdown

| Workstream                    | Scope                                                         |
| ----------------------------- | ------------------------------------------------------------- |
| UX discovery and API contract | Confirm engine entry points, async states, batch behavior     |
| Single-rule AI generation     | Prompt, preview, use/discard in existing modal                |
| Recommendation discovery      | Launch recommendation, detect profiling availability          |
| Async operation status        | Pending/profiling/generating/ready/failed, resume support     |
| Batch recommendation review   | Group, preview, filter, select/deselect                       |
| Rule-set integration          | Stage choices, prevent duplicates, preserve edits, persist    |
| Audit instrumentation         | Candidate correlation with use/discard/save events            |
| Tests                         | Loading/errors, partial validation failures, session recovery |

Estimate each stream once product requirements and backend contracts are aligned.

Use a shared frontend recommendation domain model—for example, a candidate with `engine: "DQDL" | "SQL"`—and engine-specific preview/validation adapters. Avoid creating two wholly separate rule-management experiences unless product requirements demand it.

## 11. Follow-up action checklist

- [ ] Document the single-rule generation versus batch recommendation journeys. **Owner:** Frontend + Product.
- [ ] Confirm DQDL/SQL rule-type choices and whether mixed rule sets are supported. **Owner:** Product + Architecture.
- [ ] Agree on batch recommendation and asynchronous job API contracts. **Owner:** Frontend + Backend.
- [ ] Verify ProDQ rule syntax, profiling API capabilities, and validated SQL examples. **Owner:** Backend + ProDQ.
- [ ] Decompose frontend work into reviewable, separately estimable stories. **Owner:** Frontend.
- [ ] Define candidate-level audit events and persistence semantics. **Owner:** Frontend + Backend.

## 12. Evolution across the three meetings

| Meeting   | Focus              | Evolution                                                                                     |
| --------- | ------------------ | --------------------------------------------------------------------------------------------- |
| Meeting 1 | Feature discovery  | Explore natural-language generation, validation, and user-friendly authoring                  |
| Meeting 2 | MVP refinement     | Clarify use/discard flow, audits, retry behavior, and the single-rule modal                   |
| Meeting 3 | Platform expansion | Add ProDQ SQL, metadata-driven batches, Sage, profiling, async jobs, and broader architecture |

**Overall assessment:** Meeting 3 combines a relatively achievable AI-assisted authoring MVP with a more ambitious metadata-driven recommendation platform. They are related but should **not** be treated as one delivery commitment.

**Recommended direction:** Preserve single-rule **Generate with AI** as an independently deliverable experience. Design batch recommendation as a separate capability that shares components and audit infrastructure. Do not commit the batch UX estimate until product and backend agree on engine semantics, profiling lifecycle, and API contracts.

That puts frontend engineering in a position to shape the system—not merely compensate for decisions made elsewhere.
