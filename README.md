# AgenticDesignReview

A set of Claude Code skills that audit a code base to spot software design issues.

## Installation

Requires [Claude Code](https://claude.com/claude-code). Paste both commands:

```
/plugin marketplace add jeanguillaumelef/AgenticDesignReview
/plugin install agentic-design-review@agentic-design-review
```

## Usage

Open Claude Code in the repository you want to audit and run:

```
/design-review
```

It runs every available family skill and produces one report: findings grouped by High, Medium and Low, each with file and symbol evidence and a confidence level. The report header states which families were assessed and which were not built yet, so you know what it does not cover. Nothing in your code is modified.

You can also run a single family, for example `/design-review-coupling`.

Optionally, add a `design-context.md` at your repository root to declare your intended architecture and deliberate exceptions. See [the template](design-issues/design-context.template.md).

## Skills

| Skill | Family | Status |
|---|---|---|
| `/design-review` | Runs all available families and aggregates the report | Built |
| `/design-review-coupling` | [01 Coupling](design-issues/families/01-coupling.md) | Built |
| Cohesion and responsibility | [02](design-issues/families/02-cohesion-and-responsibility.md) | Planned |
| Abstraction and inheritance | [03](design-issues/families/03-abstraction-and-inheritance.md) | Planned |
| Encapsulation and data | [04](design-issues/families/04-encapsulation-and-data.md) | Planned |
| Dependency direction and modularity | [05](design-issues/families/05-dependency-direction-and-modularity.md) | Planned |
| Knowledge duplication | [06](design-issues/families/06-knowledge-duplication.md) | Planned |
| Change patterns (git history) | [07](design-issues/families/07-change-patterns.md) | Planned |
| Domain modelling | [08](design-issues/families/08-domain-modelling.md) | Planned |
| Test design | [09](design-issues/families/09-test-design.md) | Planned |
| Error handling | [10](design-issues/families/10-error-handling.md) | Planned |

## Intent

### The problem

Software design problems build up quietly. Linters and tests don't see them, and a diff review doesn't show them. By the time they hurt, they are expensive to fix. In my career, design issues have always been overlooked or hard to spot by hand, and finding them meant a senior engineer reading the whole codebase, which is slow, rare and subjective.

This became obvious when I started generating code with AI: design issues appeared faster. It also showed me that AI could be the way to find them.

### What this is

AgenticDesignReview is a set of Claude Code skills, distributed as a plugin. You run it on demand against a repository. It audits the code, the tests and the change history, and it produces a prioritized, evidence-backed report of software design issues.

It is for any engineer, who maintains, inherits or wants a periodic design health check of a codebase. It audits one repository at a time.

### What makes it different

Static analyzers are rule-based, language-specific and need configuration. This tool uses an LLM to reason about intent and design principles, so it also catches semantic problems such as a leaky abstraction or a misplaced responsibility. It does the analysis itself and doesn't delegate to other tools.

It needs no setup. It reads your README and docs first (`docs/`, architecture docs, ADRs) so it assumes as little as possible. It states every assumption it still has to make. You can optionally provide a `design-context.md` to declare your intended architecture and deliberate exceptions, and the tool can draft one from its own assumptions for you to correct.

It uses git history as evidence. Findings in files that change often are ranked higher.

### Principles

- **Read-only and advisory**: it never modifies your code, and it is not a gate. Severity (High, Medium, Low) reflects the cost of change.
- **Evidence required**: every finding cites concrete files, symbols and dependencies, and carries a confidence level. A finding with no evidence is not reported.
- **Precision over coverage**: success is measured by the share of findings a human agrees are real, not by the number of findings.
- **Stable where it matters**: High findings are the ones to trust. The design aims to make them reproducible across runs by relying on measurable signals and by requiring more than one signal to corroborate a High finding. Uncertain findings are reported at a lower severity.
- **Honest about coverage**: every report states what was assessed, what couldn't be assessed (for example, issues that need runtime data), and which issue families didn't apply.
- **Prioritized, not exhaustive**: on large repositories, it focuses on where change happens and doesn't promise to read every file.

### Scope

The goal is a design audit that is language- and paradigm-agnostic. Today the catalogue is object-oriented first and has been exercised mainly on object-oriented codebases, so results on other paradigms will be less complete.

Out of scope: auto-fixing or refactoring code, reviewing PR diffs, replacing security or test tooling, and enforcing a single "correct" architecture style.
