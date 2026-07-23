# Two-Phase Discovery for Remediation Workflows

The repository summary payload only indicates where remediation is needed. Because it contains only a `pastDue` count, the workflow also needs a second payload containing the detailed remediation advice for those actions.

The summary payload identifies the scope of work. The advice payload defines the work itself.

## Two-Phase Discovery Model

```text
Repository summary discovery
            ↓
Identify repositories with pastDue > 0
            ↓
Fetch remediation-advice payloads
            ↓
Extract and correlate past-due advice
            ↓
Produce implementation-ready findings
```

## Revised Pipeline

```text
Summary payload
      ↓
Summary Discovery Agent
      ↓
Affected repositories
      ↓
Advice Retrieval Step
      ↓
Remediation advice payloads
      ↓
Advice Discovery Agent
      ↓
Implementation-ready findings
      ↓
Planning Agent
      ↓
Implementation Agent
```

## 1. Store Both Payload Types Separately

```text
agent-work/
├── inputs/
│   ├── repository-summary.json
│   └── remediation-advice/
│       ├── payments-api.json
│       └── customer-portal.json
│
├── discovery/
│   ├── affected-repositories.json
│   └── remediation-findings.json
│
├── plans/
└── remediation/
```

Do not combine the raw payloads immediately. Keeping them separate makes troubleshooting, auditing, and rerunning individual stages easier.

## 2. Phase One: Find Affected Repositories

### Input

```text
agent-work/inputs/repository-summary.json
```

Example:

```json
{
  "repos": [
    {
      "repoName": "payments-api",
      "repoUrl": "https://github.example.com/team/payments-api",
      "adviceSummary": {
        "pastDue": 2
      }
    }
  ]
}
```

### Output

```text
agent-work/discovery/affected-repositories.json
```

Example:

```json
{
  "repositories": [
    {
      "repoName": "payments-api",
      "repoUrl": "https://github.example.com/team/payments-api",
      "expectedPastDueCount": 2,
      "status": "advice-payload-required"
    }
  ]
}
```

The Summary Discovery Agent should stop here. It should not plan or implement anything.

## 3. Retrieve the Detailed Advice Payload

For each affected repository, retrieve the payload containing the `advice` array.

Conceptually:

```text
payments-api
      ↓
GET remediation advice
      ↓
agent-work/inputs/remediation-advice/payments-api.json
```

This retrieval is better handled by a deterministic script or API client than by an agent:

```text
scripts/fetch-remediation-advice.ts
```

The script can read `affected-repositories.json` and fetch advice for every listed repository.

## 4. Phase Two: Extract Actionable Advice

Suppose the detailed payload looks like this:

```json
{
  "repoName": "payments-api",
  "repoUrl": "https://github.example.com/team/payments-api",
  "advice": [
    {
      "adviceId": "ADV-123",
      "title": "Upgrade vulnerable dependency",
      "status": "PAST_DUE",
      "description": "Upgrade example-package to version 4.2.0 or later.",
      "dueDate": "2026-06-30",
      "category": "dependency",
      "severity": "high"
    },
    {
      "adviceId": "ADV-456",
      "title": "Enable branch protection",
      "status": "PAST_DUE",
      "description": "Require pull-request reviews on the default branch.",
      "dueDate": "2026-07-01",
      "category": "repository-setting",
      "severity": "medium"
    }
  ]
}
```

The Advice Discovery Agent should:

1. Read the `advice` array.
2. Select only entries representing past-due actions.
3. Preserve all properties needed for implementation.
4. Correlate the advice with `repoName` and `repoUrl`.
5. Verify the extracted count against `adviceSummary.pastDue`.
6. Produce implementation-ready findings.

## 5. Produce a Normalized Discovery Artifact

Write the normalized result to:

```text
agent-work/discovery/remediation-findings.json
```

Example:

```json
{
  "runId": "2026-07-23-001",
  "repositories": [
    {
      "repoName": "payments-api",
      "repoUrl": "https://github.example.com/team/payments-api",
      "localPath": "repos/payments-api",
      "expectedPastDueCount": 2,
      "actualPastDueCount": 2,
      "countMatches": true,
      "actions": [
        {
          "adviceId": "ADV-123",
          "title": "Upgrade vulnerable dependency",
          "description": "Upgrade example-package to version 4.2.0 or later.",
          "status": "PAST_DUE",
          "dueDate": "2026-06-30",
          "category": "dependency",
          "severity": "high"
        },
        {
          "adviceId": "ADV-456",
          "title": "Enable branch protection",
          "description": "Require pull-request reviews on the default branch.",
          "status": "PAST_DUE",
          "dueDate": "2026-07-01",
          "category": "repository-setting",
          "severity": "medium"
        }
      ]
    }
  ]
}
```

This artifact becomes the contract between discovery and planning.

## Preserve the Original Advice Data

Do not over-summarize the advice. If implementation depends on fields such as these, preserve them:

- `adviceId`
- `title`
- `description`
- `recommendation`
- `status`
- `dueDate`
- `severity`
- `category`
- `resource`
- `currentValue`
- `recommendedValue`
- `documentationUrl`
- `remediationSteps`

The exact fields depend on the actual payload.

A safe pattern is to retain both normalized fields and the complete source object:

```json
{
  "normalized": {
    "adviceId": "ADV-123",
    "title": "Upgrade vulnerable dependency",
    "status": "PAST_DUE"
  },
  "sourceAdvice": {
    "adviceId": "ADV-123",
    "title": "Upgrade vulnerable dependency",
    "status": "PAST_DUE",
    "description": "Upgrade example-package to version 4.2.0 or later."
  }
}
```

This gives downstream agents clean fields while preventing useful source information from being discarded.

## Reconcile the Counts

The second discovery phase should compare:

```ts
expectedPastDueCount === actualPastDueAdvice.length
```

Example mismatch:

```json
{
  "expectedPastDueCount": 2,
  "actualPastDueCount": 1,
  "countMatches": false,
  "status": "discovery-incomplete"
}
```

When the counts do not match, planning should not proceed automatically.

Possible causes include:

- Pagination was not fully retrieved.
- The detailed payload is stale.
- Status filtering is incorrect.
- The summary and advice endpoints update at different times.
- An advice item is missing.
- Status values differ from what the agent expected.

## Agent Responsibilities

### Summary Discovery Agent

```markdown
# Summary Discovery Agent

## Purpose

Identify repositories that have past-due actions.

## Responsibilities

1. Read `agent-work/inputs/repository-summary.json`.
2. Evaluate every object in the `repos` array.
3. Read `repoName` and `repoUrl`.
4. Read `adviceSummary.pastDue`.
5. Include the repository only when `pastDue > 0`.
6. Store the value as `expectedPastDueCount`.
7. Write results to `agent-work/discovery/affected-repositories.json`.

## Restrictions

- Do not retrieve remediation details.
- Do not clone repositories.
- Do not create implementation plans.
```

### Advice Discovery Agent

```markdown
# Advice Discovery Agent

## Responsibilities

For every repository identified by summary discovery:

1. Read its remediation advice payload.
2. Inspect every object in the `advice` array.
3. Select only advice objects representing past-due actions.
4. Preserve all properties required to understand and remediate each action.
5. Include `repoName`, `repoUrl`, and `localPath`.
6. Compare the extracted action count with `expectedPastDueCount`.
7. Mark the repository incomplete when the counts do not match.
8. Write normalized findings to `agent-work/discovery/remediation-findings.json`.

## Restrictions

- Do not implement remediation.
- Do not invent missing advice details.
- Do not remove source fields that may be required downstream.
```

## Planning Consumes Only Enriched Findings

The Planning Agent should read:

```text
agent-work/discovery/remediation-findings.json
```

It should not need to interpret the summary payload or find the advice payload itself. Its input should already answer:

- Which repository is affected?
- How many actions are expected?
- Which advice objects describe the actions?
- What does each action require?
- Are the discovery counts consistent?
- Where will the repository be cloned?

## Recommended Full Flow

```text
repository-summary.json
        ↓
Summary Discovery Agent
        ↓
affected-repositories.json
        ↓
Advice Retrieval Script
        ↓
remediation-advice/<repo>.json
        ↓
Advice Discovery Agent
        ↓
remediation-findings.json
        ↓
Planning Agent
        ↓
remediation-plan.json
        ↓
Approval
        ↓
Implementation Agent
        ↓
Validation
        ↓
Review
```

## Core Distinction

> The summary payload identifies the remediation scope. The advice payload defines the remediation work.

Do not send the Implementation Agent forward until both payloads have been correlated into one validated discovery artifact.
