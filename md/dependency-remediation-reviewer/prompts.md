Create two GitHub Copilot custom agents for this repository:

- `.github/agents/dependency-remediation.agent.md`
- `.github/agents/dependency-reviewer.agent.md`

The purpose is to automate dependency audit remediation across projects.

The `dependency-remediation` agent should accept a list of dependencies that need upgrading, inspect how each dependency is managed, perform the upgrades, fix compatibility issues caused by those upgrades, run the appropriate build/tests/typecheck/lint commands, and continue through the entire list until each item is either successfully remediated or genuinely blocked.

It must understand direct dependencies, transitive dependencies, BOMs, parent POMs, dependency management, peer dependencies, and coupled dependency groups. It should prefer upgrading the controlling parent/BOM rather than adding unnecessary individual version overrides.

The `dependency-reviewer` agent should independently review the remediation afterward. It should compare the final repository state against the original audit list, verify that the required versions are actually resolved, inspect the diff for unsafe or unnecessary changes, confirm tests were not disabled or weakened, and report PASS/FAIL/BLOCKED for every audit item.

Before creating the agents:

1. Inspect this repository.
2. Identify the build system, package managers, languages, test commands, lint/typecheck commands, and existing Copilot instructions.
3. Inspect any existing `.github/agents`, `AGENTS.md`, or `.github/copilot-instructions.md` files.
4. Use the custom-agent format actually supported by the installed Copilot environment.
5. Do not guess unsupported model names or frontmatter fields.

For now, do NOT upgrade any dependencies or modify application source code.

Only:

1. create the two agent files,
2. validate their configuration,
3. show me their final contents,
4. explain how to invoke them,
5. give me an example prompt for supplying a dependency audit list.
