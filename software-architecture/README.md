# Software Architecture Plugin

Design internal software architecture using principles from "A Philosophy of Software Design" by John Ousterhout.

## Overview

This plugin takes structured requirements documents (produced by the `software-requirements` plugin) and designs software architecture that minimizes complexity through:

- **Deep modules**: Simple interfaces hiding complex implementations
- **Information hiding**: Encapsulating complexity within modules
- **Different layer, different abstraction**: Each layer adds meaningful abstraction
- **Pull complexity downward**: Making things easier for callers
- **Define errors out of existence**: Designing APIs that can't fail

## Usage

### Command

```
/design-architecture [requirements-file-or-feature-name]
```

Analyzes requirements and interactively designs architecture, saving to `docs/architecture/`.

### Agent

The `architecture-designer` agent triggers automatically when you discuss designing architecture after requirements are complete:

- "Design the architecture for user authentication"
- "Architect the payment processing system"
- "Create architecture from the requirements"

## Output

Architecture documents include:

- Module structure with Mermaid diagrams
- Responsibility and interface definitions
- Module depth analysis (Deep/Medium/Shallow)
- Design decisions and trade-offs
- Complexity analysis

## Prerequisites

- Requirements document in `docs/requirements/` (use `software-requirements` plugin)

## Integration with software-requirements

This plugin is designed to work after `software-requirements`:

1. Use `/refine-requirements` to create structured requirements
2. Use `/design-architecture` to design the architecture
3. Architecture references and implements the requirements
