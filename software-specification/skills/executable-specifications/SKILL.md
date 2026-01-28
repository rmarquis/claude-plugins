---
name: Executable Specifications
description: This skill should be used when the user asks to "write specifications", "create test specs", "specify interfaces", "contract testing", "behavior specifications", "property-based testing", "BDD specs", "given when then", "specification-first", "contract-first", "executable contracts", "kotlin-test specifications", "test before implementation", "formal contracts", "interface contracts", or discusses writing executable tests that define behavior before implementation. Provides patterns for contract, behavior, and property-based specifications using kotlin-test.
version: 0.1.0
---

# Executable Specifications

Write executable test specifications in Kotlin using `kotlin-test` that serve as formal contracts, behavioral documentation, and property invariants. Specifications are written BEFORE implementation and define what the system must do.

## Core Philosophy

**Specification-first development** means:
1. Architecture defines module structure and interfaces
2. Requirements define acceptance criteria and constraints
3. **Specifications formalize both into executable tests**
4. Implementation satisfies the specifications

Specifications bridge the gap between design documents and working code. They are the source of truth for what an implementation must do.

## Deriving Specifications from Architecture + Requirements

### Mapping Architecture to Specifications

| Architecture Artifact | Specification Type |
|----------------------|-------------------|
| Module interface | Contract specification class |
| Interface method | Contract test methods |
| Module responsibility | Behavior specification class |
| Module interaction | Integration contract spec |
| Design constraint | Property specification |

### Mapping Requirements to Specifications

| Requirement Artifact | Specification Type |
|---------------------|-------------------|
| Functional requirement | Behavior spec test(s) |
| Acceptance criterion | Individual behavior test |
| Non-functional requirement | Property or contract test |
| Error handling rule | Contract error test |
| Business invariant | Property-based test |

### Derivation Process

1. **Read architecture document** - extract modules, interfaces, responsibilities
2. **Read requirements document** - extract FRs, NFRs, acceptance criteria
3. **For each module interface**: create a contract spec class with tests for each method
4. **For each acceptance criterion**: create a behavior spec test with Given-When-Then
5. **For each invariant**: create a property spec test with random generation

## Three Specification Levels

### Level 1: Interface Contract Specifications

Contract specs define the obligations of each interface method.

**What to specify:**
- Pre-conditions: what must be true before calling
- Post-conditions: what must be true after calling
- Return guarantees: what the return value must satisfy
- Error conditions: when and what exceptions are thrown
- Idempotency: whether repeated calls produce the same result

**Structure:**

```kotlin
package specifications.<feature>.contracts

import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertNotNull
import kotlin.test.assertNull
import kotlin.test.assertFailsWith

/**
 * Contract specification for [InterfaceName].
 *
 * Architecture: docs/architecture/<feature>.md
 * Module: [ModuleName]
 */
class <Interface>ContractSpec {

    private val sut: <Interface> = TODO("Provide implementation")

    // --- Method: save ---

    @Test
    fun `save returns entity with generated id`() {
        val input = createValidEntity()
        val result = sut.save(input)
        assertNotNull(result.id)
    }

    @Test
    fun `save throws for invalid entity`() {
        val invalid = createInvalidEntity()
        assertFailsWith<IllegalArgumentException> {
            sut.save(invalid)
        }
    }

    // --- Method: findById ---

    @Test
    fun `findById returns null for non-existent id`() {
        val result = sut.findById(nonExistentId())
        assertNull(result)
    }

    // --- Method: delete ---

    @Test
    fun `delete is idempotent`() {
        val id = existingEntityId()
        sut.delete(id)
        sut.delete(id) // second call should not throw
    }
}
```

**Guidelines:**
- One spec class per interface
- Group tests by method using comments
- Test the happy path first, then error conditions
- Use helper methods for test data creation
- Name tests as statements of what must be true

### Level 2: Behavior Specifications (BDD-Style)

Behavior specs verify acceptance criteria using Given-When-Then structure.

**What to specify:**
- User-facing behaviors from acceptance criteria
- Workflow steps from functional requirements
- Error scenarios from edge case requirements

**Structure:**

```kotlin
package specifications.<feature>.behaviors

import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertTrue
import kotlin.test.assertFalse

/**
 * Behavior specification for [Feature].
 *
 * Requirements: docs/requirements/<feature>.md
 * Traces: FR-1, FR-2, FR-3
 */
class <Feature>BehaviorSpec {

    // FR-1: User Registration
    @Test
    fun `given valid credentials when user registers then account is created`() {
        // Given
        val credentials = validCredentials()

        // When
        val result = registrationService.register(credentials)

        // Then
        assertTrue(result.isSuccess)
        assertNotNull(result.accountId)
    }

    // FR-1: User Registration - duplicate check
    @Test
    fun `given existing email when user registers then registration fails with duplicate error`() {
        // Given
        val existing = existingUserCredentials()

        // When
        val result = registrationService.register(existing)

        // Then
        assertTrue(result.isFailure)
        assertEquals("EMAIL_DUPLICATE", result.errorCode)
    }
}
```

**Guidelines:**
- Test names read as natural language sentences
- Comment traces requirement ID above each test or group
- Given section sets up preconditions
- When section performs exactly one action
- Then section verifies postconditions
- One spec class per feature or user story

### Level 3: Property-Based Specifications

Property specs verify invariants that must hold for all inputs using random generation.

**What to specify:**
- Roundtrip properties: encode then decode yields equivalent
- Idempotence: applying operation twice equals applying once
- Commutativity: order of operands doesn't matter (where applicable)
- Invariant preservation: state remains valid after any operation sequence

**Structure:**

```kotlin
package specifications.<feature>.properties

import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertTrue
import kotlin.random.Random

/**
 * Property-based specification for [ModuleName].
 *
 * Architecture: docs/architecture/<feature>.md
 * Requirements: docs/requirements/<feature>.md
 * Module: [ModuleName]
 */
class <Module>PropertySpec {

    private val random = Random(seed = 42) // deterministic

    // --- Generators ---

    private fun Random.alphaString(length: IntRange = 1..50): String =
        (1..nextInt(length.first, length.last + 1))
            .map { ('a'..'z').random(this) }
            .joinToString("")

    private fun Random.positiveInt(max: Int = Int.MAX_VALUE): Int =
        nextInt(1, max)

    // --- Properties ---

    @Test
    fun `roundtrip - save then findById returns equivalent entity`() {
        repeat(100) {
            val entity = random.validEntity()
            val saved = sut.save(entity)
            val found = sut.findById(saved.id)
            assertEquals(saved, found)
        }
    }

    @Test
    fun `idempotence - delete called twice has same effect as once`() {
        repeat(100) {
            val entity = random.validEntity()
            val saved = sut.save(entity)
            sut.delete(saved.id)
            sut.delete(saved.id) // should not throw
        }
    }

    @Test
    fun `invariant - entity count never negative after operations`() {
        repeat(100) {
            val count = sut.count()
            assertTrue(count >= 0, "Entity count must never be negative")
        }
    }
}
```

**Guidelines:**
- Use `Random(seed = 42)` for deterministic, reproducible tests
- Use `repeat(100)` for lightweight property iteration
- Write generators as extension functions on `Random`
- Only use `kotlin.random` - no external property testing libraries
- Name tests as property statements: "roundtrip - ...", "idempotence - ...", "invariant - ..."

## kotlin-test Patterns

### Assertion Patterns

```kotlin
// Equality
assertEquals(expected, actual)
assertNotEquals(unexpected, actual)

// Nullability
assertNotNull(value)
assertNull(value)

// Boolean
assertTrue(condition)
assertFalse(condition)

// Exceptions
assertFailsWith<IllegalArgumentException> {
    riskyOperation()
}

// Collections
assertTrue(list.contains(element))
assertEquals(expectedSize, list.size)
assertTrue(list.isEmpty())
```

### Test Organization with Nested Classes

```kotlin
class ModuleSpec {

    inner class `when valid input` {
        @Test fun `returns success result`() { /* ... */ }
        @Test fun `persists the entity`() { /* ... */ }
    }

    inner class `when invalid input` {
        @Test fun `throws IllegalArgumentException`() { /* ... */ }
        @Test fun `does not modify state`() { /* ... */ }
    }

    inner class `when entity not found` {
        @Test fun `returns null`() { /* ... */ }
    }
}
```

### Test Data Helpers

```kotlin
class UserSpec {

    // Factory methods for test data
    private fun validUser(
        name: String = "Test User",
        email: String = "test@example.com"
    ) = User(name = name, email = email)

    private fun invalidUser() = User(name = "", email = "not-an-email")

    // Generators for property tests
    private fun Random.user() = User(
        name = alphaString(1..50),
        email = email()
    )

    private fun Random.alphaString(length: IntRange = 1..50): String =
        (1..nextInt(length.first, length.last + 1))
            .map { ('a'..'z').random(this) }
            .joinToString("")

    private fun Random.email(): String =
        "${alphaString(3..10)}@${alphaString(3..8)}.${alphaString(2..3)}"
}
```

### Test Lifecycle

```kotlin
class StatefulModuleSpec {

    private lateinit var sut: Module

    // Setup before each test - recreate clean state
    // In kotlin-test, use init block or property initialization
    private val repository = InMemoryRepository()

    @Test
    fun `test with clean state`() {
        // repository is fresh for each test class instance
    }
}
```

## Specification Quality Checklist

### Contract Completeness
- [ ] Every public method has at least one contract test
- [ ] Happy path tested for each method
- [ ] Error conditions tested for each method
- [ ] Null/empty input behavior specified
- [ ] Return type guarantees verified
- [ ] Idempotency tested where relevant

### Behavior Coverage
- [ ] Every acceptance criterion has a corresponding test
- [ ] Given-When-Then structure consistently applied
- [ ] Test names readable as specification sentences
- [ ] Requirement IDs traced in comments
- [ ] Error scenarios from requirements covered

### Property Quality
- [ ] Roundtrip properties for all serialization/persistence operations
- [ ] Idempotence properties for all delete/cleanup operations
- [ ] State invariants verified after operation sequences
- [ ] Generators produce valid domain objects
- [ ] Deterministic seeds used for reproducibility

### Traceability
- [ ] Every spec file references its source architecture document
- [ ] Every behavior test traces to a requirement ID
- [ ] Module names match architecture document
- [ ] README.md index is complete and accurate

## Additional Resources

### Reference Files

For detailed guidance, consult:
- **`references/kotlin-test-patterns.md`** - Idiomatic kotlin-test usage and assertion patterns
- **`references/specification-checklist.md`** - Detailed completeness and quality checklist
- **`references/property-testing-guide.md`** - Property identification and lightweight generator patterns

### Example Files

Working examples in `examples/`:
- **`examples/interface-contract-specs.md`** - Contract spec example for a Repository interface
- **`examples/behavior-specs.md`** - BDD-style behavior spec example for user registration
- **`examples/property-based-specs.md`** - Property-based spec example with generators
