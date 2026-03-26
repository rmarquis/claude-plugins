# claude-plugins

Claude Code plugins for specification-driven development.

## Plugins

### software-requirements

Transform informal feature descriptions into structured requirements documents.

- `/requirements-refine` — interactively refine requirements with clarifying questions
- Skill: `requirements-engineering` | Agent: `requirements-refiner`
- Output: `docs/requirements/<feature>.md`

### software-architecture

Design internal software architecture using Ousterhout's "A Philosophy of Software Design" principles.

- `/architecture-design` — design module structure from requirements
- Skill: `architecture-design` | Agent: `architecture-designer`
- Output: `docs/architecture/<feature>.md`

### software-specification

Generate executable kotlin-test specifications (contracts, behaviors, properties) before implementation.

- `/specification-write` — generate specs from architecture and requirements
- Skill: `specification-engineering` | Agent: `specification-writer`
- Output: `docs/specifications/<feature>/`

### kotlin-functional

Generate and review functional-first, idiomatic Kotlin implementations with KDoc.

- `/functional-implement` — generate implementation from specifications
- `/functional-review` — review code for functional patterns and idiomatic usage
- Skill: `functional-idioms` | Agents: `functional-implementer`, `functional-reviewer`
- Output: `src/main/kotlin/<package>/`

## Workflow

```
requirements → architecture → specification → implementation
     ↓              ↓               ↓               ↓
  /requirements   /architecture   /specification   /functional
    -refine         -design         -write           -implement
```

## Installation

Copy a plugin directory to your project's `.claude-plugin/` directory, or point Claude Code at it:

```bash
claude --plugin-dir path/to/plugin
```
