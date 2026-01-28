# Software Specification Plugin

Bridge architecture and implementation through executable test specifications in Kotlin using `kotlin-test`.

## Overview

This plugin implements **specification-first development**: write executable specifications BEFORE implementation. Specifications serve as:

- **Formal contracts** that interfaces must satisfy
- **Behavioral documentation** that is always up-to-date
- **Property invariants** that must hold for all inputs

Specifications are derived from architecture documents (produced by `software-architecture`) and requirements documents (produced by `software-requirements`).

## Components

- **Command**: `/specify` - Generate executable specifications from architecture and requirements
- **Agent**: `specification-writer` - Autonomous agent for writing test specifications
- **Skill**: `executable-specifications` - Knowledge for writing contract, behavior, and property-based specs

## Usage

### Command

```
/specify [architecture-file-or-module-name]
```

Reads architecture and requirements documents, then generates executable specifications saved to `docs/specifications/<feature-name>/`.

### Proactive Triggering

The agent activates when you discuss specification-driven development:

- "Write specifications for the user repository interface"
- "Create contract tests before implementing"
- "Formalize the module interfaces with tests"

## Output

Specifications are saved to `docs/specifications/<feature-name>/` with this structure:

```
docs/specifications/<feature-name>/
├── contracts/           # Interface contract specifications
├── behaviors/           # BDD-style behavior specifications
└── properties/          # Property-based specifications
```

Each file is a `.kt` specification file using `kotlin-test` that can later be moved to the test source set for compilation.

### Three Specification Levels

1. **Contract Specs** - What each interface method must do (pre/post-conditions, return guarantees, error conditions)
2. **Behavior Specs** - Given-When-Then style, derived from acceptance criteria
3. **Property Specs** - Invariants that must always hold (roundtrip, idempotence, commutativity)

## Prerequisites

- Architecture document in `docs/architecture/` (use `software-architecture` plugin)
- Requirements document in `docs/requirements/` (use `software-requirements` plugin)

## Integration

This plugin completes the specification-first workflow:

1. Use `/refine-requirements` to create structured requirements
2. Use `/design-architecture` to design the architecture
3. Use `/specify` to generate executable specifications
4. Implement against the specifications in `src/main/kotlin/` + `src/test/kotlin/`

## Design Decisions

- **Output to `docs/specifications/`** rather than test source set - specs serve as documentation first, moved to tests when implementation begins
- **Property generators via `kotlin.random`** - no external dependencies, lightweight extension functions on `Random`
- **kotlin-test only** - standard library, no Kotest or JUnit 5 dependency
