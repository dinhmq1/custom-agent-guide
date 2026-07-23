# Discovery-to-Remediation Agent Pipeline

When a payload needs to pass through discovery, use a staged pipeline. Each agent should have a narrow responsibility and produce a concrete artifact for the next stage.

## End-to-End Pipeline

```text
Raw payload
    ↓
Input validation
    ↓
Discovery Agent
    ↓
Discovery findings
    ↓
Planning Agent
    ↓
Approved remediation plan
    ↓
Implementation Agent
    ↓
Validation and tests
    ↓
Review Agent
    ↓
Final remediation report
```

## 1. Store the Raw Payload

Place the original payload somewhere agents treat as read-only:

```text
agent-work/inputs/repository-advice.json
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
    },
    {
      "repoName": "customer-ui",
      "repoUrl": "https://github.example.com/team/customer-ui",
      "adviceSummary": {
        "pastDue": 0
      }
    }
  ]
}
```

Do not let agents overwrite this file. It is the source-of-truth input for the run.

## 2. Validate the Payload

Before discovery begins, validate that the payload has the expected structure. A script such as `scripts/validate-input.ts` should check that:

- `repos` exists and is an array.
- `repoName` is a non-empty string.
- `repoUrl` is a valid string.
- `adviceSummary` exists.
- `pastDue` is a non-negative number.

Example TypeScript types:

```ts
type RepositoryAdvice = {
  repoName: string;
  repoUrl: string;
  adviceSummary: {
    pastDue: number;
  };
};

type RepositoryAdvicePayload = {
  repos: RepositoryAdvice[];
};
```

The result should be either:

```text
Input valid
```

or a structured error report:

```json
{
  "status": "invalid",
  "errors": [
    {
      "path": "repos[2].adviceSummary.pastDue",
      "message": "Expected a non-negative number"
    }
  ]
}
```

Do not continue to discovery if validation fails.

## 3. Discovery Agent Analyzes the Payload

The Discovery Agent reads:

```text
agent-work/inputs/repository-advice.json
```

Its only responsibility is to identify repositories requiring action. The filter is:

```ts
repo.adviceSummary.pastDue > 0
```

For every matching repository, return:

- `repoName`
- `repoUrl`
- `pastDue`
- Optionally, `localPath`
- Discovery status

Write the result to:

```text
agent-work/discovery/findings.json
```

Example:

```json
{
  "runId": "2026-07-23-001",
  "status": "completed",
  "repositoryCount": 2,
  "remediationRequiredCount": 1,
  "repositories": [
    {
      "repoName": "payments-api",
      "repoUrl": "https://github.example.com/team/payments-api",
      "pastDue": 2,
      "localPath": "repos/payments-api",
      "status": "remediation-required"
    }
  ]
}
```

### Discovery Agent Restrictions

The Discovery Agent should not:

- Clone repositories.
- Modify code.
- Decide how to remediate an issue.
- Infer the exact past-due actions if they are not in the payload.
- Change the original input.

At this point, `pastDue: 2` means only that the repository has two past-due actions requiring further inspection. It does not identify those actions unless the payload includes their details.

## 4. Enrich the Findings or Inspect the Repository

This stage depends on the payload contents. If the payload only provides a count, such as:

```json
{
  "pastDue": 2
}
```

then another step must determine what those two actions actually are.

You can use a dedicated **Repository Inspection Agent** or make this part of the Planning Agent’s work.

Its process is:

```text
Read discovery findings
        ↓
Clone affected repository
        ↓
Inspect repository advice details
        ↓
Identify specific past-due actions
        ↓
Record evidence
```

Write the result to a repository-specific artifact, such as:

```text
agent-work/discovery/payments-api-details.json
```

Example:

```json
{
  "repoName": "payments-api",
  "repoUrl": "https://github.example.com/team/payments-api",
  "pastDue": 2,
  "actions": [
    {
      "id": "action-123",
      "title": "Upgrade vulnerable dependency",
      "status": "past-due",
      "evidence": {
        "package": "example-package",
        "currentVersion": "1.2.0",
        "requiredVersion": "1.4.2"
      }
    },
    {
      "id": "action-456",
      "title": "Enable branch protection",
      "status": "past-due"
    }
  ]
}
```

This creates an important distinction:

- **Initial discovery:** Which repositories need work?
- **Detailed discovery:** What work does each repository need?

## 5. Planning Agent Creates the Remediation Plan

The Planning Agent reads:

```text
agent-work/discovery/findings.json
agent-work/discovery/*-details.json
```

It produces either:

```text
agent-work/plans/remediation-plan.json
```

or:

```text
agent-work/plans/remediation-plan.md
```

Use JSON for machine execution and Markdown for human review. In practice, keeping both can be useful.

### Machine-Readable Plan

```json
{
  "runId": "2026-07-23-001",
  "approvalStatus": "pending",
  "repositories": [
    {
      "repoName": "payments-api",
      "repoUrl": "https://github.example.com/team/payments-api",
      "localPath": "repos/payments-api",
      "branchName": "remediation/past-due-actions",
      "actions": [
        {
          "actionId": "action-123",
          "summary": "Upgrade example-package",
          "proposedChanges": [
            "Update package.json",
            "Update lockfile",
            "Run dependency tests"
          ],
          "validation": [
            "npm run typecheck",
            "npm test",
            "npm run lint"
          ]
        }
      ]
    }
  ]
}
```

The Planning Agent should identify:

- Affected files
- Likely changes
- Dependencies between actions
- Validation commands
- Risks
- Work requiring manual intervention
- Anything the Implementation Agent must not change

The Planning Agent should not implement the changes.

## 6. Add an Approval Gate

Before implementation begins, add a simple approval marker:

```text
agent-work/plans/APPROVED
```

Alternatively, include an approval status in the plan:

```json
{
  "approvalStatus": "approved"
}
```

The Implementation Agent should refuse to proceed unless the plan is approved. This protects against an agent changing multiple repositories based on incomplete discovery.

## 7. Implementation Agent Clones and Remediates

The Implementation Agent reads the approved plan. For each repository, it should:

```text
Check whether repos/<repoName> exists
        ↓
Clone or update the repository
        ↓
Create a remediation branch
        ↓
Implement one planned action
        ↓
Run validation
        ↓
Record the result
        ↓
Continue to the next action
```

Example local structure:

```text
repos/
└── payments-api/
    ├── .git/
    ├── src/
    ├── tests/
    └── package.json
```

Use either one branch per action:

```text
remediation/action-123
```

or one branch for the entire repository run:

```text
remediation/past-due-actions-2026-07-23
```

Write the repository report to:

```text
agent-work/remediation/payments-api-report.json
```

Example:

```json
{
  "repoName": "payments-api",
  "branchName": "remediation/past-due-actions-2026-07-23",
  "status": "partially-completed",
  "actions": [
    {
      "actionId": "action-123",
      "status": "resolved",
      "filesChanged": [
        "package.json",
        "package-lock.json"
      ],
      "validation": {
        "typecheck": "passed",
        "tests": "passed",
        "lint": "passed"
      }
    },
    {
      "actionId": "action-456",
      "status": "manual-action-required",
      "reason": "Branch protection must be configured through repository settings"
    }
  ]
}
```

## 8. Validation Agent Verifies the Changes

A separate Validation or Testing Agent should independently run the required commands.

It reads:

- The approved plan
- The modified repository
- The remediation report

It verifies:

- Intended files changed.
- Unrelated files did not change.
- Tests pass.
- Type checking passes.
- Lint passes.
- The original past-due condition is actually resolved.
- Manual actions are clearly marked.

Write the result to:

```text
agent-work/remediation/payments-api-validation.json
```

Example:

```json
{
  "repoName": "payments-api",
  "status": "passed",
  "checks": [
    {
      "name": "typecheck",
      "status": "passed"
    },
    {
      "name": "tests",
      "status": "passed"
    },
    {
      "name": "scope-check",
      "status": "passed"
    }
  ]
}
```

## 9. Review Agent Inspects the Full Change

The Review Agent evaluates the actual Git diff and reports findings by severity.

Example:

```json
{
  "repoName": "payments-api",
  "decision": "changes-requested",
  "findings": [
    {
      "severity": "major",
      "file": "src/example.ts",
      "line": 42,
      "message": "The new fallback hides the dependency initialization failure."
    }
  ]
}
```

If there are blockers or major findings, send the work through this loop:

```text
Review Agent
      ↓
Implementation Agent fixes findings
      ↓
Validation Agent reruns checks
      ↓
Review Agent rechecks
```

## 10. Create the Final Report

After every repository is processed, create:

```text
agent-work/remediation/final-report.json
```

Example:

```json
{
  "runId": "2026-07-23-001",
  "status": "partially-completed",
  "summary": {
    "repositoriesEvaluated": 2,
    "repositoriesRequiringRemediation": 1,
    "actionsFound": 2,
    "actionsResolved": 1,
    "manualActionsRequired": 1,
    "actionsFailed": 0
  },
  "repositories": [
    {
      "repoName": "payments-api",
      "repoUrl": "https://github.example.com/team/payments-api",
      "branchName": "remediation/past-due-actions-2026-07-23",
      "resolved": 1,
      "manual": 1,
      "failed": 0
    }
  ]
}
```

## Recommended Practical Pipeline

For the current stage of the project, these agents and scripts are sufficient:

1. Input Validator
2. Discovery Agent
3. Planning Agent
4. Human approval gate
5. Implementation Agent
6. Validation script or Testing Agent
7. Review Agent

You do not necessarily need a separate agent for every stage. Scripts are often better for deterministic work.

### Use Code for Predictable Tasks

- Parsing JSON
- Checking types
- Filtering `pastDue > 0`
- Cloning repositories
- Running commands
- Generating counts

### Use Agents for Judgment-Heavy Tasks

- Understanding remediation advice
- Locating affected code
- Deciding on an implementation strategy
- Reviewing correctness

## Artifact Flow

```text
agent-work/inputs/repository-advice.json
              ↓
agent-work/discovery/findings.json
              ↓
agent-work/discovery/<repo>-details.json
              ↓
agent-work/plans/remediation-plan.json
              ↓
repos/<repoName>/
              ↓
agent-work/remediation/<repo>-report.json
              ↓
agent-work/remediation/<repo>-validation.json
              ↓
agent-work/reviews/<repo>-review.json
              ↓
agent-work/remediation/final-report.json
```

## Core Design Rule

Do not let the Implementation Agent consume the raw payload and immediately start changing repositories.

**Discovery** should turn raw data into trusted findings. **Planning** should turn those findings into bounded, approved instructions. **Implementation** should execute only the approved plan, with validation and review before the run is considered complete.
