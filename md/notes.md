What kind of custom agent should I create after creating the planning agent?

After a Planning Agent, the best next custom agent is an Implementation Agent.

The planning agent decides what should be built. The implementation agent turns that approved plan into working code without quietly expanding scope.

1. Implementation Agent
   Give it responsibilities like:

Read the approved plan before changing anything.
Implement one task or milestone at a time.
Follow your existing architecture, conventions, and design system.
Use React functional components with TypeScript and TSX.
Avoid unrelated refactors.
Add or update tests for meaningful behavior.
Run type-checking, linting, and tests after changes.
Report files changed, decisions made, and unresolved issues.
Stop when requirements are ambiguous rather than inventing features.
A useful operating rule would be:

Implement the approved plan exactly. Make the smallest coherent change that fully satisfies the current task. Do not redesign adjacent code or add speculative functionality.

2. Code Review Agent
   Create this immediately after the implementation agent. It should not write the original implementation because reviewers are more useful when they are independent.

Have it inspect:

Correctness and edge cases
State-management mistakes
React rendering and hook issues
Type safety
Security and authorization boundaries
Accessibility
Test quality
Unnecessary complexity
Performance problems that actually matter
Whether the code matches the approved plan
Require findings to be ranked:

Blocker — unsafe or functionally incorrect
Major — likely bug or maintainability problem
Minor — worthwhile improvement
Optional — preference, not a requirement
This prevents the review from becoming a pile of subjective style comments.

3. Test Agent
   This agent should derive tests from requirements rather than merely increasing coverage.

For your HomeMenu-style application, it could focus on:

Recommendation-engine behavior
Pantry quantity and expiration logic
Grocery-list aggregation
Authentication and authorization
Supabase row-level security assumptions
Form validation
Loading, empty, and error states
Date and timezone boundaries
User interactions rather than implementation details
Its core rule should be:

Test behavior users and business rules depend on. Do not create brittle tests solely to increase coverage percentages.

4. Debugging Agent
   This is different from the implementation agent. Feed it a failure, logs, reproduction steps, and relevant code.

It should:

Reproduce or precisely characterize the failure.
Generate several hypotheses.
Rank them by likelihood.
Gather evidence.
Identify the root cause.
Apply the smallest safe fix.
Add a regression test.
Tell it not to rewrite large sections of code before proving the root cause.

5. Architecture Agent
   Create this later, not immediately. Use it for decisions that cross boundaries, such as:

Supabase versus a dedicated backend
Database schema changes
Recommendation-engine boundaries
Background jobs
Caching
Offline support
Authentication architecture
Deployment and observability
It should produce architecture decision records rather than implementation code:

Context
Decision
Alternatives considered
Trade-offs
Risks
Migration path
How the decision can be reversed
Best agent sequence
A strong workflow would be:

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
I would avoid creating dozens of highly specialized agents early. Start with these three:

Planning Agent
Implementation Agent
Code Review Agent
That gives you genuine separation of concerns: one defines the work, one performs it, and one challenges it. The Implementation Agent should be your next one.

what's a work-around when copilot agent has vision disabled and can't browse links? It's possible to export the page to text some how?

Yes. In fact, one of the best workarounds is to convert the page into Markdown or plain text before handing it to the agent. Most coding agents are dramatically better at reasoning over structured text than screenshots or HTML.

Here are the approaches I recommend, ordered from best to acceptable.

1. Use a Markdown converter (my favorite)
   For web pages:

npx @mizchi/readability https://example.com
or

npx url-to-markdown https://example.com
or

curl https://example.com | pandoc -f html -t markdown
The agent receives:

headings
paragraphs
code blocks
lists
instead of hundreds of lines of HTML.

2. Use Reader Mode
   Most browsers can produce a clean reader view.

Examples:

Safari Reader
Firefox Reader
Edge Immersive Reader
Then:

Copy all
Paste into Copilot Agent
This removes:

ads
navigation
sidebars
CSS
JavaScript
leaving mostly the article.

3. Use a CLI fetcher
   If you're comfortable in the terminal:

curl https://example.com
or

wget -qO- https://example.com
Pipe it through something like:

pandoc
or

html2text
Example:

curl https://example.com \
| html2text 4. Use a browser extension that exports Markdown
Extensions like:

MarkDownload
Copy as Markdown
Markdown Here (for some workflows)
let you right-click:

Save page as Markdown

Then give that .md file to the agent.

5. Use Playwright
   If you're already using Playwright for automation:

const html = await page.content();
or

const text = await page.locator("body").innerText();
You can even create a small script that outputs:

docs/page.md
every time.

6. Use Microsoft's MarkItDown
   Microsoft created an excellent open-source tool called MarkItDown specifically for converting documents and web content into LLM-friendly Markdown.

It can convert:

HTML
PDFs
Word documents
PowerPoint
Excel
many other formats
into clean Markdown that's much easier for agents to consume.

7. Create a preprocessing agent (my recommendation)
   Since you're building a multi-agent workflow, add a lightweight Document Normalizer Agent before your Planning or Implementation Agent.

Its pipeline could be:

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
This avoids making your implementation agent spend tokens parsing noisy HTML.

For software engineering documentation
If you're feeding React, TypeScript, or API documentation to Copilot, I'd use this pipeline:

Website
↓
MarkItDown
↓
Markdown (.md)
↓
Chunk into sections
↓
Planning Agent
↓
Implementation Agent
This preserves headings, code examples, tables (when possible), and links in a format that's much more digestible for coding agents than raw HTML or screenshots.

If you're using GitHub Copilot Agent in VS Code, you can also automate this with a small script or VS Code task that takes a URL, converts it to Markdown, and saves it into a docs/ folder so the agent can reference it as part of your workspace.

ok so i have the json payload that I want the implementing agent to evaluate the repos array of objects and to look for the pastDue property in the adviceSummary object and report back the value if it's greater than 0.

That's a good fit for an implementation agent because the requirements are specific and bounded. I'd phrase the request so the agent knows exactly what to inspect and what not to do.

For example:

Task

Inspect the JSON payload.

The payload contains a `repos` array.

For each object in `repos`:

1. Read the `adviceSummary` object.
2. Check the `pastDue` property.
3. If `pastDue` is greater than 0:
   - Record the repository identifier (use the repository name or id if available).
   - Record the `pastDue` value.
4. Ignore repositories where `pastDue` is missing, null, or 0.

Output

Return a summary containing:

- Total number of repositories with `pastDue > 0`
- Each matching repository
- Its corresponding `pastDue` value

Do not modify the payload.
Do not infer missing values.
Do not make any code changes.
If your agent is actually writing code, make the prompt even more implementation-oriented:

Implement logic that evaluates the `repos` array in the API response.

Requirements:

- Iterate over every repository in `repos`.
- Safely access `repo.adviceSummary?.pastDue`.
- Treat missing or undefined values as 0.
- Detect repositories where `pastDue > 0`.
- Return or expose the matching repositories and their `pastDue` counts.
- Use TypeScript.
- Keep the implementation simple and avoid unnecessary abstractions.
- Do not change unrelated code.
  That gives the agent enough direction without overconstraining the implementation. It can choose whether a simple filter, reduce, or loop is most appropriate while still satisfying the behavior you want.

also the pastDue value means n actions required. So if pastDue is 2 that means 2 actions are past due. Also, return the repoName and repoUrl values as those are required later on when the agent does the implementation to resolve the pastDue actions

I'd include that domain knowledge explicitly in the prompt. The more context you give the implementation agent about what the data means, the less likely it is to make incorrect assumptions.

Here's how I'd write the task:

Task

Evaluate the JSON payload and identify repositories that have overdue actions.

The payload contains a `repos` array.

For each repository:

1. Read the `adviceSummary` object.
2. Read the `pastDue` property.
3. The `pastDue` value represents the number of actions required for that repository.
   - `pastDue = 0` means there are no overdue actions.
   - `pastDue = 2` means there are two overdue actions that must be resolved.
4. Only include repositories where `pastDue > 0`.

For every matching repository, return:

- `repoName`
- `repoUrl`
- `pastDue`

Output

Return an array of objects in the following format:

[
{
"repoName": "...",
"repoUrl": "...",
"pastDue": 2
}
]

Requirements

- Safely handle missing or undefined `adviceSummary` or `pastDue` values.
- Treat missing `pastDue` as 0.
- Do not modify the original payload.
- Do not attempt to resolve any overdue actions yet.
- This task is only responsible for identifying repositories that require action.
- The returned `repoName` and `repoUrl` will be consumed by a later implementation step that performs the remediation.
  One additional suggestion: separate the workflow into discovery and remediation. Have this first step produce a clean list of repositories requiring attention, then let a second step iterate over that list to perform the actual fixes. That separation makes the process easier to test, rerun, and extend if additional action types are added later.

how do I separate the workflow into discovery and remediation?

Separate it into two agents or two explicit phases with a saved handoff artifact between them.

Discovery phase
The discovery step only inspects the payload and produces a structured list of repositories requiring work.

It should not clone repositories, edit files, open pull requests, or attempt remediation.

Discovery Agent Instructions
Evaluate the provided JSON payload.

The payload contains a repos array. For every repository:

Read adviceSummary.pastDue.

The pastDue value represents the number of overdue actions for that repository.

Include the repository only when pastDue > 0.

Return:

repoName

repoUrl

pastDue

Treat a missing, null, invalid, or undefined pastDue value as 0.

Do not modify the payload.
Do not access or modify any repository.
Do not attempt to resolve the overdue actions.

Return valid JSON using this format:

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

The output becomes the discovery manifest. Save it somewhere predictable, such as:

.agent-output/past-due-repositories.json
Example:

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
Remediation phase
The remediation step reads only the discovery manifest. It should not need to inspect the original large payload again.

Remediation Agent Instructions
Read the discovery manifest at:

.agent-output/past-due-repositories.json

Process only the repositories listed in repositoriesRequiringRemediation.

For each repository:

Read repoName, repoUrl, and pastDue.

Access the repository using repoUrl.

Determine which actions are past due.

Verify that the number of identified overdue actions is consistent with pastDue.

Resolve only the confirmed overdue actions.

Run the appropriate validation, tests, linting, and type-checking.

Record the result for that repository.

Do not process repositories that are absent from the discovery manifest.
Do not assume what an overdue action is without evidence from the repository or associated tooling.
Do not make unrelated changes.
Do not silently report success when validation fails.

Return valid JSON using this format:

{
"results": [
{
"repoName": "example-repository",
"repoUrl": "https://example.com/example-repository",
"expectedPastDueActions": 2,
"identifiedActions": [],
"resolvedActions": [],
"status": "resolved | partially-resolved | blocked | failed",
"validation": {
"testsPassed": true,
"lintPassed": true,
"typeCheckPassed": true
},
"notes": []
}
]
}

Recommended control flow
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
The validation gate matters. Before remediation starts, verify:

The repository count is reasonable.
repoName and repoUrl are present.
repoUrl points to an allowed host.
pastDue is a positive integer.
There are no duplicate repositories.
A simple TypeScript model for the handoff would be:

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
The key design rule is:

Discovery decides where work is required. Remediation decides what changes are required and performs them.

This gives you a stable audit trail and prevents the implementation agent from jumping directly from a large payload into uncontrolled repository changes.
