# Structuring a Multi-Agent Project Directory

For a project that uses separate planning, discovery, implementation, testing, and review agents, keep agent instructions separate from application code. Give every agent a predictable place to read inputs and write artifacts.

## Recommended Project Layout

```text
my-project/
├── .github/
│   ├── copilot-instructions.md
│   └── agents/
│       ├── planning.agent.md
│       ├── discovery.agent.md
│       ├── implementation.agent.md
│       ├── testing.agent.md
│       └── review.agent.md
│
├── docs/
│   ├── architecture/
│   │   ├── system-overview.md
│   │   └── decisions/
│   │       └── 0001-example-decision.md
│   ├── requirements/
│   │   └── product-requirements.md
│   ├── external/
│   │   └── exported-page.md
│   └── workflows/
│       └── discovery-remediation.md
│
├── agent-work/
│   ├── inputs/
│   │   ├── repository-advice.json
│   │   └── task-context.md
│   ├── discovery/
│   │   ├── findings.json
│   │   └── findings.md
│   ├── plans/
│   │   └── implementation-plan.md
│   ├── remediation/
│   │   └── remediation-report.md
│   └── reviews/
│       └── review-report.md
│
├── scripts/
│   ├── export-page.ts
│   ├── discover-past-due.ts
│   └── validate-agent-output.ts
│
├── src/
│   ├── app/
│   ├── components/
│   ├── features/
│   ├── hooks/
│   ├── services/
│   ├── types/
│   └── utils/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
├── package.json
├── tsconfig.json
├── README.md
└── TASKS.md
```

## Separation of Responsibilities

Treat the workflow artifacts as separate concerns:

```text
agent-work/inputs
        ↓
agent-work/discovery
        ↓
agent-work/plans
        ↓
source-code changes
        ↓
agent-work/remediation
        ↓
agent-work/reviews
```

### `agent-work/inputs/`

Store raw data that agents must not modify.

Example: `agent-work/inputs/repository-advice.json`

```ts
type RepositoryAdvice = {
  repoName: string;
  repoUrl: string;
  adviceSummary: {
    pastDue: number;
  };
};
```

This file contains the repository array and related advice fields.

### `agent-work/discovery/`

The Discovery Agent analyzes the inputs but does not change repositories.

Example: `findings.json`

```json
{
  "repositories": [
    {
      "repoName": "example-repository",
      "repoUrl": "https://github.example.com/example-repository",
      "pastDue": 2,
      "status": "remediation-required"
    }
  ]
}
```

Only include repositories where:

```text
adviceSummary.pastDue > 0
```

### `agent-work/plans/`

The Planning Agent reads the discovery findings and writes an approved remediation plan.

Example: `agent-work/plans/implementation-plan.md`

```markdown
# Remediation Plan

## Repository

- Name: example-repository
- URL: https://github.example.com/example-repository
- Past-due actions: 2

## Proposed Work

1. Inspect the two past-due recommendations.
2. Map each recommendation to affected files.
3. Implement the smallest safe changes.
4. Run validation.
5. Record unresolved items.
```

### `agent-work/remediation/`

The Implementation Agent records what it actually changed.

Example: `remediation-report.md`

```markdown
# Remediation Report

## Completed

- Resolved action 1
- Resolved action 2

## Validation

- Type check: Passed
- Tests: Passed
- Lint: Passed

## Files Changed

- `src/example.ts`
- `tests/example.test.ts`
```

## Agent Instruction Files

Keep each agent narrowly scoped.

### `discovery.agent.md`

```markdown
# Discovery Agent

## Purpose

Analyze repository advice data and identify repositories requiring remediation.

## Responsibilities

1. Read `agent-work/inputs/repository-advice.json`.
2. Evaluate every object in the `repos` array.
3. Read `adviceSummary.pastDue`.
4. Ignore repositories where `pastDue` is zero or missing.
5. For repositories where `pastDue` is greater than zero, return:
   - `repoName`
   - `repoUrl`
   - `pastDue`
6. Write results to `agent-work/discovery/findings.json`.

## Restrictions

- Do not modify application code.
- Do not clone or remediate repositories.
- Do not invent missing values.
- Do not interpret `pastDue` as a boolean.
- Treat `pastDue` as the number of actions requiring remediation.
```

### `implementation.agent.md`

```markdown
# Implementation Agent

## Purpose

Remediate repositories identified during discovery.

## Preconditions

Do not begin unless these files exist:

- `agent-work/discovery/findings.json`
- `agent-work/plans/implementation-plan.md`

## Responsibilities

1. Read the approved plan.
2. Process only repositories listed in the discovery findings.
3. Resolve the documented past-due actions.
4. Make the smallest coherent code changes.
5. Run relevant tests, linting, and type checks.
6. Write a remediation report.

## Restrictions

- Do not perform discovery again.
- Do not add unrequested features.
- Do not perform unrelated refactoring.
- Do not claim an action is resolved without validation.
```

## Application Code Structure

For a React and TypeScript project, organize primarily by feature rather than placing every component in one global folder:

```text
src/
├── app/
│   ├── App.tsx
│   ├── router.tsx
│   └── providers.tsx
│
├── features/
│   └── repository-advice/
│       ├── components/
│       │   ├── RepositoryAdviceTable.tsx
│       │   └── PastDueBadge.tsx
│       ├── hooks/
│       │   └── useRepositoryAdvice.ts
│       ├── services/
│       │   └── repositoryAdviceService.ts
│       ├── types/
│       │   └── repositoryAdvice.ts
│       ├── utils/
│       │   └── filterPastDueRepositories.ts
│       └── index.ts
│
├── components/
│   └── shared/
│       ├── Button/
│       └── LoadingState/
│
├── services/
│   └── httpClient.ts
│
├── types/
│   └── common.ts
│
└── utils/
    └── formatError.ts
```

Feature-specific code should stay inside the feature. Only genuinely reusable code belongs in global components, services, types, or utilities.

## Recommended Minimal Version

You do not need every folder immediately. Start with:

```text
my-project/
├── .github/
│   ├── copilot-instructions.md
│   └── agents/
│       ├── planning.agent.md
│       ├── discovery.agent.md
│       └── implementation.agent.md
├── agent-work/
│   ├── inputs/
│   ├── discovery/
│   ├── plans/
│   └── remediation/
├── docs/
├── src/
├── tests/
├── package.json
├── tsconfig.json
├── README.md
└── TASKS.md
```

Expand the structure as the project grows. The important principle is to keep agent instructions, workflow artifacts, documentation, and application code clearly separated.
