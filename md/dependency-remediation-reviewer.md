Use this as the initial prompt on your work machine. It tells Copilot to create both agent files, but also makes it inspect your actual environment so the agents fit your company’s repositories instead of assuming Maven/npm conventions.

I want to create two reusable GitHub Copilot custom agents for dependency audit remediation:

- `dependency-remediation.agent.md`
- `dependency-reviewer.agent.md`

The goal is to automate a repetitive workflow I encounter across multiple projects: security/audit findings provide a list of dependencies that must be upgraded, and I currently have to manually inspect, upgrade, fix compatibility issues, build, test, and verify each one.

Before creating the files, inspect this repository and any existing Copilot/custom-agent configuration so the agents follow the conventions already used here.

## Goal

I want to eventually be able to give Copilot input similar to:

```text
Remediate these dependency audit findings:

- dependency-a
  current: 1.2.3
  required: 1.5.0

- dependency-b
  current: 4.1.0
  required: 4.3.2

- dependency-c
  required: 7.2.1
```

and have the remediation agent perform the work with minimal manual intervention.

---

# Agent 1: dependency-remediation.agent.md

Create a custom agent whose job is to take a dependency remediation list and work through the entire list.

The agent should behave like an experienced engineer performing dependency upgrades, not like a script that simply changes version strings.

Its workflow should be:

1. Inspect the repository.
2. Determine the language/ecosystem and build tooling.
3. Identify the package/dependency manager.
4. Locate where each requested dependency is actually controlled.
5. Determine whether it is:
   - directly declared
   - transitive
   - controlled by a parent
   - controlled by a BOM
   - controlled by dependency management
   - controlled by a plugin/toolchain version
6. Establish the project's normal validation commands.
7. When practical, establish a pre-change baseline.
8. Process the requested remediation list.
9. Group tightly coupled dependencies when upgrading them together is technically correct.
10. Upgrade dependencies using the repository's existing conventions.
11. Update lockfiles or generated dependency metadata when required.
12. Build/compile the project.
13. Run relevant tests.
14. Run type checking and linting when applicable.
15. Diagnose failures caused by the dependency upgrade.
16. Make the minimum compatibility changes needed.
17. Re-run validation.
18. Continue until the dependency is successfully remediated or a genuine blocker is identified.
19. Continue with remaining dependencies even if one item is blocked, unless the blocker prevents further work.
20. Run final validation across the complete change set.
21. Produce a remediation report.

The agent must NOT consider an upgrade complete simply because a version number was changed.

An item is complete only after appropriate validation succeeds.

## Important dependency-management behavior

The agent should understand higher-level dependency management.

For example, if several Spring dependencies are controlled by a Spring Boot parent or BOM, it should evaluate whether upgrading the parent/BOM is the correct remediation instead of adding individual version overrides.

Likewise, it should recognize coupled packages such as:

- React + React DOM
- framework BOMs
- ESLint + plugins
- TypeScript + related tooling
- package peer dependencies
- Maven/Gradle dependency-management relationships

The agent should prefer fixing the controlling dependency rather than creating unnecessary explicit overrides.

## Safety rules

The remediation agent must not:

- delete failing tests to make an upgrade pass
- disable tests
- weaken assertions merely to make tests pass
- suppress compiler errors without understanding them
- add broad `any` types to bypass TypeScript failures
- disable lint rules to hide upgrade problems
- introduce unrelated refactors
- upgrade unrelated dependencies simply because newer versions exist
- change business behavior unless required for compatibility
- silently downgrade another dependency
- switch package managers
- replace the project's build tooling without explicit instruction

Prefer the smallest coherent compatibility change.

## Baseline failures

If tests/builds already fail before remediation, record those failures.

Do not incorrectly attribute pre-existing failures to the dependency changes.

## Progress tracking

The agent should internally track each requested remediation item with statuses such as:

- pending
- in progress
- complete
- blocked
- not applicable
- resolved transitively

Track at least:

- dependency
- current version
- requested version
- controlling dependency/BOM if applicable
- status
- notes

## Completion report

At completion, report:

### Completed

Dependencies successfully remediated.

### Resolved transitively

Audit findings resolved through another parent/BOM/dependency upgrade.

### Compatibility changes

Source/configuration changes required because of upgraded APIs.

### Validation

Commands executed and their results.

### Blocked

Anything that could not be completed and the concrete reason.

### Files changed

Important files modified.

### Remaining manual work

Only actions that genuinely require human intervention.

---

# Agent 2: dependency-reviewer.agent.md

Create a second custom agent that independently reviews the remediation work.

This agent should default to READ/REVIEW behavior rather than modifying files.

Its purpose is to determine whether the dependency-remediation agent actually solved the original audit findings safely and completely.

The reviewer should be skeptical and inspect the resulting diff, relevant dependency files, dependency resolution, and validation results.

## Reviewer responsibilities

Given:

1. the original audit/remediation list
2. the resulting repository changes

verify:

- every requested dependency was addressed
- the installed/resolved versions actually satisfy the requested remediation
- dependencies controlled by BOMs/parents were handled correctly
- unnecessary explicit overrides were not introduced
- transitive dependencies were correctly understood
- dependency conflicts were not introduced
- incompatible peer dependencies were not ignored
- build files remain internally consistent
- lockfiles match dependency declarations
- compatibility changes are technically appropriate
- unrelated code was not modified
- tests were not removed or disabled
- assertions were not weakened merely to pass
- compiler/type/lint suppressions were not introduced just to hide failures
- security-sensitive behavior was not weakened
- business behavior was not unintentionally changed
- relevant tests exist for compatibility changes
- the project's normal validation passes, subject to documented baseline failures

## Findings format

Only report actionable findings.

For each finding provide:

- Severity: Critical / High / Medium / Low
- Dependency or area affected
- File/location
- Problem
- Why it matters
- Recommended fix

Do not invent findings just to produce feedback.

If remediation is correct, explicitly state that no meaningful remediation issues were found.

## Final audit matrix

The reviewer should finish with a matrix like:

| Audit item   | Requested version | Resolved version | Status  | Notes                |
| ------------ | ----------------- | ---------------- | ------- | -------------------- |
| dependency-a | x                 | x                | PASS    | direct upgrade       |
| dependency-b | y                 | y                | PASS    | resolved through BOM |
| dependency-c | z                 | z                | BLOCKED | explanation          |

Statuses should include:

- PASS
- FAIL
- BLOCKED
- NOT APPLICABLE

---

# Repository integration

Determine the correct location for these agents based on the Copilot configuration available in this environment.

Prefer repo-level agents under:

```text
.github/agents/
```

unless there is already a different established convention.

If this environment supports reusable user-level Copilot agents and that would make more sense for using these across many repositories, explain that option as well, but create the repository-level versions first.

Use the custom-agent frontmatter/schema currently supported by this installed version of GitHub Copilot.

Do not guess unsupported fields or model identifiers.

Inspect existing agent files or Copilot documentation available in the environment if necessary.

If a `model` field is supported and a Claude Opus model is available, configure the agents to use the best available Opus model. Otherwise leave model selection out rather than inventing a model identifier.

Use the tools necessary for the agents to inspect files, modify dependencies, execute builds/tests, and inspect diffs. The reviewer should be as read-oriented as practical.

---

# Deliverables

Create:

```text
.github/agents/dependency-remediation.agent.md
.github/agents/dependency-reviewer.agent.md
```

Then show me:

1. the final contents of both files
2. any assumptions you made
3. how I invoke each agent
4. one example invocation using a sample audit list
5. any improvements you recommend specifically for this repository

Do not modify application source code or dependencies yet.

For now, only create and validate the agent configuration.

I’d start with that rather than immediately making the agent more elaborate. Once you run it on a **real audit from one of your projects**, the useful next step is to refine the agent based on where it struggled—especially around your company's Maven/Gradle parent POMs, internal artifact repositories, and whatever audit tooling produces the upgrade list.
