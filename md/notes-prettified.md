# Custom Agent Workflow Notes

## Recommended Agent After a Planning Agent

After a **Planning Agent**, the best next custom agent is an **Implementation Agent**.

The Planning Agent decides what should be built. The Implementation Agent turns that approved plan into working code without quietly expanding scope.

### 1. Implementation Agent

Give it responsibilities such as:

- Read the approved plan before changing anything.
- Implement one task or milestone at a time.
- Follow the existing architecture, conventions, and design system.
- Use React functional components with TypeScript and TSX.
- Avoid unrelated refactors.
- Add or update tests for meaningful behavior.
- Run type-checking, linting, and tests after changes.
- Report files changed, decisions made, and unresolved issues.
- Stop when requirements are ambiguous rather than inventing features.

> **Operating rule:** Implement the approved plan exactly. Make the smallest coherent change that fully satisfies the current task. Do not redesign adjacent code or add speculative functionality.

### 2. Code Review Agent

Create this immediately after the Implementation Agent. It should not write the original implementation, because reviewers are more useful when they are independent.

Have it inspect:

- Correctness and edge cases
- State-management mistakes
- React rendering and hook issues
- Type safety
- Security and authorization boundaries
- Accessibility
- Test quality
- Unnecessary complexity
- Meaningful performance problems
- Whether the code matches the approved plan

Require findings to be ranked:

- **Blocker** — Unsafe or functionally incorrect
- **Major** — Likely bug or maintainability problem
- **Minor** — Worthwhile improvement
- **Optional** — Preference, not a requirement

This prevents the review from becoming a pile of subjective style comments.

### 3. Test Agent

This agent should derive tests from requirements rather than merely increasing coverage.

For a HomeMenu-style application, it could focus on:

- Recommendation-engine behavior
- Pantry quantity and expiration logic
- Grocery-list aggregation
- Authentication and authorization
- Supabase row-level security assumptions
- Form validation
- Loading, empty, and error states
- Date and timezone boundaries
- User interactions rather than implementation details

> **Core rule:** Test the behavior and business rules users depend on. Do not create brittle tests solely to increase coverage percentages.

### 4. Debugging Agent

This differs from the Implementation Agent. Feed it a failure, logs, reproduction steps, and relevant code.

It should:

1. Reproduce or precisely characterize the failure.
2. Generate several hypotheses.
3. Rank them by likelihood.
4. Gather evidence.
5. Identify the root cause.
6. Apply the smallest safe fix.
7. Add a regression test.

Tell it not to rewrite large sections of code before proving the root cause.

### 5. Architecture Agent

Create this later, not immediately. Use it for decisions that cross boundaries, such as:

- Supabase versus a dedicated backend
- Database schema changes
- Recommendation-engine boundaries
- Background jobs
- Caching
- Offline support
- Authentication architecture
- Deployment and observability

It should produce architecture decision records rather than implementation code:

- Context
- Decision
- Alternatives considered
- Trade-offs
- Risks
- Migration path
- How the decision can be reversed

### Recommended Sequence

```text
Planning Agent
      ↓
Implementation Agent
      ↓
Test Agent
      ↓
Code Review Agent
      ↓
Debugging Agent, only when needed
      ↓
Documentation / Release Agent
```

Avoid creating dozens of highly specialized agents early. Start with these three:

1. Planning Agent
2. Implementation Agent
3. Code Review Agent

That provides genuine separation of concerns: one defines the work, one performs it, and one challenges it. The Implementation Agent should be the next one.

## Workaround When Copilot Agent Cannot Browse or Use Vision

One of the best workarounds is to convert a page into Markdown or plain text before handing it to the agent. Coding agents are generally better at reasoning over structured text than screenshots or raw HTML.

### 1. Use a Markdown Converter

For web pages, possible tools include:

```bash
npx @mizchi/readability https://example.com
npx url-to-markdown https://example.com
curl https://example.com | pandoc -f html -t markdown
```

The agent receives headings, paragraphs, code blocks, and lists instead of hundreds of lines of HTML.

### 2. Use Reader Mode

Most browsers can produce a clean reader view:

- Safari Reader
- Firefox Reader
- Edge Immersive Reader

Then copy all of the reader view and paste it into Copilot Agent. This removes ads, navigation, sidebars, CSS, and JavaScript while retaining the article content.

### 3. Use a CLI Fetcher

If you are comfortable in the terminal:

```bash
curl https://example.com
wget -qO- https://example.com
```

Pipe the result through a converter such as `pandoc` or `html2text`:

```bash
curl https://example.com | html2text
```

### 4. Use a Browser Extension

Extensions such as **MarkDownload**, **Copy as Markdown**, and **Markdown Here** can export or copy page content as Markdown.

### 5. Use Playwright

If you are already using Playwright for automation:

```ts
const html = await page.content();
const text = await page.locator("body").innerText();
```

You can create a small script that outputs a file such as `docs/page.md`.

### 6. Use Microsoft MarkItDown

Microsoft’s open-source **MarkItDown** tool converts HTML, PDF, Word documents, PowerPoint presentations, Excel files, and other formats into LLM-friendly Markdown.

### 7. Create a Preprocessing Agent

For a multi-agent workflow, add a lightweight **Document Normalizer Agent** before the Planning or Implementation Agent:

```text
URL
 ↓
Fetch page
 ↓
Extract readable content
 ↓
Convert to Markdown
 ↓
Remove navigation
 ↓
Keep code blocks
 ↓
Summarize structure
 ↓
Feed to Copilot Agent
```

For React, TypeScript, or API documentation, a useful pipeline is:

```text
Website → MarkItDown → Markdown (.md) → Chunk into sections
                                           ↓
                                    Planning Agent
                                           ↓
                                  Implementation Agent
```

This preserves headings, code examples, tables when possible, and links in a more digestible format than raw HTML or screenshots.

## Discovering Repositories With Past-Due Actions

Suppose the JSON payload contains a `repos` array of objects. The Implementation Agent should inspect each repository’s `adviceSummary.pastDue` property and report repositories where the value is greater than zero.

### Discovery Requirements

For each object in `repos`:

1. Read the `adviceSummary` object.
2. Check the `pastDue` property.
3. Treat `pastDue` as the number of actions required for that repository.
4. Record `repoName`, `repoUrl`, and `pastDue` when `pastDue > 0`.
5. Ignore missing, null, invalid, undefined, or zero values.

The discovery step must not modify the payload, resolve actions, or make code changes.

Expected output:

```json
[
  {
    "repoName": "...",
    "repoUrl": "...",
    "pastDue": 2
  }
]
```

> `pastDue = 0` means there are no overdue actions. `pastDue = 2` means there are two overdue actions that must be resolved.

## Separating Discovery and Remediation

Separate the workflow into two agents—or two explicit phases—with a saved handoff artifact between them.

### Discovery Phase

The Discovery Agent only inspects the payload and produces a structured list of repositories requiring work. It must not clone repositories, edit files, open pull requests, or attempt remediation.

#### Discovery Agent Instructions

Evaluate the provided JSON payload. For every repository in `repos`:

- Read `adviceSummary.pastDue`.
- Include the repository only when `pastDue > 0`.
- Return `repoName`, `repoUrl`, and `pastDue`.
- Treat missing, null, invalid, or undefined `pastDue` values as `0`.
- Do not modify the payload or access any repository.

Return valid JSON in this format:

```json
{
  "repositoriesRequiringRemediation": [
    {
      "repoName": "example-repository",
      "repoUrl": "https://example.com/example-repository",
      "pastDue": 2
    }
  ],
  "repositoryCount": 1,
  "totalPastDueActions": 2
}
```

Save this discovery manifest somewhere predictable, such as:

```text
.agent-output/past-due-repositories.json
```

Example manifest:

```json
{
  "repositoriesRequiringRemediation": [
    {
      "repoName": "payments-ui",
      "repoUrl": "https://github.example.com/team/payments-ui",
      "pastDue": 2
    },
    {
      "repoName": "account-service",
      "repoUrl": "https://github.example.com/team/account-service",
      "pastDue": 1
    }
  ],
  "repositoryCount": 2,
  "totalPastDueActions": 3
}
```

### Remediation Phase

The Remediation Agent reads only the discovery manifest. It should not need to inspect the original large payload again.

#### Remediation Agent Instructions

Read:

```text
.agent-output/past-due-repositories.json
```

Process only the repositories listed in `repositoriesRequiringRemediation`. For each repository:

1. Read `repoName`, `repoUrl`, and `pastDue`.
2. Access the repository using `repoUrl`.
3. Determine which actions are past due.
4. Verify that the number of identified overdue actions is consistent with `pastDue`.
5. Resolve only the confirmed overdue actions.
6. Run the appropriate validation, tests, linting, and type-checking.
7. Record the result for that repository.

Do not process repositories absent from the manifest. Do not assume what an overdue action is without evidence from the repository or associated tooling. Do not make unrelated changes or silently report success when validation fails.

Return valid JSON in this format:

```json
{
  "results": [
    {
      "repoName": "example-repository",
      "repoUrl": "https://example.com/example-repository",
      "expectedPastDueActions": 2,
      "identifiedActions": [],
      "resolvedActions": [],
      "status": "resolved",
      "validation": {
        "testsPassed": true,
        "lintPassed": true,
        "typeCheckPassed": true
      },
      "notes": []
    }
  ]
}
```

### Recommended Control Flow

```text
API payload
    ↓
Discovery Agent
    ↓
past-due-repositories.json
    ↓
Human or automated validation gate
    ↓
Remediation Agent
    ↓
remediation-results.json
```

Before remediation starts, verify:

- The repository count is reasonable.
- `repoName` and `repoUrl` are present.
- `repoUrl` points to an allowed host.
- `pastDue` is a positive integer.
- There are no duplicate repositories.

### TypeScript Handoff Model

```ts
type RepositoryRequiringRemediation = {
  repoName: string;
  repoUrl: string;
  pastDue: number;
};

type DiscoveryManifest = {
  repositoriesRequiringRemediation: RepositoryRequiringRemediation[];
  repositoryCount: number;
  totalPastDueActions: number;
};
```

> **Key design rule:** Discovery decides where work is required. Remediation decides what changes are required and performs them.

This creates a stable audit trail and prevents the Implementation Agent from jumping directly from a large payload into uncontrolled repository changes.
