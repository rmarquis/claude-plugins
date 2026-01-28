---
description: Generate executable test specifications from architecture and requirements documents
argument-hint: [architecture-file-or-module-name]
allowed-tools: Read, Write, AskUserQuestion, Glob
---

Generate executable Kotlin test specifications from architecture and requirements documentation. Specifications serve as formal contracts that must pass before implementation begins.

**Process:**

1. **Locate Architecture Document**
   - If $ARGUMENTS provided, find matching architecture file in `docs/architecture/`
   - If no argument, list available architecture files and ask which to use
   - Read and analyze the architecture document, focusing on module structure and interfaces

2. **Locate Requirements Document**
   - Find the corresponding requirements file in `docs/requirements/`
   - Read and extract acceptance criteria from functional requirements
   - Note non-functional requirements that imply property invariants

3. **Identify Specification Targets**
   For each module in the architecture:
   - Extract interface methods and their responsibilities
   - Map acceptance criteria from requirements to module behaviors
   - Identify invariants and properties (roundtrip, idempotence, commutativity, state validity)

4. **Ask Clarifying Questions**
   Use AskUserQuestion to resolve:
   - Which modules to prioritize for specification
   - Package naming conventions for the project
   - Any additional invariants or edge cases the user wants covered
   - Whether to generate all three levels (contracts, behaviors, properties) or a subset

5. **Generate Contract Specifications**
   For each module interface, create `docs/specifications/<feature-name>/contracts/<ModuleName>ContractSpec.kt`:

   ```kotlin
   package specifications.<feature>.contracts

   import kotlin.test.Test
   import kotlin.test.assertEquals
   import kotlin.test.assertNotNull
   import kotlin.test.assertNull
   import kotlin.test.assertFailsWith

   /**
    * Contract specification for [ModuleName].
    *
    * Verifies the formal contract each implementation must satisfy.
    *
    * Architecture: docs/architecture/<feature-name>.md
    * Module: [ModuleName]
    */
   class <ModuleName>ContractSpec {

       // TODO: Replace with real implementation when available
       private val sut: <InterfaceName> = TODO("Provide implementation")

       @Test
       fun `method returns expected result for valid input`() {
           // Arrange
           // Act
           // Assert
       }

       @Test
       fun `method throws for invalid input`() {
           assertFailsWith<IllegalArgumentException> {
               // Act with invalid input
           }
       }
   }
   ```

6. **Generate Behavior Specifications**
   For each set of acceptance criteria, create `docs/specifications/<feature-name>/behaviors/<Feature>BehaviorSpec.kt`:

   ```kotlin
   package specifications.<feature>.behaviors

   import kotlin.test.Test
   import kotlin.test.assertEquals
   import kotlin.test.assertTrue

   /**
    * Behavior specification for [Feature].
    *
    * Derived from acceptance criteria in requirements.
    *
    * Requirements: docs/requirements/<feature-name>.md
    * Traces: FR-1, FR-2, ...
    */
   class <Feature>BehaviorSpec {

       @Test
       fun `given <precondition> when <action> then <expected outcome>`() {
           // Given
           // When
           // Then
       }
   }
   ```

7. **Generate Property-Based Specifications**
   For identified invariants, create `docs/specifications/<feature-name>/properties/<ModuleName>PropertySpec.kt`:

   ```kotlin
   package specifications.<feature>.properties

   import kotlin.test.Test
   import kotlin.test.assertEquals
   import kotlin.random.Random

   /**
    * Property-based specification for [ModuleName].
    *
    * Verifies invariants that must hold for all inputs.
    *
    * Architecture: docs/architecture/<feature-name>.md
    * Module: [ModuleName]
    */
   class <ModuleName>PropertySpec {

       private val random = Random(seed = 42) // deterministic for reproducibility

       @Test
       fun `roundtrip property - save then find returns equivalent`() {
           repeat(100) {
               // Generate random input
               // Perform operation
               // Assert invariant holds
           }
       }
   }
   ```

8. **Include Generator Functions**
   When property specs need random data, include lightweight generators:

   ```kotlin
   // Generators using kotlin.random (no external dependencies)
   private fun Random.alphaString(length: IntRange = 1..50): String =
       (1..nextInt(length.first, length.last + 1))
           .map { ('a'..'z').random(this) }
           .joinToString("")

   private fun Random.email(): String =
       "${alphaString(3..10)}@${alphaString(3..8)}.${alphaString(2..3)}"
   ```

9. **Add Traceability Comments**
   Every specification file must include:
   - Reference to the source architecture document and module
   - Reference to the source requirements document and requirement IDs
   - Clear mapping between test names and acceptance criteria

10. **Create Specification Index**
    Generate `docs/specifications/<feature-name>/README.md`:

    ```markdown
    # [Feature] Specifications

    ## Contract Specifications
    | File | Module | Methods Covered |
    |------|--------|----------------|

    ## Behavior Specifications
    | File | Requirements Traced | Acceptance Criteria |
    |------|--------------------|--------------------|

    ## Property Specifications
    | File | Module | Properties Verified |
    |------|--------|-------------------|

    ## Traceability Matrix
    | Requirement | Contract Spec | Behavior Spec | Property Spec |
    |-------------|--------------|---------------|---------------|
    ```

11. **Review with User**
    Present summary of generated specifications and ask if adjustments are needed.
    Make revisions as requested.

**Tips:**
- Start with contract specs - they define the minimum interface obligations
- Behavior specs should use language directly from acceptance criteria
- Not every module needs property specs - focus on stateful operations and data transformations
- Use `TODO("Provide implementation")` for implementations that don't exist yet
- Keep generators simple using `kotlin.random` extension functions

Use the executable-specifications skill for guidance on specification patterns, kotlin-test idioms, and property testing techniques.
