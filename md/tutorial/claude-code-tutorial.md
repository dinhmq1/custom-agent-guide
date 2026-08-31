# Claude Code Tutorial for Experienced Software Engineers

The best mental model is:

> **Claude Code is an AI software engineer operating inside your repository through the terminal.**

It can inspect your codebase, search across files, run commands and tests, modify multiple files, reason about architecture, work from project instructions, and delegate parts of a task to specialized agents.

## 1. The Basic Workflow

Start Claude Code from the root of your repository:

```bash
cd ~/projects/my-app
claude
```

Then interact with it conversationally:

```text
Explain the architecture of this repository.
Do not modify anything yet.
```

This is a good first command because Claude will inspect the project instead of immediately writing code.

Then try:

```text
Trace the flow for creating a recipe from the React UI
through the API and database.
```

For a larger codebase, Claude Code is particularly useful because it can follow relationships across components, services, types, tests, configuration, and infrastructure instead of only looking at the file currently open in your IDE.

## 2. Prompt in Phases

Avoid starting with:

```text
Build the feature.
```

Instead, begin with discovery:

```text
Investigate how authentication currently works.

Do not modify anything.

Identify:
- relevant files
- data flow
- existing conventions
- potential issues
```

Once you understand the findings, request a plan:

```text
Create an implementation plan.

Do not modify files yet.

Include:
1. files that need modification
2. types/interfaces that need changes
3. API changes
4. test changes
5. potential regressions
```

Then implement:

```text
Implement the plan.

Follow existing project patterns.
Do not introduce new dependencies unless necessary.
Run the relevant tests afterward.
```

The reliable workflow is:

```text
DISCOVER
   ↓
PLAN
   ↓
IMPLEMENT
   ↓
TEST
   ↓
REVIEW
```

This is dramatically more reliable than telling an agent to code blindly.

## 3. Inspect and Trace the Repository

Suppose your project contains:

```text
src/
├── components/
├── hooks/
├── pages/
├── services/
├── types/
└── utils/
```

Ask:

```text
Find every place where Recipe is created, updated,
or transformed.

Give me the dependency chain.
Do not change anything.
```

Claude might determine a flow such as:

```text
RecipeForm.tsx
     ↓
useCreateRecipe.ts
     ↓
recipeService.ts
     ↓
POST /recipes
     ↓
recipeController.ts
     ↓
recipeService.ts
     ↓
Supabase
```

This ability to **trace systems** is one of Claude Code's strongest uses.

## 4. Give Claude Permanent Project Instructions

A major feature is `CLAUDE.md`. Put one in your repository root:

```text
my-app/
├── CLAUDE.md
├── package.json
├── src/
└── ...
```

For a React and TypeScript application, it might contain:

````markdown
# Project Guidelines

## Stack

- React
- TypeScript
- Vite
- Supabase
- CSS Modules
- Vitest

## React

- Use functional components.
- Do not use React.FC.
- Use TypeScript.
- Prefer composition over large components.
- Keep components focused on presentation where practical.

## TypeScript

- Never use `any` unless absolutely necessary.
- Prefer explicit domain types.
- Avoid unnecessary type assertions.
- Prefer discriminated unions for complex state.

## Architecture

UI components must not call Supabase directly.

Use:

Component → Hook → Service → Supabase

## Testing

Run:

```bash
bun test
```

before considering a task complete.

## Working Style

Before implementing large changes:

1. Inspect the existing implementation.
2. Explain the findings.
3. Propose a plan.
4. Implement it.
5. Run tests.
6. Review the diff.
````

Now you do not have to repeat these instructions constantly.

## 5. Ask Claude to Create `CLAUDE.md`

Claude can bootstrap the file for you:

```text
Analyze this repository and create a CLAUDE.md.

Capture:
- architecture
- coding conventions
- commands
- testing practices
- directory conventions
- important constraints

Keep it concise and useful for future coding agents.
```

Inspect the generated file. Do not accept it blindly; Claude may infer conventions incorrectly.

## 6. Request Targeted Modifications

Instead of:

```text
Fix RecipeForm.
```

Give it explicit requirements:

```text
Update RecipeForm.tsx.

Requirements:
- Add optional prep time.
- Store minutes as a number.
- Reject negative values.
- Do not introduce new dependencies.
- Follow the existing form validation pattern.
- Update relevant tests.
- Do not modify unrelated files.
```

Specificity dramatically reduces agent drift.

## 7. Ask Claude to Find the Existing Pattern

Before asking Claude to build something new:

```text
Find an existing feature in this repository that most closely
matches what we're about to build.

Explain which patterns we should reuse.
Do not modify anything.
```

Then:

```text
Implement the new feature using that pattern.
```

For enterprise development, this works extremely well because existing conventions often matter more than theoretically ideal architecture.

## 8. Use Claude as a Debugger

Suppose you encounter:

```text
TypeError: Cannot read properties of undefined
```

Do not simply paste the error and say “fix.” Try:

```text
Investigate this error.

Do not modify code yet.

Determine:
1. where the undefined value originates
2. why TypeScript didn't catch it
3. the smallest correct fix
4. whether similar bugs exist elsewhere
```

Then:

```text
Implement the smallest safe fix and add a regression test.
```

This keeps Claude from rewriting half the component unnecessarily.

## 9. Make Claude Run Tests

After implementation:

```text
Run the relevant unit tests.

If anything fails:
- determine whether your change caused it
- fix regressions
- do not change unrelated tests merely to make them pass
```

Then:

```text
Run lint and TypeScript checks.
```

For example:

```bash
bun test
bun run lint
bun run typecheck
```

Claude Code can execute these commands itself when permissions allow it.

## 10. Make Claude Review Its Own Work

After it finishes:

```text
Review your implementation as if you were a senior engineer
reviewing someone else's pull request.

Look specifically for:
- incorrect assumptions
- edge cases
- unnecessary complexity
- TypeScript issues
- React rendering problems
- state synchronization bugs
- missing tests
- accidental scope expansion

Do not modify anything yet.
```

Then:

```text
Fix only the issues you identified that are genuinely worth fixing.
```

This second reasoning pass frequently catches problems.

## 11. Review the Git Diff

A useful final step:

```text
Review the git diff.

For every modified file, explain:
- why it changed
- what behavior changed
- any risks introduced

Flag anything unrelated to the task.
```

Or simply:

```text
Review git diff for accidental changes.
```

This is one of the easiest ways to keep autonomous coding under control.

## 12. Work with Git

A workflow can look like:

```text
Implement issue #123.
```

Then:

```text
Show me the diff.
```

```text
Review the diff for bugs.
```

```text
Generate a concise commit message.
```

Example:

```text
feat(recipes): add preparation time field
```

You can also ask:

```text
Generate a PR description containing:

## Summary
## Changes
## Testing
## Risks
```

## 13. Manage Context Deliberately

A Claude Code session can accumulate a lot of information. If you spend an hour debugging something and then switch to an unrelated feature, starting a fresh session can improve results.

Think of context as:

```text
Useful context
+ Old assumptions
+ Earlier failed experiments
+ Conversation history
```

Too much unrelated history can increase drift. When practical, use:

```text
1 feature = 1 focused Claude session
```

## 14. Let Claude Explore Before Naming Files

Developers often prompt:

```text
Modify src/components/RecipeForm.tsx
```

when they think they know where the change belongs. Often it is better to say:

```text
Find where recipe creation is implemented and determine
which files should change.
```

Claude may discover:

```text
RecipeForm.tsx
useRecipeForm.ts
recipeSchema.ts
recipeService.ts
recipe.test.ts
```

That repository-level reasoning is exactly why Claude Code is more useful than ordinary autocomplete.

## 15. Constrain Its Scope

Autonomy without boundaries is dangerous. Useful guardrails include:

```text
Do not refactor unrelated code.
```

```text
Prefer the smallest correct change.
```

```text
Do not introduce new dependencies.
```

```text
Do not modify public interfaces unless necessary.
```

```text
Ask before changing architecture.
```

```text
Do not change tests simply to accommodate incorrect behavior.
```

## 16. Use Claude Code for Refactoring

A good refactoring prompt:

```text
Analyze RecipePlanner.tsx.

Do not modify anything yet.

Identify:
- responsibilities
- state ownership
- derived state
- side effects
- potential component boundaries
- hooks that could be extracted

Then propose a refactoring plan that preserves behavior.
```

Then:

```text
Perform the refactor incrementally.

After each meaningful step:
- ensure TypeScript passes
- ensure tests still pass

Do not change external behavior.
```

This is much safer than:

```text
Clean this up.
```

## 17. Learn Unfamiliar Codebases

For example:

```text
Teach me this repository.

Start with the highest-level architecture and then walk down
into the frontend.

Explain it as if I'm joining the team as a senior engineer.
```

Then drill down:

```text
Explain the state management architecture.
```

```text
Explain how server state differs from UI state here.
```

```text
Show me where domain boundaries are weak.
```

This turns Claude Code into an interactive codebase onboarding tool.

## 18. Use Specialized Agents

For sufficiently large projects, you can separate responsibilities:

```text
.github/
└── agents/
    ├── architect.agent.md
    ├── frontend.agent.md
    ├── backend.agent.md
    ├── test.agent.md
    └── reviewer.agent.md
```

Conceptually:

```text
                 Orchestrator
                      │
       ┌──────────────┼──────────────┐
       ↓              ↓              ↓
 Architect        Frontend        Backend
       │              │              │
       └──────────────┼──────────────┘
                      ↓
                    Tests
                      ↓
                   Reviewer
```

An architecture agent could receive instructions such as:

```text
You are responsible for system design.

Do not implement features.

Responsibilities:
- inspect architecture
- identify affected components
- identify boundaries
- detect coupling
- produce implementation plans

Your output should contain:
1. Current architecture
2. Required changes
3. Files affected
4. Risks
5. Recommended implementation order
```

Another agent can then perform the implementation.

## 19. Upgrade Dependencies Safely

Instead of:

```text
Upgrade everything.
```

Use:

```text
Analyze this dependency upgrade:

react-router-dom
current: 6.22
target: latest approved version

Do not modify anything.

Determine:
1. breaking changes
2. affected files
3. APIs we currently use that changed
4. migration requirements
5. test coverage that exercises those APIs
6. risk level
```

Then:

```text
Produce a remediation plan.
```

Then:

```text
Execute the remediation.

Rules:
- modify only necessary files
- do not suppress TypeScript errors
- do not remove tests to get green
- run unit tests
- run typecheck
- report unresolved issues
```

This workflow can eventually be automated:

```text
dependency discovery
        ↓
compatibility analysis
        ↓
remediation
        ↓
build
        ↓
tests
        ↓
review
```

This is exactly the type of repetitive engineering task where Claude Code becomes extremely valuable.

## 20. Use Built-In Help and Interaction Patterns

Claude Code's capabilities and commands continue to evolve, so use its built-in help rather than memorizing every slash command:

```text
/help
```

You will commonly interact through a combination of natural-language instructions, slash commands, tool permissions, repository instructions such as `CLAUDE.md`, and specialized skills or agents.

## 21. Treat Permissions Carefully

Claude may need permission to:

- Read files
- Write files
- Execute shell commands
- Run Git commands
- Run tests

Giving an agent unlimited execution power is convenient, but unrestricted mode is not a good default for important repositories. A safer progression is:

```text
Read-only exploration
        ↓
Allow edits
        ↓
Allow safe commands
        ↓
Review diff
```

For personal projects where everything is committed and easily recoverable, you can be more permissive.

## 22. Use Git as a Safety Net

Before giving Claude a large autonomous task, run:

```bash
git status
```

Make sure your work is committed, then let Claude make changes. If things go sideways, inspect them with:

```bash
git diff
```

You can then revert the unwanted changes. A clean Git state makes experimenting with coding agents far less stressful.

## 23. Reusable Claude Code Prompts

### Investigation and Planning

```text
We need to implement the following change:

[REQUIREMENT]

First, investigate the codebase.

Do not modify anything yet.

Determine:
1. current architecture relevant to this feature
2. affected files
3. existing patterns we should follow
4. types/interfaces affected
5. potential edge cases
6. test coverage affected
7. implementation risks

Then propose the smallest reasonable implementation plan.

Wait to implement until the analysis and plan are complete.
```

### Implementation and Verification

```text
Implement the plan.

Requirements:
- follow existing architecture
- use TypeScript
- do not introduce unnecessary dependencies
- avoid unrelated refactoring
- preserve backward compatibility where possible
- add or update relevant tests

After implementation:
1. run tests
2. run typecheck
3. review git diff
4. identify any remaining risks
```

This is a much better workflow than repeatedly asking Claude to “fix this.”

## Recommended Learning Progression

### Stage 1 — Repository Understanding

```text
Explore
Search
Trace
Explain
```

### Stage 2 — Controlled Coding

```text
Investigate
Plan
Implement
Test
Review
```

### Stage 3 — Project Memory

```text
CLAUDE.md
architecture rules
coding conventions
commands
```

### Stage 4 — Automation

```text
repeatable prompts
skills
hooks
agents
```

### Stage 5 — Agentic Engineering

```text
discovery agent
        ↓
planning agent
        ↓
implementation agent
        ↓
test agent
        ↓
review agent
```

At this point, Claude Code stops being merely an AI coding assistant and starts acting more like an **engineering workflow engine**.
