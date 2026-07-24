# UI Styling Agent Guide

A dedicated **UI Styling Agent** is a good fit for applying a reference application's visual standards to an MVP without copying the reference application's implementation.

The key distinction is to separate **visual rules** from **reference implementation details**.

## Agent Responsibilities

The UI Styling Agent should:

- Read the reference application's `styles.md` before making changes.
- Treat `styles.md` as the primary styling source of truth.
- Inspect the MVP component being modified.
- Reuse the MVP's existing architecture, components, state management, and conventions.
- Change markup only when necessary to support styling or accessibility.
- Add styles using the MVP's existing styling system.
- Match the reference application's:
  - Colors
  - Spacing
  - Typography
  - Borders
  - Shadows
  - Corner radii
  - Interaction states
  - Responsive behavior
  - Component density
- Avoid copying React source code from the reference application.
- Avoid introducing unrelated functionality or redesigning the component.
- Report styling rules that are missing or ambiguous in `styles.md`.

## Suggested Project Structure

```text
project/
├── agents/
│   ├── planner.agent.md
│   ├── discovery.agent.md
│   ├── remediation.agent.md
│   └── ui-styling.agent.md
├── reference/
│   └── styles.md
├── repos/
│   ├── reference-app/
│   └── mvp-app/
└── tasks/
    └── styling-task.md
```

The complete reference application may not be necessary if `styles.md` is detailed enough. Using the documented style guide instead of the full implementation reduces the risk of copying architectural patterns.

## `ui-styling.agent.md`

```markdown
# UI Styling Agent

## Purpose

Update React components in the MVP application so they visually match the styling standards documented in the reference application's `styles.md`.

The goal is visual consistency, not implementation consistency.

## Primary Source of Truth

The reference application's `styles.md` file is authoritative for:

- Colors
- Typography
- Spacing
- Borders
- Border radii
- Shadows
- Component sizing
- Interaction states
- Responsive behavior
- Layout conventions
- Visual hierarchy
- Accessibility-related visual behavior

Read `styles.md` before modifying any component.

Do not infer styling conventions from the reference application's React implementation when the relevant rule is already documented in `styles.md`.

## Core Responsibilities

1. Inspect the requested MVP component and its surrounding UI.
2. Identify the applicable rules in `styles.md`.
3. Modify the MVP component so it follows those rules.
4. Preserve the MVP application's existing architecture and coding conventions.
5. Reuse existing MVP components, design tokens, utilities, and shared styles when appropriate.
6. Add or update tests only when the existing project normally tests the affected behavior.
7. Verify hover, focus, active, disabled, loading, error, and responsive states when applicable.
8. Summarize the visual changes and identify any unresolved styling ambiguity.
```

## Implementation Constraints

### Preserve MVP Architecture

Continue using the MVP application's existing:

- React component structure
- TypeScript conventions
- State management
- Routing
- Data-fetching patterns
- Form libraries
- Component libraries
- Folder structure
- Styling system
- Testing framework

Do not restructure the MVP application merely to resemble the reference application.

### Do Not Copy the Reference Implementation

Do not copy or reproduce the reference application's:

- React component source
- Component hierarchy
- Hooks
- State management
- Utility functions
- Folder structure
- Naming conventions
- CSS architecture
- Class names
- Implementation-specific markup
- Business logic

The visual result may be matched, but the implementation must remain native to the MVP.

### Styling Changes Only

Do not introduce new product functionality unless it is explicitly required by the task.

Markup may be changed only when necessary for:

- Styling
- Layout
- Semantic HTML
- Accessibility
- Responsive behavior
- Required component states

Do not modify API contracts, business logic, application flows, or data models unless the styling task explicitly requires it.

## Styling Decision Order

When implementing a visual change, use this priority order:

1. Existing MVP design-system component
2. Existing MVP shared style or design token
3. Existing MVP utility class
4. Extension of an existing MVP component
5. New reusable MVP style or component
6. Component-specific styling as a last resort

Do not add duplicate colors, spacing values, or typography values when equivalent MVP tokens already exist.

## Example: Adding a Button

When a task requires adding a button:

1. Determine the appropriate button variant from `styles.md`.
2. Reuse the MVP's existing button component when one exists.
3. Apply the reference styling standard to the MVP button component or variant.
4. Match the documented:
   - Height
   - Padding
   - Font size
   - Font weight
   - Border radius
   - Foreground color
   - Background color
   - Border
   - Icon spacing
   - Hover state
   - `:focus-visible` state
   - Active state
   - Disabled state
   - Loading state
5. Do not copy the reference application's button component or JSX.
6. Do not introduce a one-off button style when a reusable variant is appropriate.

## React Requirements

- Use React functional components.
- Use TypeScript and TSX.
- Do not use `React.FC` or `FC`.
- Preserve strong typing.
- Avoid unnecessary state.
- Avoid inline styles unless the MVP already uses them or a value must be calculated dynamically.
- Prefer semantic HTML.
- Preserve or improve keyboard and screen-reader accessibility.

## Visual Validation

Before completing the task, verify that:

- The modified component follows the relevant `styles.md` rules.
- Spacing is consistent with nearby MVP components.
- Typography and colors use the correct tokens.
- Interactive states are implemented.
- Focus indicators remain visible.
- Responsive layouts do not overflow.
- Existing functionality still works.
- No unrelated components were modified.
- No reference React code was copied.

## Handling Missing Rules

When `styles.md` does not define a required visual detail:

1. Use the closest documented component standard.
2. Follow the existing MVP design system.
3. Avoid inventing a new styling convention unnecessarily.
4. Clearly report the assumption made.

Do not silently claim an exact visual match when required styling information is missing.

## Required Output

For each task, return the following sections.

### Files Changed

List every modified file.

### Styling Rules Applied

List the relevant sections or rules from `styles.md`.

### Implementation Summary

Explain what changed in the MVP application.

### Assumptions

List decisions made because `styles.md` was incomplete or ambiguous.

### Validation

Report the checks performed, including:

- Type checking
- Linting
- Tests
- Build
- Responsive review
- Interactive-state review

### Out-of-Scope Findings

Mention unrelated styling or implementation issues without fixing them.
```

## Recommended Task Format

Each styling task should clearly identify the MVP component and its expected visual role.

```markdown
# Styling Task

## Target Application

`repos/mvp-app`

## Styling Reference

`reference/styles.md`

## Target Component

`src/features/repositories/components/RepositoryTable.tsx`

## Request

Add a primary action button labeled "Remediate selected".

The button should follow the primary button styling standard from
`styles.md`.

## Functional Requirements

- Disable the button when no repository is selected.
- Invoke the existing `onRemediateSelected` callback when clicked.
- Do not change the repository selection logic.

## Constraints

- Use the MVP application's existing Button component.
- Do not copy the reference application's button component.
- Do not make unrelated table changes.
```

## Important Distinction

Avoid instructions such as:

> Make this component the same as the reference component.

That wording encourages structural imitation.

Use:

> Apply the relevant visual standards from `styles.md` to this MVP component while preserving the MVP's architecture and behavior.

## Improving `styles.md` for Agent Use

The agent will perform better when `styles.md` contains explicit, measurable rules rather than subjective descriptions.

### Weak Guidance

```markdown
Buttons should look clean and modern.
```

### Better Guidance

```markdown
## Primary Button

- Height: 40px
- Horizontal padding: 16px
- Border radius: 8px
- Background: `--color-action-primary`
- Text color: `--color-text-on-primary`
- Font size: 14px
- Font weight: 600
- Icon gap: 8px
- Hover: darken background by one action shade
- Focus-visible: 2px outline using `--color-focus`
- Disabled opacity: 0.45
- Disabled cursor: `not-allowed`
```

Ideally, each component standard in `styles.md` should cover:

- Anatomy
- Tokens
- Dimensions
- Variants
- States
- Responsive behavior
- Accessibility requirements
- Correct-usage examples
- Anti-patterns

This makes the agent a **design-standard translator** rather than a reference-code copier.
