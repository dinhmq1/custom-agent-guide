# Using `repos/` in an Agent Orchestration Workspace

## Question

For now, I updated my project directory using the recommended minimal version. However, I changed `src/` to `repos/` because I plan to clone repositories that need remediation into that directory. Is that a good approach?

## Recommendation

Yes. Renaming `src/` to `repos/` is appropriate.

`src/` usually implies the source code for the current application. In this case, the project is an **orchestration workspace** that coordinates work across other repositories. The name `repos/` communicates that purpose more clearly.

## Recommended Directory Structure

```text
my-project/
├── .github/
│   ├── copilot-instructions.md
│   └── agents/
│       ├── planning.agent.md
│       ├── discovery.agent.md
│       └── implementation.agent.md
│
├── agent-work/
│   ├── inputs/
│   ├── discovery/
│   ├── plans/
│   └── remediation/
│
├── docs/
├── scripts/
├── repos/
├── tests/
├── .gitignore
└── README.md
```

Each cloned repository should have its own directory:

```text
repos/
├── payments-api/
├── customer-portal/
└── notifications-service/
```

## Keep Cloned Repositories Out of Git

In most cases, do not commit cloned repositories into the orchestration project. Add this to `.gitignore`:

```gitignore
repos/*
!repos/.gitkeep
```

Then create an empty placeholder file:

```text
repos/.gitkeep
```

This keeps the directory in Git without accidentally committing cloned repositories, nested `.git` directories, build output, secrets, or dependency directories.

## Use a Predictable Naming Convention

Define one local path convention:

```text
repos/<repoName>
```

The Implementation Agent can then derive the local path directly from discovery output:

```json
{
  "repoName": "payments-api",
  "repoUrl": "https://github.example.com/org/payments-api",
  "pastDue": 2,
  "localPath": "repos/payments-api"
}
```

The `localPath` field is optional, but including it reduces ambiguity between agents.

## Agent Responsibilities

The Discovery Agent should usually **not** clone anything. A cleaner workflow is:

```text
Discovery Agent
      ↓
Identifies affected repositories
      ↓
Planning Agent
      ↓
Creates the remediation plan
      ↓
Implementation Agent
      ↓
Clones into repos/
      ↓
Performs remediation
      ↓
Records results
```

Keeping discovery separate from implementation makes the workflow easier to audit, test, and repeat.

## Optional Checkout Script

As the workflow grows, add a dedicated checkout script:

```text
scripts/
├── clone-repositories.ts
├── discover-past-due.ts
└── validate-agent-output.ts
```

The checkout script can consistently handle:

- Existing repository directories
- Branch selection
- Authentication failures
- Duplicate repository names
- Invalid or disallowed repository URLs

## Final Guidance

Use `repos/` for cloned repositories and treat it as disposable working space. Keep the orchestration repository focused on agent instructions, workflow artifacts, documentation, scripts, and reports—not the contents of the cloned repositories.
