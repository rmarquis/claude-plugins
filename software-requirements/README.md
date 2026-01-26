# Software Requirements Plugin

A Claude Code plugin that transforms informal feature descriptions into structured software requirements documents.

## Features

- **Interactive refinement**: Asks clarifying questions to gather missing details
- **Proactive triggering**: Activates when you describe what you want to build
- **Structured output**: Generates requirements documents with standard sections

## Components

- **Command**: `/refine-requirements` - Explicitly start requirements refinement
- **Agent**: `requirements-refiner` - Autonomous agent for requirements analysis
- **Skill**: `requirements-engineering` - Knowledge for structuring requirements

## Usage

### Explicit Command

```
/refine-requirements
```

Then describe what you want to build.

### Proactive Triggering

Simply describe what you want to build:

- "I want to build a user authentication system..."
- "We need a feature that handles file uploads..."
- "The system should process payments..."

The agent will activate and guide you through requirements refinement.

## Output

Requirements are saved to `docs/requirements/<feature-name>.md` with these sections:

1. **Overview** - Brief description and goals
2. **Functional Requirements** - What the system must do
3. **Non-Functional Requirements** - Performance, security, scalability
4. **Constraints** - Technical or business limitations
5. **Assumptions** - Things assumed to be true
6. **Open Questions** - Items needing further clarification

## Installation

### Local Development

```bash
claude --plugin-dir "C:/Users/Remy/Documents/Projects/software-requirements"
```

### Project Installation

Copy to your project's `.claude-plugin/` directory.
