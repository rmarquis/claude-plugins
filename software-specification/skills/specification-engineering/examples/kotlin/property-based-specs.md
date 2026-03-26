# Property-Based Specification Example

This example shows property-based specifications for a `UserRepository`, using `kotlin.random` for lightweight generators without external dependencies.

## Source Architecture

From `docs/architecture/user-management.md`:

```markdown
### Module: UserRepository
- **Responsibility**: Persist and retrieve user entities
- **Interface**: CRUD operations for User entities
- **Hidden Complexity**: Storage mechanism, ID generation, caching
- **Depth Score**: Deep

**Invariants**:
- Entity saved then retrieved is equivalent to saved entity
- Count is always non-negative
- Delete is idempotent
- FindAll size equals count
```

## Property Specification

```kotlin
package specifications.usermanagement.properties

import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertTrue
import kotlin.test.assertNull
import kotlin.random.Random

/**
 * Property-based specification for UserRepository.
 *
 * Verifies invariants that must hold for ALL valid inputs, not just specific examples.
 * Uses deterministic random generation for reproducible tests.
 *
 * Architecture: docs/architecture/user-management.md
 * Requirements: docs/requirements/user-management.md
 * Module: UserRepository
 */
class UserRepositoryPropertySpec {

    // Deterministic seed for reproducibility
    private val random = Random(seed = 42)

    // TODO: Replace with real implementation when available
    private fun createRepository(): UserRepository = TODO("Provide UserRepository implementation")

    // =========================================================================
    // Generators
    // =========================================================================

    /**
     * Generate a random alphabetic string.
     */
    private fun Random.alphaString(length: IntRange = 1..50): String =
        (1..nextInt(length.first, length.last + 1))
            .map { ('a'..'z').random(this) }
            .joinToString("")

    /**
     * Generate a random valid email address.
     */
    private fun Random.email(): String {
        val local = alphaString(3..15)
        val domain = alphaString(3..10)
        val tld = listOf("com", "org", "net", "io").random(this)
        return "$local@$domain.$tld"
    }

    /**
     * Generate a random positive long ID.
     */
    private fun Random.positiveLong(max: Long = Long.MAX_VALUE - 1): Long =
        nextLong(1, max)

    /**
     * Generate a random valid User (without ID, for saving).
     */
    private fun Random.newUser(): User = User(
        id = null,
        name = alphaString(1..50),
        email = email()
    )

    /**
     * Generate a list of unique users (unique by email).
     */
    private fun Random.uniqueUsers(count: Int): List<User> {
        val emails = mutableSetOf<String>()
        return (1..count).map {
            var user: User
            do {
                user = newUser()
            } while (user.email in emails)
            emails.add(user.email)
            user
        }
    }

    // =========================================================================
    // Roundtrip Properties
    // =========================================================================

    @Test
    fun `roundtrip - save then findById returns equivalent entity`() {
        val repository = createRepository()

        repeat(100) {
            val user = random.newUser()
            val saved = repository.save(user)
            val found = repository.findById(saved.id!!)

            assertEquals(saved, found,
                "Roundtrip failed for user: $user")
        }
    }

    @Test
    fun `roundtrip - save then findByEmail returns equivalent entity`() {
        repeat(100) {
            val repository = createRepository() // fresh for each iteration
            val user = random.newUser()
            val saved = repository.save(user)
            val found = repository.findByEmail(saved.email)

            assertEquals(saved, found,
                "FindByEmail roundtrip failed for: ${saved.email}")
        }
    }

    @Test
    fun `roundtrip - update preserves identity`() {
        val repository = createRepository()

        repeat(100) {
            val original = repository.save(random.newUser())
            val modified = original.copy(name = random.alphaString())
            val updated = repository.save(modified)
            val found = repository.findById(original.id!!)

            assertEquals(original.id, updated.id,
                "ID changed after update")
            assertEquals(updated, found,
                "Updated entity not retrievable")
        }
    }

    // =========================================================================
    // Idempotence Properties
    // =========================================================================

    @Test
    fun `idempotence - delete called twice has same effect as once`() {
        val repository = createRepository()

        repeat(100) {
            val saved = repository.save(random.newUser())
            val id = saved.id!!

            // First delete
            repository.delete(id)
            val countAfterFirst = repository.count()

            // Second delete - should not throw or change state
            repository.delete(id)
            val countAfterSecond = repository.count()

            assertNull(repository.findById(id),
                "Entity should not exist after delete")
            assertEquals(countAfterFirst, countAfterSecond,
                "Count changed on second delete")
        }
    }

    @Test
    fun `idempotence - delete non-existent id has no effect`() {
        val repository = createRepository()

        repeat(100) {
            // Save some entities
            repeat(random.nextInt(1, 5)) {
                repository.save(random.newUser())
            }
            val countBefore = repository.count()

            // Delete non-existent
            val nonExistentId = UserId(random.positiveLong())
            repository.delete(nonExistentId)

            assertEquals(countBefore, repository.count(),
                "Count changed when deleting non-existent ID")
        }
    }

    // =========================================================================
    // Invariant Properties
    // =========================================================================

    @Test
    fun `invariant - count is never negative`() {
        val repository = createRepository()

        repeat(100) {
            // Random operations
            when (random.nextInt(4)) {
                0 -> repository.save(random.newUser())
                1 -> repository.findAll().randomOrNull(random)?.let {
                    repository.delete(it.id!!)
                }
                2 -> repository.findById(UserId(random.positiveLong()))
                3 -> repository.count()
            }

            assertTrue(repository.count() >= 0,
                "Count became negative")
        }
    }

    @Test
    fun `invariant - findAll size equals count`() {
        val repository = createRepository()

        repeat(100) {
            // Random operations
            repeat(random.nextInt(1, 10)) {
                when (random.nextInt(3)) {
                    0 -> repository.save(random.newUser())
                    1 -> repository.findAll().randomOrNull(random)?.let {
                        repository.delete(it.id!!)
                    }
                    2 -> { /* no-op */ }
                }
            }

            val count = repository.count()
            val findAllSize = repository.findAll().size

            assertEquals(count, findAllSize,
                "count() = $count but findAll().size = $findAllSize")
        }
    }

    @Test
    fun `invariant - all IDs in findAll are unique`() {
        val repository = createRepository()

        repeat(50) {
            // Save multiple users
            repeat(random.nextInt(5, 20)) {
                repository.save(random.newUser())
            }

            val all = repository.findAll()
            val ids = all.map { it.id }
            val uniqueIds = ids.toSet()

            assertEquals(ids.size, uniqueIds.size,
                "Duplicate IDs found in findAll()")
        }
    }

    @Test
    fun `invariant - saved entity always retrievable before delete`() {
        val repository = createRepository()

        repeat(100) {
            val saved = repository.save(random.newUser())

            // Should be retrievable immediately
            val found = repository.findById(saved.id!!)
            assertEquals(saved, found,
                "Saved entity not immediately retrievable")

            // Should be in findAll
            assertTrue(repository.findAll().contains(saved),
                "Saved entity not in findAll()")

            // Should be findable by email
            assertEquals(saved, repository.findByEmail(saved.email),
                "Saved entity not findable by email")
        }
    }

    // =========================================================================
    // State Transition Properties
    // =========================================================================

    @Test
    fun `state - repository handles interleaved operations correctly`() {
        val repository = createRepository()
        val savedIds = mutableListOf<UserId>()

        repeat(200) {
            when (random.nextInt(5)) {
                0, 1 -> {
                    // Save (more likely)
                    val saved = repository.save(random.newUser())
                    savedIds.add(saved.id!!)
                }
                2 -> {
                    // Delete random existing
                    if (savedIds.isNotEmpty()) {
                        val idToDelete = savedIds.random(random)
                        repository.delete(idToDelete)
                        savedIds.remove(idToDelete)
                    }
                }
                3 -> {
                    // Find existing
                    if (savedIds.isNotEmpty()) {
                        val idToFind = savedIds.random(random)
                        val found = repository.findById(idToFind)
                        if (found == null) {
                            // Was deleted, remove from tracking
                            savedIds.remove(idToFind)
                        }
                    }
                }
                4 -> {
                    // Verify count matches tracking
                    // (accounting for possible duplicates from random selection)
                    assertTrue(repository.count() <= savedIds.size,
                        "More entities than tracked")
                }
            }
        }

        // Final invariant checks
        assertTrue(repository.count() >= 0)
        assertEquals(repository.count(), repository.findAll().size)
    }

    // =========================================================================
    // Bulk Operation Properties
    // =========================================================================

    @Test
    fun `bulk - saving multiple users maintains all invariants`() {
        repeat(20) {
            val repository = createRepository()
            val users = random.uniqueUsers(random.nextInt(10, 50))

            val savedUsers = users.map { repository.save(it) }

            // All saved
            assertEquals(users.size, repository.count())

            // All retrievable
            savedUsers.forEach { saved ->
                assertEquals(saved, repository.findById(saved.id!!))
            }

            // All unique IDs
            val ids = savedUsers.map { it.id }.toSet()
            assertEquals(savedUsers.size, ids.size)
        }
    }
}

// Helper extension for random element from list
private fun <T> List<T>.randomOrNull(random: Random): T? =
    if (isEmpty()) null else this[random.nextInt(size)]
```

## Key Property Patterns Demonstrated

### 1. Lightweight Generators
Extension functions on `Random` produce random domain objects:
- `alphaString()` - random strings
- `email()` - valid email format
- `newUser()` - complete domain object
- `uniqueUsers()` - list with constraints

### 2. Deterministic Reproducibility
```kotlin
private val random = Random(seed = 42)
```
Same seed = same sequence of random values = reproducible test failures.

### 3. Roundtrip Properties
Verify that data survives persistence operations:
```kotlin
val saved = repository.save(user)
val found = repository.findById(saved.id!!)
assertEquals(saved, found)
```

### 4. Idempotence Properties
Verify that repeated operations don't change state:
```kotlin
repository.delete(id)
repository.delete(id) // second delete should not throw or change state
```

### 5. Invariant Properties
Verify conditions that must always hold:
- Count is never negative
- findAll().size equals count()
- All IDs are unique
- Saved entities are retrievable

### 6. Stateful Property Testing
Test that invariants hold across random operation sequences:
```kotlin
repeat(200) {
    when (random.nextInt(5)) {
        0 -> // save
        1 -> // delete
        2 -> // find
        // ...
    }
    // invariants must hold after each operation
}
```

## Notes

- Uses only `kotlin.random` - no external property testing libraries
- All tests use `repeat(N)` for controlled iteration count
- Failure messages include the input that failed for debugging
- Fresh repository instances where needed to ensure test isolation
- Generator functions are private to the spec class
