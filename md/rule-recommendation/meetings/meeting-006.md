Meeting 6 Analysis — AI Rule Recommendation Project

Meeting: 6 Date: November 2026 Project: AI Rule Recommendation Target UAT Date: December 18, 2026 Primary Focus: Architecture simplification, scope, UI/UX ownership, profiling workflows, and team responsibilities

1. Executive Summary

Meeting 6 represented a significant turning point for the AI Rule Recommendation project.

The team transitioned from brainstorming individual capabilities toward defining delivery commitments, architecture, UI responsibilities, and implementation priorities.

Key Takeaways

* UI Ownership Confirmed
    * Mike is responsible for UI/UX design and frontend implementation.
    * Additional engineers should coordinate any UI contributions through Mike.
    * The team acknowledged that another dedicated UI engineer is unnecessary.
* Five AI Use Cases Identified
    * English → DQDL
    * English → SQL
    * SQL → Spark SQL
    * Profile → DQDL
    * Profile → SQL
* Scope Prioritized
    * Three core use cases are committed.
    * Profile → DQDL is a stretch goal.
    * Profile → SQL is deferred to a future iteration.
* Architecture Simplified
    * The team favored removing automatic profile orchestration from the initial user workflow.
    * Users will initiate profiling before requesting AI recommendations.
    * Profiling completion and failure notifications were discussed.
* December 18 Target Clarified
    * The target is UAT readiness, not production deployment.
    * Production deployment depends on additional approvals and infrastructure readiness.

2. Five AI Use Cases

#	Use Case	Description	Priority
1	English → DQDL	Convert natural language into data quality rules	Core
2	English → SQL	Generate SQL from natural-language descriptions	Core
3	SQL → Spark SQL	Convert SQL expressions into Spark SQL	Core
4	Profile → DQDL	Recommend DQDL rules using dataset profiling	Stretch
5	Profile → SQL	Recommend SQL rules using dataset profiling	Future iteration

2.1 English → DQDL

Users describe a data quality requirement using natural language.

Example input:

Check that customer_email is not null.

Generated output:

Rules = [
    IsComplete "customer_email"
]

The workflow includes:

* User enters a natural-language description.
* Frontend submits the description to the generation API.
* Backend generates DQDL.
* Backend validates the expression.
* Frontend displays the generated rule.
* User accepts or rejects the recommendation.

2.2 English → SQL

Users provide natural-language descriptions that are converted into SQL expressions.

Example input:

Find customers whose account balances exceed 10000.

Generated output:

SELECT *
FROM customers
WHERE account_balance > 10000;

The frontend interaction can share components with English → DQDL.

However, SQL validation differs from DQDL validation.

2.3 SQL → Spark SQL

Users provide SQL expressions that are converted into Spark-compatible SQL.

Example input:

SELECT TOP 10 *
FROM customers;

Generated output:

SELECT *
FROM customers
LIMIT 10;

The UI should distinguish between source and target SQL dialects.

2.4 Profile → DQDL

This capability uses profiling metadata to recommend data quality rules.

The system may analyze:

* Column names
* Data types
* Null percentages
* Distinct-value counts
* Minimum and maximum values
* Logical data models
* Physical dataset metadata
* Existing rule examples

The backend retrieves similar examples through RAG and generates DQDL recommendations.

Users review the recommendations and select which rules to accept.

2.5 Profile → SQL

This capability is similar to Profile → DQDL but generates SQL-based rules.

A significant dependency is the external profiling platform referred to in the transcript as ProdIQ/Prodique.

Because this dependency is not fully controlled by the team, the use case is excluded from the initial delivery commitment.

3. Architecture Decision: Simplify Profiling Orchestration

3.1 Original Architecture

The original approach involved automatically orchestrating profiling and rule generation.

flowchart TD
    A[User Requests AI Recommendation]
    B{Profile Exists?}
    C[Trigger Profiling Job]
    D[Wait for Profile Completion]
    E[Retrieve Profile]
    F[Retrieve Logical and Physical Metadata]
    G[Perform RAG Similarity Search]
    H[Generate AI Rules]
    I[Validate Generated Rules]
    J[Display Recommendations]
    A --> B
    B -->|No| C
    C --> D
    D --> E
    B -->|Yes| E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J

Problems With the Original Architecture

* Requires long-running workflow orchestration.
* Requires asynchronous job tracking.
* Introduces more failure scenarios.
* Depends on multiple backend systems.
* Adds infrastructure complexity.
* Makes debugging more difficult.
* Increases the risk of missing the December deadline.

Profiling could take several minutes or potentially around 20 minutes depending on the dataset.

3.2 Revised Architecture

The team favored a simpler approach in which profiling is an explicit prerequisite.

flowchart TD
    A[User Selects Distribution]
    B{Profile Exists?}
    C[Show Generate Profile Action]
    D[User Starts Profiling]
    E[Profiling Job Runs]
    F{Profiling Successful?}
    G[Notify User of Completion]
    H[Notify User of Failure]
    I[Show Actionable Error]
    J[Enable AI Rule Recommendation]
    K[Generate Recommended Rules]
    L[Review Recommendations]
    M[Accept and Save Rules]
    A --> B
    B -->|No| C
    B -->|Yes| J
    C --> D
    D --> E
    E --> F
    F -->|Yes| G
    F -->|No| H
    H --> I
    G --> J
    J --> K
    K --> L
    L --> M

Why the Revised Architecture Is Better for the MVP

* Separates profiling from AI generation.
* Reduces workflow complexity.
* Simplifies frontend state management.
* Makes failures easier to isolate.
* Allows users to leave while profiling runs.
* Improves the likelihood of reaching UAT by December 18.

Important: This removes automatic end-to-end orchestration from the initial workflow. It does not eliminate backend coordination for profiling, RAG retrieval, or AI generation.

4. Profiling Architecture

The two profile-driven use cases depend on different backend systems.

Profile → DQDL

Uses AWS Glue for profiling.

flowchart TD
    A[User Selects Distribution]
    B{Profile Exists?}
    C[Trigger AWS Glue Job]
    D[Wait for Completion]
    E[Notify User]
    F[Retrieve Profile Metadata]
    G[RAG Similarity Search]
    H[Generate DQDL Recommendations]
    A --> B
    B -->|No| C
    C --> D
    D --> E
    E --> F
    B -->|Yes| F
    F --> G
    G --> H

Profile → SQL

Uses the external profiling platform.

flowchart TD
    A[User Selects Distribution]
    B{External Profile Exists?}
    C[Request Profiling]
    D[External Platform Executes Job]
    E[Receive Completion Status]
    F[Notify User]
    G[Retrieve Profile]
    H[Generate SQL Recommendations]
    A --> B
    B -->|No| C
    C --> D
    D --> E
    E --> F
    F --> G
    B -->|Yes| G
    G --> H

Recommended Frontend Abstraction

The UI should not need separate profiling components for each provider.

type ProfilingStatus =
  | "NOT_STARTED"
  | "QUEUED"
  | "RUNNING"
  | "COMPLETED"
  | "FAILED";
type ProfilingProvider = "GLUE" | "PRODIQ";
interface ProfilingJob {
  jobId: string;
  distributionId: string;
  status: ProfilingStatus;
  provider: ProfilingProvider;
  errorMessage?: string;
  completedAt?: string;
}

This is a proposed frontend model, not a finalized backend contract.

5. RAG-Based Rule Recommendation

The meeting provided more detail about how profile-based recommendations are expected to work.

RAG Workflow

flowchart TD
    A[Distribution Profile]
    B[Logical Data Model]
    C[Physical Dataset Metadata]
    D[Recommendation Service]
    E[Similarity Search]
    F[Relevant Rule Examples]
    G[LLM Prompt Context]
    H[Generate Recommended Rules]
    I[Validate Rules]
    J[Return Recommendations]
    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    F --> G
    D --> G
    G --> H
    H --> I
    I --> J

Key Architectural Components

Component	Responsibility
Profiling service	Generate statistical metadata
Metadata service	Retrieve logical and physical metadata
RAG knowledge base	Store and retrieve relevant examples
Similarity search	Find similar datasets or rules
LLM service	Generate recommendations
Validation service	Verify generated expressions
Frontend	Present recommendations and capture decisions

Important Observation

Profile-based recommendations are more complex than natural-language generation.

Natural-language generation primarily transforms an input description into an expression.

Profile-driven recommendation combines:

* Dataset analysis
* Metadata retrieval
* Similarity search
* Prompt construction
* AI generation
* Rule validation
* Recommendation selection

This justifies a different UI experience.

6. UI/UX Architecture Recommendation

The five use cases should not require five independent frontend implementations.

I recommend three reusable interaction patterns.

Pattern A: Natural-Language Generation

Used by:

* English → DQDL
* English → SQL

UI elements:

* Natural-language prompt input
* Generate action
* Loading state
* Generated expression editor
* Validation feedback
* Accept/reject actions

Pattern B: SQL Conversion

Used by:

- SQL → Spark SQL

UI elements:

* Source SQL editor
* Target dialect indicator
* Convert action
* Generated Spark SQL editor
* Validation feedback

Pattern C: Profile-Driven Recommendations

Used by:

* Profile → DQDL
* Profile → SQL

UI elements:

* Distribution selection
* Profile availability status
* Generate Profile action
* Profiling progress/status
* Recommendation list
* Multi-selection
* Accept/save actions

Proposed React Component Architecture

AiRuleWorkbench
│
├── AiUseCaseSelector
│
├── NaturalLanguageGenerator
│   ├── PromptInput
│   ├── GenerateButton
│   ├── GeneratedExpressionEditor
│   └── ValidationFeedback
│
├── SqlDialectConverter
│   ├── SourceSqlEditor
│   ├── TargetSqlEditor
│   ├── ConvertButton
│   └── ValidationFeedback
│
└── ProfileRuleRecommendations
    ├── ProfileStatus
    ├── GenerateProfileButton
    ├── ProfilingProgress
    ├── RecommendationList
    ├── RecommendationSelection
    └── AcceptRecommendations

Shared Components

Shared
├── GenerationStatus
├── ErrorFeedback
├── CodeEditor
├── UseGeneratedRuleButton
├── RecommendationCard
└── FeatureAvailability

UI Placement Considerations

The existing Rule Authoring/Editing modal is a natural location for English → DQDL.

However, not all five use cases necessarily belong in that modal.

For example:

* English → SQL may belong near SQL filter authoring.
* SQL → Spark SQL may belong within an SQL transformation experience.
* Profile-driven recommendations may require a separate panel or workflow.

The goal should be to reuse interaction patterns without forcing unrelated tasks into one modal.

7. Profiling UI States

The revised architecture requires clear profiling states.

State	Description	User Action
NOT_STARTED	No profile exists	Generate Profile
QUEUED	Job submitted	View status
RUNNING	Profiling underway	Wait or leave
COMPLETED	Profile available	Recommend Rules
FAILED	Profiling unsuccessful	Review error and retry

Example TypeScript State

interface ProfileState {
  status: ProfilingStatus;
  jobId?: string;
  errorMessage?: string;
}

Design Requirement

Profiling status must persist independently of the React component lifecycle.

If a user leaves and returns, the UI needs to retrieve the current profiling state from the backend.

The frontend should not be responsible for maintaining long-running job state in memory.

8. Notifications

The team favored notifications when profiling completes or fails.

Notification Events

Event	Notification
Profile completed	Profile ready; recommendations available
Profile failed	Profiling unsuccessful; corrective action required

The delivery mechanism was discussed in terms of email, but the final implementation was not established.

Error Scenarios

Potential failures include:

* Dataset contains no rows.
* SQL filter returns no matching records.
* Invalid SQL expression.
* Profiling infrastructure failure.
* External profiling service unavailable.

Recommended Error Messages

Empty dataset

Profiling could not complete because no records were found.
Verify that the selected distribution contains data
and review the SQL filter.

Invalid SQL

The SQL expression could not be validated.
Review the query syntax and selected columns
before retrying.

Infrastructure failure

The profiling job could not complete.
Please retry. If the issue continues,
contact support.

Key Principle

The frontend should display actionable errors rather than generic failure notifications.

9. AI-Generated Rule Editing and Provenance

The team raised an important question:

If a user edits an AI-generated rule, should it still count as AI-generated?

This question affects analytics, persistence, and rule classification.

Recommended Data Model

interface RuleGenerationMetadata {
  origin: "MANUAL" | "AI_GENERATED";
  wasEditedAfterGeneration: boolean;
  generationId?: string;
  originalExpression?: string;
}

Examples

Scenario	Origin	Edited
Manually authored rule	MANUAL	false
AI-generated rule accepted unchanged	AI_GENERATED	false
AI-generated rule modified by user	AI_GENERATED	true

Recommendation

Separate rule provenance from modification history.

However, editing generated recommendations should remain outside the initial scope until product and engineering agree on the desired behavior.

For UAT, accepting or discarding generated rules may be sufficient.

10. SQL Validation

The team discussed challenges around SQL validation.

Syntactic Validation

Checks whether SQL is grammatically correct.

Examples:

* Missing parentheses
* Invalid SQL keywords
* Malformed expressions
* Incorrect syntax

Semantic Validation

Checks whether SQL is meaningful against the actual data source.

Examples:

* Unknown columns
* Missing tables
* Unsupported functions
* Incorrect data types

Libraries Discussed

* JSQLParser
* SQLGlot

The team expressed concerns about incomplete coverage across SQL dialects.

Potential Validation Solution

The external SQL platform may expose an API capable of validating queries.

If it performs an EXPLAIN operation against the appropriate data source, it may detect syntax and many semantic errors.

However, the team needs to confirm how the endpoint actually validates queries.

Recommended API Contract

interface ExpressionValidationResult {
  valid: boolean;
  validationType: "SYNTAX" | "SEMANTIC" | "BOTH";
  errors: Array<{
    code: string;
    message: string;
    line?: number;
    column?: number;
  }>;
}

Open Questions

* Does validation use the target database?
* Does it support multiple SQL dialects?
* Can it validate table and column existence?
* Can it validate database-specific functions?
* What is the error response format?
* How are permissions enforced?
* Is the validation operation read-only?

11. Feature Flags and Production Deployment

The team reiterated that LLM-connected services cannot automatically be deployed to production.

Production deployment requires additional organizational approvals.

Deployment Strategy

Component	Production Strategy
Existing Rule Authoring UI	Deploy normally
AI UI functionality	Keep behind feature flag
AI generation backend	Requires approval
LLM integrations	Requires approval
Profile recommendation services	Subject to environment and approval requirements

Recommended Behavior

Feature flags should:

* Hide or disable unavailable AI functionality.
* Prevent unapproved API calls.
* Support controlled rollout.
* Allow existing rule authoring features to continue functioning.

Backend authorization and environment restrictions must also be enforced independently of frontend flags.

12. Team Responsibilities

The meeting clarified ownership as the team prepares for its new pod structure.

Area	Responsibility
UI/UX	Mike
Frontend development	Mike
English → DQDL generation	Existing backend team
English → SQL API	New engineer
SQL validation investigation	New engineer / backend
RAG ingestion	Backend team
Prompt construction	Backend team
Profile integrations	Backend team
Orchestration, if needed	Potentially new engineer
Product scope	Product team and engineering

Important Discussion

The team explicitly acknowledged:

“We don’t need a UI person. We need someone to help with the backend.”

It was also confirmed that Mike would remain responsible for the UI.

Interpretation

This is a positive ownership signal.

The team recognizes that the frontend is already covered.

However, implementation ownership and final product decision authority are different responsibilities.

The opportunity is to use frontend ownership to influence broader workflow and architectural decisions.

13. Project Timeline

November 12 — Flow Review

Expected deliverables:

* Updated diagrams for all five use cases
* Revised profiling workflow
* UI component strategy
* Integration questions
* Initial discussion of backend contracts

Mid-November — Initial Integration

Expected activities:

* English → DQDL API integration
* Shared UI component implementation
* Generation and validation feedback
* Feature flag implementation

Late November–Early December — Broader Development

Expected activities:

* English → SQL integration
* SQL → Spark SQL conversion
* Profiling status handling
* Notification integration
* Recommendation selection
* Validation and error handling

December 18 — UAT Target

The team is targeting UAT readiness by December 18, 2026.

This is not a production deployment commitment.

14. Delivery Risks

Risk	Severity	Explanation
External profiling integration	High	Depends on another system
SQL semantic validation	High	Validation approach remains uncertain
Five-use-case complexity	Medium–High	Multiple workflows must be designed
API contract readiness	Medium	Frontend integration depends on backend definitions
Profiling notifications	Medium	Asynchronous event handling is required
Rule editing/provenance	Medium	Persistence behavior unresolved
Production approvals	High	External organizational dependency

Biggest Risk

The greatest delivery risk is not necessarily frontend implementation complexity.

It is the possibility of discovering unclear or incomplete service contracts late in the development cycle.

The team should agree on API contracts before building tightly coupled integrations.

15. Decisions and Open Questions

Decisions and Agreed Direction

Topic	Outcome
Five AI use cases	Identified
Three core commitments	Communicated
Profile → DQDL	Stretch goal
Profile → SQL	Future iteration
UI ownership	Mike
Profile-first workflow	Preferred approach
Profiling completion notifications	Supported
Profiling failure notifications	Supported
December 18	UAT target

Open Questions

Question	Required Follow-Up
Where does profile readiness come from?	Define status API
How are profiling jobs tracked?	Define job lifecycle
How are notifications sent?	Define backend notification strategy
Can users edit AI-generated rules?	Product decision
How are accepted recommendations persisted?	Define persistence contract
How does SQL semantic validation work?	Investigate external validation API
What happens when profile generation fails?	Define error behavior
Where should each AI use case appear in the UI?	UX decision
What is the final production rollout plan?	Approval and deployment planning

16. Action Items for Mike

Before the Next Meeting

* Update the flow diagrams for all five AI use cases.
* Replace automatic profile orchestration with the profile-first approach.
* Include successful and failed profiling states.
* Incorporate notification behavior.
* Identify reusable UI components.
* Document API dependencies.
* Review recommendation acceptance and persistence behavior.
* Publish the diagrams in Confluence.
* Review the updated diagrams with the team.

Questions to Ask

* How does the frontend determine whether a profile exists?
* What endpoint retrieves profiling job status?
* What API contracts are ready for integration?
* Are generated recommendations persisted before acceptance?
* Does accepting a recommendation save the rule immediately?
* Can users edit recommended rules in the initial release?
* Who owns notification delivery?
* What happens if users navigate away during profiling?

17. Leadership Assessment

Meeting 6 provided several positive signals about Mike’s role in the project.

Strengths Demonstrated

1. Ownership

The team reaffirmed Mike’s responsibility for UI development.

2. Architectural Participation

Mike participated in discussions involving profiling orchestration, backend dependencies, and asynchronous workflows.

3. UX Thinking

Questions about notifications, profiling latency, and failure handling helped clarify the user journey.

4. System-Level Awareness

The discussion included how UI workflows depend on external services, feature flags, and validation mechanisms.

Areas for Improvement

Mike can strengthen his leadership position by proposing structured solutions rather than waiting for implementation requirements.

For example:

I’ve organized the five use cases into three reusable UI patterns. For profile-based recommendations, I propose a separate profiling prerequisite with persistent job status and completion notifications. Before implementation, we need to agree on the profile status API and recommendation acceptance behavior.

This demonstrates product thinking, architectural awareness, and ownership.

18. Overall Assessment

Meeting Effectiveness: 8/10

What Went Well

* Clearer delivery scope
* Confirmed UI ownership
* Simplified profiling architecture
* Identified critical dependencies
* Established a UAT target
* Aligned on the need for reusable UI components

What Needs Improvement

* API contracts remain undefined.
* Notification implementation is unclear.
* Recommendation persistence is unresolved.
* SQL semantic validation needs investigation.
* Editing AI-generated recommendations is undecided.
* Some architecture decisions still require product confirmation.

19. Final Recommendation

The highest-value next step is to create a unified UX specification covering all five AI use cases.

That specification should include:

* User journey diagrams
* Screen and component responsibilities
* Profiling state transitions
* Backend API contracts
* Error and loading states
* Recommendation acceptance behavior
* Feature flag requirements
* Delivery priorities

Prioritize the three committed use cases. Design Profile → DQDL as a future-compatible extension, and avoid allowing Profile → SQL to expand the initial implementation scope.

The broader opportunity is to demonstrate that frontend engineering contributes to system architecture, not just presentation.

Meeting 6 provides a strong foundation for taking that ownership.