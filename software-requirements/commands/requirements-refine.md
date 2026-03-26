---
description: Transform informal feature descriptions into structured requirements
argument-hint: [initial description (optional)]
allowed-tools: Read, Write, AskUserQuestion, Glob
---

Start the requirements refinement process for a software feature or system.

If an initial description was provided ($ARGUMENTS), analyze it. Otherwise, ask the user to describe what they want to build.

**Process:**

1. **Gather Initial Description**
   - If $ARGUMENTS provided, use it as starting point
   - Otherwise, prompt user: "Please describe the feature or system you want to build"

2. **Analyze the Description**
   - Identify core functionality and goals
   - Note implied requirements
   - Find ambiguities and vague terms
   - List missing information

3. **Ask Clarifying Questions**
   - Use AskUserQuestion to ask 3-5 focused questions
   - Prioritize questions that unblock multiple requirements
   - Cover: users/roles, scale, integrations, error handling, security
   - Offer concrete options when possible

4. **Generate Requirements Document**
   - Create structured markdown with these sections:
     - Overview: Brief description and goals
     - Functional Requirements: FR-1, FR-2, etc. with description, acceptance criteria, priority
     - Non-Functional Requirements: Performance, security, reliability
     - Constraints: Technical and business limitations
     - Assumptions: Dependencies and assumed context
     - Open Questions: Items needing further clarification
   - Use priority levels: Must Have, Should Have, Nice to Have
   - Include specific, measurable acceptance criteria

5. **Save the Document**
   - Create `docs/requirements/` directory if it doesn't exist
   - Save as `docs/requirements/<feature-name>.md`
   - Use kebab-case filename derived from feature name

6. **Review with User**
   - Present summary of generated requirements
   - Ask if any sections need revision
   - Make adjustments as requested

Use the requirements-engineering skill for guidance on structuring requirements and question patterns.
