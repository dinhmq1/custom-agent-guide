Here is the complete Meeting 9 analysis in Markdown format, ready to copy and paste into VS Code, GitHub, or Confluence.

Meeting 9 — AI Rule Recommendation Project

Date: November 17, 2026
Focus: Architecture Documentation, Liquibase Integration, AI Demonstration, and Sprint Planning
Project: AI-Powered Data Quality Rule Recommendation

⸻

1. Executive Summary

Meeting 9 focused on the engineering infrastructure, architecture governance, and integration requirements needed to move the AI Rule Recommendation project toward production readiness.

Unlike previous meetings, which concentrated on AI rule generation, profiling, and database design, this meeting focused on the operational and architectural requirements surrounding the AI applications.

Key Topics

1. High-Level Design Documentation (HLDD)
    * Create HLDD documentation for two AI applications.
    * Register the applications in Harmony.
    * Generate architecture documentation using an approved documentation skill.
    * Obtain architecture approval from Shika.
2. Liquibase Integration
    * Determine how to execute database migrations for Python-based AI Services.
    * Avoid circular dependencies between applications.
    * Investigate standalone deployment and migration strategies.
3. Integrated AI Demonstration
    * Postpone the planned demonstration.
    * Demonstrate English-to-SQL, English-to-DQDL, and RAG together.
    * Complete additional testing and optimization.
4. Sprint Backlog
    * Create stories for HLDD documentation and Liquibase integration.
    * Organize remaining engineering tasks.

Overall Assessment

The project is transitioning from individual AI capability development toward system integration, architecture governance, and enterprise deployment readiness.

The team must now ensure that AI generation, retrieval, database schemas, and deployment processes operate together reliably.

⸻

2. Key Decisions

Area	Decision	Status
Architecture	Create HLDD documentation for two AI applications	Assigned
Harmony	Verify registration of both applications	Pending
Architecture Governance	Obtain Shika’s approval	Required
AI Demo	Avoid demonstrating English-to-SQL independently	Agreed
RAG	Wait for an initial working implementation	In Progress
Liquibase	Research migration execution for Python AI Services	Open
Dependencies	Avoid circular application dependencies	Constraint
Sprint Planning	Create HLDD and Liquibase stories	Agreed

Important: The team has not selected a final Liquibase deployment strategy.

⸻

3. High-Level Design Documentation (HLDD)

3.1 Discussion

An engineer, apparently Irish/Avish, was assigned to coordinate with James on architecture documentation.

James is responsible for ensuring that the two AI applications are represented in Harmony.

The assigned engineer will then use a documentation-generation skill shared by Mike Marcos to help create the required documentation.

The documentation will undergo internal review before being submitted to Shika for approval.

3.2 Proposed Workflow

flowchart TD
    A["Register AI Applications in Harmony"]
    B["Verify Application Metadata"]
    C["Use Documentation Generation Skill"]
    D["Generate HLDD Documentation"]
    E["Generate Confluence Documentation"]
    F["Internal Technical Review"]
    G["Submit to Shika"]
    H{"Approved?"}
    I["Address Review Feedback"]
    J["Architecture Approved"]
    A --> B
    B --> C
    C --> D
    C --> E
    D --> F
    E --> F
    F --> G
    G --> H
    H -->|No| I
    I --> F
    H -->|Yes| J

3.3 Recommended Documentation Contents

The HLDD should describe:

* System architecture and application boundaries.
* API contracts and service interactions.
* Application dependencies.
* Database schemas and ownership.
* AI inference and prompt-management workflows.
* RAG architecture.
* Authentication and authorization.
* Deployment topology.
* Sequence diagrams.
* Error handling.
* Observability and monitoring.

3.4 Architecture Governance

Shika is involved in architecture review and approval.

The team indicated that architecture reviewers have rigorous design expectations.

Recommendation: Engage architecture reviewers early rather than waiting until the system is fully implemented.

3.5 AI-Generated Documentation

An engineer demonstrated documentation created using Copilot.

The generated documentation included:

* Architecture overviews.
* High-level system designs.
* Sequence diagrams.
* Application interaction flows.

These materials may provide a starting point for the required HLDD.

However, AI-generated documentation must be verified against the actual implementation.

Key takeaway: Documentation quality should be measured by technical accuracy, not visual presentation.

⸻

4. Liquibase Integration for AI Services

4.1 Current Architecture

The team currently uses Liquibase in Java applications.

Database migrations run when the Java application starts.

However, AI Services is implemented in Python.

AI Services also uses its own database schema, which contains system prompts and other service-related data.

The team needs a mechanism to deploy these database changes consistently.

4.2 Identified Problem

A proposed approach was to execute Liquibase migrations through the DQP AI application.

However, the team identified a potential dependency problem:

* DQP AI depends on AI Services.
* AI Services requires database schema initialization.
* Triggering AI Services migrations through DQP AI could introduce circular deployment dependencies.

The exact dependency cycle needs to be verified.

4.3 Existing Application Relationship

flowchart TD
    UI["React UI"]
    DQP["DQP AI Application"]
    AIS["AI Services - Python"]
    DB[("AI Services Database")]
    UI --> DQP
    DQP --> AIS
    AIS --> DB

This diagram is a simplified conceptual representation of the application relationship, not a verified production architecture.

4.4 Migration Options

Option	Advantages	Disadvantages
Java Application Startup	Existing implementation pattern	Coupled to Java application lifecycle
Python Application Startup	Migration execution near schema owner	Potential concurrency and deployment issues
DQP AI Application	Reuses existing Java tooling	Creates cross-service coupling
Standalone Liquibase Job	Independent and language-agnostic	Requires deployment configuration
CI/CD Migration Stage	Explicit deployment ordering	Requires pipeline integration

4.5 Recommended Architecture

Use an independent Liquibase migration job within the deployment pipeline.

flowchart TD
    A["CI/CD Pipeline"]
    B["Build and Validate"]
    C["Execute Liquibase Migrations"]
    D{"Migration Successful?"}
    E["Update AI Services Database"]
    F["Deploy AI Services"]
    G["Deploy DQP AI Application"]
    H["Stop Deployment and Report Failure"]
    A --> B
    B --> C
    C --> D
    D -->|Yes| E
    E --> F
    F --> G
    D -->|No| H

4.6 Benefits

This approach provides:

1. Separation of Concerns
    * Database migrations are not controlled by unrelated applications.
2. Language Independence
    * Liquibase can run without relying on Java or Python application startup.
3. Deployment Consistency
    * Migrations execute before dependent application deployments.
4. Reduced Dependency Coupling
    * The DQP AI application does not need to manage AI Services database initialization.
5. Operational Visibility
    * Migration success or failure can be tracked as a distinct deployment step.

4.7 Additional Considerations

The team should investigate:

* Schema version management.
* Concurrent migration execution.
* Migration permissions.
* Environment-specific configuration.
* Database rollback or forward-fix procedures.
* System prompt versioning.
* Sensitive prompt content.
* Deployment failure handling.

Important architectural distinction: System prompts may require a different lifecycle from ordinary schema migrations.

The team should decide whether prompt content should be managed through Liquibase seed data or through a dedicated prompt-management process.

⸻

5. Integrated AI Demonstration

5.1 Discussion

An engineer reported progress on the initial English-to-SQL implementation.

The implementation is functional at a basic level but requires additional testing and optimization.

An internal demonstration was discussed.

However, the lead requested that the team postpone the broader demonstration until multiple AI capabilities could be shown together.

5.2 Required Capabilities

The proposed integrated demonstration includes:

Capability	Status	Notes
English-to-SQL	Initial Implementation	Testing and optimization required
English-to-DQDL	Not Clearly Established	Included in intended demonstration
RAG	Not Ready	Initial working implementation needed

5.3 Proposed Integrated Flow

flowchart TD
    A["User Provides Request"]
    B["AI Application"]
    C["English-to-SQL"]
    D["English-to-DQDL"]
    E["RAG Knowledge Retrieval"]
    F["Generated Rule or SQL"]
    G["Validation"]
    H["User Reviews Result"]
    A --> B
    B --> C
    B --> D
    C -.-> E
    D -.-> E
    E -.-> F
    C --> F
    D --> F
    F --> G
    G --> H

Note: This is a conceptual demonstration diagram. The actual service orchestration and use of RAG in individual generation paths have not been confirmed.

5.4 Analysis

The lead wants the team to present an integrated product rather than disconnected technical demonstrations.

This creates a more coherent story for stakeholders.

However, internal engineering demonstrations should still be encouraged.

Early demonstrations can help identify integration issues before the formal presentation.

5.5 Scope Clarification Needed

Meeting 6 identified these use cases:

* English-to-SQL.
* SQL-to-Spark SQL.
* Profile-to-DQDL.

Meeting 9 instead emphasizes:

* English-to-SQL.
* English-to-DQDL.
* RAG.

This difference should be clarified.

It is not yet clear whether English-to-DQDL refers to natural-language rule generation, profiling-based recommendations, or both.

⸻

6. Sprint Planning and Jira Stories

Story 1: HLDD Documentation

Title: Create and Obtain Approval for AI Application HLDD Documentation

Description:

Prepare architecture documentation for the two AI applications using Harmony metadata and the approved documentation-generation workflow.

Acceptance Criteria:

* [ ]	Verify both applications are registered in Harmony.
* [ ]	Review existing architecture information.
* [ ]	Generate HLDD documentation.
* [ ]	Create Confluence documentation.
* [ ]	Include architecture and sequence diagrams.
* [ ]	Validate technical accuracy.
* [ ]	Complete internal review.
* [ ]	Submit to Shika.
* [ ]	Address feedback and obtain approval.

Owner: Irish/Avish — name to be confirmed.

Dependencies: James, Harmony, architecture approval process.

⸻

Story 2: Liquibase Deployment Strategy

Title: Research and Design Liquibase Deployment Strategy for Python AI Services

Description:

Investigate and propose an appropriate approach for executing Liquibase database migrations for Python-based AI Services through the deployment pipeline.

Acceptance Criteria:

* [ ]	Identify existing AI Services schema requirements.
* [ ]	Review current Java Liquibase implementation.
* [ ]	Evaluate standalone Liquibase execution.
* [ ]	Evaluate CI/CD integration.
* [ ]	Identify deployment dependencies.
* [ ]	Propose a solution that avoids circular dependencies.
* [ ]	Define migration failure handling.
* [ ]	Document the recommended architecture.
* [ ]	Review the proposal with the team.

Owner: Irish/Avish.

Scope: Research and architecture design; implementation is not yet authorized by the meeting decision.

⸻

7. Implications for Frontend Development

7.1 No New UI Assignment

Meeting 9 did not introduce a new UI implementation requirement.

An engineer offered Mike assistance with frontend development if necessary.

However, the discussion has implications for future frontend integration.

7.2 Unified AI Recommendation Workflow

The UI should provide a consistent experience across generation capabilities.

Recommended interaction:

flowchart TD
    A["User Opens Rule Authoring"]
    B{"Generation Mode"}
    C["Manual Authoring"]
    D["Generate with AI"]
    E["Enter Natural Language"]
    F["Request Recommendations"]
    G["Display AI Suggestions"]
    H["Validate and Review"]
    I{"User Decision"}
    J["Accept Suggestion"]
    K["Discard Suggestion"]
    L["Populate Existing Rule Editor"]
    M["Save Rule"]
    A --> B
    B --> C
    B --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I -->|Accept| J
    I -->|Reject| K
    J --> L
    L --> M
    C --> M

This design is consistent with the previously discussed approach of integrating AI-assisted authoring into the existing modal.

7.3 Suggested Frontend API Contract

A common response model would help avoid creating unrelated frontend implementations for different AI capabilities.

type RuleLanguage = "SQL" | "DQDL";
type SuggestionStatus =
  | "PENDING"
  | "ACCEPTED"
  | "REJECTED";
type SuggestionSource =
  | "NATURAL_LANGUAGE"
  | "PROFILE";
interface RuleSuggestion {
  id: string;
  language: RuleLanguage;
  expression: string;
  description?: string;
  status: SuggestionStatus;
  source: SuggestionSource;
  validationErrors?: string[];
}

Note: This is a proposed TypeScript interface, not a confirmed backend API model.

7.4 Frontend Integration Concerns

The frontend should account for:

* AI generation loading states.
* Validation failures.
* API timeouts.
* Retry behavior.
* Multiple rule recommendations.
* Accept and discard interactions.
* Rule provenance.
* Feature flags.
* Persistence of accepted suggestions.
* Consistent error messaging.

7.5 Recommended Frontend Priority

Before implementing additional screens, confirm:

1. Which backend application owns recommendation orchestration?
2. Whether generation capabilities use unified or separate APIs.
3. How recommendation statuses are represented.
4. Whether generated suggestions are persisted immediately.
5. How validation results are returned.

⸻

8. Architecture Risks

Risk	Impact	Mitigation
Delayed HLDD Approval	Production readiness delays	Engage architecture reviewers early
Incomplete RAG	Integrated demo delays	Define a minimum viable demonstration
Liquibase Coupling	Deployment failures	Independent migration execution
Incorrect AI Documentation	Misleading system design	Verify against source code
Unclear Service Boundaries	API integration problems	Document application ownership
Inconsistent API Contracts	Frontend rework	Agree on shared contracts
Unclear Demo Scope	Misaligned team expectations	Confirm required use cases

⸻

9. Open Questions

Architecture

1. What are the official names of the two AI applications?
2. Which application owns recommendation orchestration?
3. Which service owns persistence of AI-generated suggestions?
4. What architecture documentation is mandatory for approval?
5. What information must Harmony contain?

Liquibase

1. Which deployment pipeline should execute migrations?
2. Should migrations live in a separate repository?
3. How will schema and system prompt updates be versioned?
4. How will migration failures be handled?
5. What is the required application deployment order?

AI Capabilities

1. What is the minimum RAG functionality required for the demo?
2. Does English-to-DQDL include profiling-driven recommendations?
3. Is SQL-to-Spark SQL still part of the initial delivery?
4. How will AI outputs be validated?
5. What dependencies exist between generation capabilities?

Frontend

1. Will the UI consume a unified recommendation API?
2. How will generated rules be associated with profile results?
3. Will unaccepted suggestions be persisted?
4. How will rejected recommendations be tracked?
5. Which capabilities will be controlled by separate feature flags?

⸻

10. Evolution Across Meetings

Meeting	Focus	Outcome
1–2	AI Rule Generation UX	AI-assisted rule authoring and acceptance workflow
3	Data Lake and SQL Rules	Environment and SQL-generation requirements
4–5	System Architecture	Expanded architectural exploration
6	Use Cases and Delivery	Initial scope, profiling prerequisites, team assignments
7	Database Design	Rule suggestion storage and feedback tracking
8	Profiling Recommendations	How to generate and display column-level suggestions
9	Architecture Governance	HLDD, Liquibase deployment, integrated demonstration

Project Evolution

flowchart TD
    A["AI Rule Generation Concept"]
    B["Define User Experience"]
    C["Define AI Use Cases"]
    D["Design Recommendation Persistence"]
    E["Integrate Profiling Metadata"]
    F["Build AI Services"]
    G["Define Deployment Architecture"]
    H["Complete Architecture Governance"]
    I["Integrated Testing and Demonstration"]
    J["Production Readiness"]
    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I
    I --> J

This diagram represents the project’s conceptual progression, not a confirmed dependency schedule.

⸻

11. Recommendations for Mike

Recommendation 1: Participate in Integrated Demo Planning

The team’s integrated demonstration should show a complete user workflow rather than isolated API responses.

The UI is central to communicating the value of the AI capabilities.

Participating in demo planning creates an opportunity to influence the overall product experience.

Recommendation 2: Request a Unified API Contract

As multiple AI capabilities are developed, frontend integration complexity can increase rapidly.

Request an architecture discussion covering:

* API endpoints.
* Request payloads.
* Response models.
* Validation.
* Error handling.
* Persistence.
* Recommendation lifecycle.

This can prevent avoidable frontend rework.

Recommendation 3: Ensure Frontend Architecture Is Documented

The HLDD should include:

* React application boundaries.
* Frontend-to-backend communication.
* Authentication behavior.
* Feature flag integration.
* Rule authoring workflow.
* Recommendation acceptance.
* Error handling.

The frontend is an architectural component, not simply a presentation layer.

Recommendation 4: Maintain Visibility Into Backend Decisions

You do not need to own Liquibase or Harmony.

However, understanding how those systems affect deployment and integration will help you make stronger engineering decisions.

Recommendation 5: Connect Previous Design Discussions to Current Integration Work

Meetings 7 and 8 introduced important questions regarding:

* Recommendation persistence.
* Accepted and rejected suggestions.
* Profiling-derived recommendations.
* Rule authoring integration.

Meeting 9 introduces the integration environment in which those capabilities must operate.

Connect these discussions by helping the team define a complete recommendation lifecycle.

⸻

12. Final Assessment

Meeting 9 marks a transition from feature development toward enterprise architecture and integrated delivery.

Confirmed Outcomes

* HLDD documentation is required.
* Harmony registration must be verified.
* Architecture approval is required.
* Liquibase deployment requires further research.
* The team wants an integrated AI demonstration.
* Two new backlog stories were requested.

Outstanding Decisions

* Final Liquibase execution strategy.
* Exact application dependency boundaries.
* RAG readiness criteria.
* Unified recommendation API contracts.
* Final demonstration scope.

Key Takeaway

The project is evolving beyond generating valid SQL and DQDL expressions.

The engineering challenge now involves delivering a reliable, integrated AI recommendation platform with clear ownership, versioned database changes, approved architecture, and consistent user workflows.

For the UI engineer, the strongest opportunity is to influence the end-to-end product architecture by connecting AI generation, recommendation persistence, validation, and rule acceptance into one cohesive experience.