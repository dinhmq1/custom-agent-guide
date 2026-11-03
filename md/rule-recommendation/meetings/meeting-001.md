# DQC AI Rule Recommendation — Meeting 1 Analysis

**Date:** October 26, 2026  
**Meeting:** Initial technical planning  
**Source:** Meeting notes
**Project:** AI-Powered Data Quality Rule Recommendation

---

## 1. Executive Summary

The meeting focused on introducing AI-powered rule generation into the existing Data Quality Rule Authoring application while establishing the foundation for a more sophisticated recommendation engine using RAG, OpenSearch, data profiling, and logical data models.

Two distinct capabilities were discussed:

1. **Natural Language to DQDL Generation:** Users describe data quality requirements in plain English, and an LLM generates the corresponding Data Quality Definition Language (DQDL).
2. **Context-Aware Rule Recommendation:** The system examines existing rules, column profiles, and logical metadata to recommend appropriate rules for new datasets.

The first capability is suitable for a small MVP. The second requires additional architecture, data integration, and operational work.

The team leaned toward delivering a demonstrable first version within approximately three weeks.

### Overall Assessment

The MVP is reasonably well-defined, but the distinction between a working demonstration and a production-ready AI service needs to be made explicit.

**Primary risk:** Scope expansion beyond what can realistically be delivered within three weeks.

---

## 2. Meeting Discussion and Emerging Decisions

| Area              | Direction                                                    |
| ----------------- | ------------------------------------------------------------ |
| MVP functionality | Generate one DQDL rule from a natural-language description   |
| User interface    | Introduce an AI generation action in Rule Authoring          |
| User control      | Allow users to review, edit, and save generated rules        |
| Backend           | Call an LLM through an AI service                            |
| Validation        | Validate DQDL and retry generation when validation fails     |
| Retry limit       | Approximately three attempts discussed                       |
| Architecture      | Prefer a separate Rule Recommendation API                    |
| RAG               | Ingest existing rules and profiling metadata into OpenSearch |
| Logical metadata  | Investigate Sage/Data Hub integration                        |
| Timeline          | Approximately three weeks for MVP/UAT                        |
| Production        | Additional governance, deployment, and security requirements |

**Important:** These represent emerging directions, not necessarily finalized architectural decisions.

---

## 3. Proposed System Architecture

### 3.1 Capability A: Natural Language to DQDL (MVP)

```text
┌─────────────────────────────┐
│ React Rule Authoring UI     │
│                             │
│ User enters description     │
│ and clicks Generate with AI │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Rule Recommendation API     │
│                             │
│ Generation orchestration    │
│ Validation coordination     │
│ Error handling              │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ Core AI Service / LLM       │
│                             │
│ Generate DQDL               │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│ DQDL Validation API         │
│                             │
│ Parse generated DQDL        │
│ Return validation errors    │
└──────────────┬──────────────┘
               │
          Valid output?
           /       \
         No         Yes
         │           │
         ▼           ▼
    Retry LLM    Return DQDL
    (bounded)        │
                     ▼
              User reviews,
              edits and saves
```

### MVP Characteristics

- Does not require RAG.
- Does not require vector embeddings.
- Does not require logical data model integration.
- Reuses existing rule validation and saving workflows.
- Keeps AI generation separate from committing changes.

### 3.2 Capability B: Context-Aware Rule Recommendation

```text
Existing DQ Rules ────────────┐
                             │
Column Profiling Data ────────┤
                             │
Logical Data Models ──────────┤
                             │
Column Metadata ──────────────┘
              │
              ▼
    Batch Ingestion Pipeline
              │
              ▼
    Feature Engineering
              │
              ▼
    Embedding Generation
              │
              ▼
    OpenSearch Knowledge Base
              │
              ▼
      Similarity Search
              │
              ▼
    Retrieved Rule Context
              │
              ▼
        LLM Generation
              │
              ▼
    Recommended DQDL Rules
              │
              ▼
       User Review / Save
```

### Additional Runtime Dependencies

For a new dataset, the recommendation engine may need to:

1. Check whether profiling information exists.
2. Trigger profiling if necessary.
3. Retrieve logical metadata.
4. Transform metadata into a searchable representation.
5. Search the knowledge base.
6. Construct an LLM prompt using retrieved context.
7. Generate and validate recommendations.
8. Notify the user if asynchronous processing is required.

This is significantly more complex than the MVP generation workflow.

---

## 4. Frontend UX Analysis

### 4.1 The Description Field Problem

One of the most important discussions concerned whether the existing rule description field should also serve as the AI generation prompt.

These fields serve different purposes.

| Existing Rule Description | AI Generation Prompt          |
| ------------------------- | ----------------------------- |
| Describes a saved rule    | Instructs AI to create a rule |
| Persistent metadata       | Potentially temporary input   |
| Primarily descriptive     | Primarily instructional       |
| Used for documentation    | Used for code generation      |

**Example rule description:**

> Customer email must be present.

**Example AI prompt:**

> Generate a DQDL rule ensuring customer_email is not null and does not contain empty strings.

Reusing the same field creates potential ambiguity.

### Risks

- Users may not understand whether they are documenting a rule or requesting generation.
- Editing a description could unintentionally affect regeneration.
- Generated DQDL could overwrite manually authored code.
- Existing rules could be modified unexpectedly.
- The application could lose user work during regeneration.

### 4.2 Recommended UX

Introduce a distinct **Generate with AI** mode within the existing rule definition area.

#### Manual Mode

- Display the existing DQDL editor.
- Preserve current editing functionality.
- Preserve existing validation and saving workflows.

#### AI Mode

- Display a dedicated natural-language prompt field.
- Provide a Generate Rule action.
- Show loading and error states.
- Display generated DQDL in a preview.
- Allow users to apply, discard, or regenerate the result.

#### Suggested Interaction

```text
Rule Definition
────────────────────────────────────

[ Write DQDL ] [ Generate with AI ]

Describe the rule you want to create:

┌──────────────────────────────────┐
│ Ensure customer_email is not     │
│ null and contains valid values.  │
└──────────────────────────────────┘

                   [ Generate Rule ]

Generated DQDL Preview:

┌──────────────────────────────────┐
│ Rules = [                        │
│   IsComplete "customer_email"    │
│ ]                                │
└──────────────────────────────────┘

       [ Discard ] [ Use This Rule ]
```

This example is illustrative; actual generated rules must be validated.

### UX Design Principles

1. AI generation should be optional.
2. Manual rule authoring must remain available.
3. Generated output should not silently overwrite existing code.
4. Users should explicitly apply generated rules.
5. The prompt should remain conceptually separate from rule metadata.
6. Errors should be actionable and understandable.
7. The UI should clearly indicate when AI is being used.

### Recommended State Model

```text
MANUAL_EDITING
      │
      ▼
AI_PROMPT_EDITING
      │
      ▼
GENERATING
   ┌──┴─────────┐
   ▼            ▼
PREVIEW       ERROR
   │            │
   ├── Apply    └── Retry / Edit Prompt
   │
   ├── Discard
   │
   └── Regenerate
```

Generated DQDL should remain temporary until the user explicitly accepts it.

---

## 5. Backend Architecture Concerns

### 5.1 Authentication and Authorization

The team discussed security for the new AI service, including IAM roles and onboarding.

Some security work was suggested for a later phase.

**Assessment:** Deferring production security requirements is acceptable only for an appropriately isolated proof of concept using approved data.

Before production deployment, define:

- Which users and services can invoke the API.
- How service-to-service authentication works.
- What data may be submitted to the LLM.
- How authorization is enforced.
- Whether prompts and generated content are logged.
- How sensitive metadata is protected.

### 5.2 LLM Validation and Retry Loop

The proposed approach is:

1. Generate DQDL using the LLM.
2. Submit DQDL to the validation API.
3. If invalid, retrieve structured validation errors.
4. Feed those errors back to the LLM.
5. Retry up to a configured limit.
6. Return valid DQDL or a controlled failure.

This is a reasonable architecture.

However, validation must distinguish between:

**Syntactic correctness**

Does the generated DQDL parse successfully?

**Semantic correctness**

Does the generated rule actually implement the user's requirement?

A syntactically valid rule may still check the wrong column or implement incorrect logic.

#### Recommendations

- Use bounded retries.
- Define request timeouts.
- Return structured error responses.
- Preserve correlation IDs.
- Track generation attempts.
- Log validation failure categories.
- Avoid returning unvalidated DQDL as successful output.

### 5.3 Knowledge Base Freshness

The team discussed periodically ingesting existing rules and metadata, potentially every one or two weeks.

Questions remain:

- How are updated rules handled?
- How are deleted rules removed?
- How is stale metadata detected?
- What triggers re-ingestion?
- Can individual documents be updated?
- How are ingestion failures recovered?

A periodic full refresh may work initially, but freshness requirements should be documented.

### 5.4 Index Lifecycle

The generic AI service reportedly supports:

- Knowledge-base creation.
- Knowledge-base specifications.
- Ingestion job creation.
- Document loading.
- Ingestion statistics.
- Index promotion.
- Previous index retention.

The meeting included discussion of OpenSearch aliases versus storing the active index name in a database.

Both approaches are possible, but the selected design must support:

- Atomic promotion.
- Concurrent searches during promotion.
- Rollback.
- Index retention.
- Failure recovery.
- Consistent active-index resolution.

Matching document counts is useful but insufficient as the sole ingestion validation criterion.

### 5.5 Document Granularity

The team described representing a column and its associated rules, profile, and logical metadata as a document.

This is plausible, but alternative structures should be evaluated.

For example:

- One document per column.
- One document per rule.
- One document per rule-column association.

The correct choice depends on retrieval relevance, filtering requirements, traceability, and update frequency.

---

## 6. Service Boundaries and Technical Debt

The meeting highlighted that the existing Rule Authoring application already contains responsibilities beyond rule authoring.

Examples discussed included:

- Profiling.
- Anomaly-related functionality.
- Rule authoring.
- Potential AI integration.

The team recognized that this creates tight coupling and makes maintenance harder.

### Recommended Service Responsibilities

| Component               | Responsibility                                 |
| ----------------------- | ---------------------------------------------- |
| Rule Authoring UI       | User interaction, editing, preview, save       |
| Rule Authoring API      | Rule lifecycle and persistence                 |
| Rule Recommendation API | AI generation and recommendation orchestration |
| Core AI Service         | LLM and knowledge-base capabilities            |
| Profiling Service       | Profiling jobs and results                     |
| Metadata Integration    | Logical model retrieval                        |
| DQDL Validation API     | Deterministic rule parsing and validation      |

Avoid moving every new AI-related responsibility into a single catch-all service.

---

## 7. MVP Scope and Delivery Strategy

The three-week timeline is plausible only with a constrained scope.

### Must Have

- Natural-language to DQDL generation.
- Backend generation endpoint.
- Validation and bounded retries.
- Frontend AI generation interaction.
- Preview and explicit apply.
- Basic error handling.
- Logging and observability.
- A demonstrable end-to-end workflow.

### Separate Workstream

- Profile-based knowledge-base ingestion.
- OpenSearch indexing.
- Embedding generation.
- Retrieval evaluation.
- Sage/Data Hub discovery.

### Later Iterations

- Context-aware recommendations.
- Asynchronous profiling.
- Notifications.
- Advanced recommendation ranking.
- Full production rollout.

### Important Distinction

A successful proof of concept is not equivalent to a production-ready system.

Production release requires approved security, governance, reliability, deployment, and operational controls.

---

## 8. Missing AI Evaluation Strategy

The meeting did not clearly establish measurable AI quality criteria.

A useful evaluation framework should include:

| Metric                  | Purpose                                              |
| ----------------------- | ---------------------------------------------------- |
| Generation success rate | How often the LLM returns a usable result            |
| Syntax validation rate  | How often generated DQDL parses                      |
| Semantic correctness    | Whether the rule implements the intended requirement |
| Retry rate              | How frequently generation requires correction        |
| Failure rate            | How often generation cannot complete                 |
| Latency                 | Time required to generate a result                   |
| User acceptance rate    | How often users apply suggested rules                |
| Manual edit frequency   | How often users modify generated output              |

### Suggested Test Dataset

Create representative prompts covering:

- Null checks.
- Uniqueness checks.
- Data type constraints.
- Range checks.
- Pattern matching.
- Cross-column comparisons.
- Invalid or ambiguous requirements.
- Unsupported requests.

The team should define expected outputs or acceptance criteria before evaluating results.

---

## 9. Team Dynamics and Leadership Opportunities

The meeting included a direct request for Mike to develop UX prototypes for the AI feature.

This creates an opportunity to influence the solution beyond implementation.

### Positive Contributions

**Questioning field semantics**

Raising concerns about repurposing the description field demonstrated attention to product behavior and data semantics.

**Protecting existing user work**

The discussion about editing existing rules highlighted the need to prevent accidental overwrites.

**Understanding backend integration**

Questions about the API flow showed an effort to connect frontend behavior with system architecture.

### Opportunity for Greater Technical Ownership

Instead of presenting only a UI mockup, bring a compact design proposal containing:

1. UX interaction flows.
2. Frontend state transitions.
3. API request and response contracts.
4. Error handling.
5. Overwrite protection.
6. Acceptance criteria.

This positions frontend engineering as a participant in architectural design rather than merely a consumer of backend decisions.

---

## 10. Questions for the Next Meeting

### Question 1

**Is AI generation a temporary suggestion or a direct modification of the rule definition?**

Recommendation: Treat it as a temporary suggestion until the user explicitly applies it.

### Question 2

**Should the AI prompt be separate from the existing rule description?**

Recommendation: Yes. The fields have different semantic purposes.

### Question 3

**What should the Rule Recommendation API return?**

Consider:

- Generated DQDL.
- Validation status.
- Error details.
- Generation metadata.
- Correlation ID.

### Question 4

**How will the MVP be evaluated?**

Define success in terms of both syntactic validity and semantic correctness.

### Question 5

**Which architecture is intended to survive beyond the POC?**

Clarify whether the new Rule Recommendation API is the intended long-term orchestration layer.

---

## 11. Recommended Action Plan

### Before the Next Meeting

- [ ] Create two UX concepts: inline AI mode and compact generation action.
- [ ] Define frontend states for generation, preview, apply, discard, and failure.
- [ ] Draft the Rule Recommendation API contract.
- [ ] Define how existing DQDL is protected from accidental overwrites.
- [ ] Align on the three-week MVP scope.
- [ ] Identify which RAG tasks belong to a separate workstream.
- [ ] Propose basic AI evaluation criteria.

### Suggested Technical Deliverables

**UX proposal**

A concise comparison of interaction patterns and their tradeoffs.

**API contract**

A proposed request/response specification for natural-language DQDL generation.

**State model**

An explicit description of how generated DQDL is stored temporarily and applied.

**Acceptance criteria**

A measurable definition of successful AI generation.

---

## 12. Final Assessment

This was a productive discovery and planning meeting, but it exposed a common engineering risk: the team has a reasonably clear idea of the demonstration it wants while the long-term service boundaries and operational requirements are still being negotiated.

The strongest strategy is to support the three-week MVP while keeping the scope deliberately small, testable, and architecturally coherent.

For Mike, the opportunity is to move beyond owning the React implementation and help define:

- How users interact with AI safely.
- How the API supports the interaction.
- How generated results are validated.
- How existing work is protected.
- How the team measures whether the feature actually works.

**Key takeaway:** The frontend is not simply where AI output is displayed. It is where AI-generated suggestions become trusted, user-approved changes to data quality rules.

---

_Analysis based on the supplied meeting transcript. Speaker attribution and some technical terms may be uncertain because the source is an imperfect speech-to-text transcript._
