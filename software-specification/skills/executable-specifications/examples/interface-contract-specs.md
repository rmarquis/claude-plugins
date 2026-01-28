# Interface Contract Specification Example

This example shows contract specifications for a `UserRepository` interface, derived from an architecture document.

## Source Architecture

From `docs/architecture/user-management.md`:

```markdown
### Module: UserRepository
- **Responsibility**: Persist and retrieve user entities
- **Interface**: CRUD operations for User entities
- **Hidden Complexity**: Storage mechanism, ID generation, caching
- **Depth Score**: Deep
```

## Contract Specification

```kotlin
package specifications.usermanagement.contracts

import kotlin.test.Test
import kotlin.test.assertEquals
import kotlin.test.assertNotNull
import kotlin.test.assertNull
import kotlin.test.assertTrue
import kotlin.test.assertFailsWith

/**
 * Contract specification for UserRepository interface.
 *
 * Verifies the formal contract that any UserRepository implementation must satisfy.
 * These contracts define WHAT the repository must do, not HOW it does it.
 *
 * Architecture: docs/architecture/user-management.md
 * Module: UserRepository
 */
class UserRepositoryContractSpec {

    // TODO: Replace with real implementation when available
    private val repository: UserRepository = TODO("Provide UserRepository implementation")

    // --- save ---

    @Test
    fun `save returns the saved entity with generated id`() {
        // Arrange
        val user = User(
            id = null, // no ID yet
            name = "John Doe",
            email = "john@example.com"
        )

        // Act
        val saved = repository.save(user)

        // Assert
        assertNotNull(saved.id, "Saved entity must have a generated ID")
        assertEquals(user.name, saved.name)
        assertEquals(user.email, saved.email)
    }

    @Test
    fun `save with existing id updates the entity`() {
        // Arrange
        val original = repository.save(User(null, "Original", "original@example.com"))
        val modified = original.copy(name = "Modified")

        // Act
        val updated = repository.save(modified)

        // Assert
        assertEquals(original.id, updated.id, "ID should not change on update")
        assertEquals("Modified", updated.name)
    }

    @Test
    fun `save throws for invalid user`() {
        // Arrange
        val invalidUser = User(null, "", "not-an-email") // blank name, invalid email

        // Act & Assert
        assertFailsWith<IllegalArgumentException> {
            repository.save(invalidUser)
        }
    }

    @Test
    fun `save throws for duplicate email`() {
        // Arrange
        repository.save(User(null, "First User", "duplicate@example.com"))
        val duplicate = User(null, "Second User", "duplicate@example.com")

        // Act & Assert
        assertFailsWith<DuplicateEmailException> {
            repository.save(duplicate)
        }
    }

    // --- findById ---

    @Test
    fun `findById returns the entity for existing id`() {
        // Arrange
        val saved = repository.save(User(null, "Test User", "test@example.com"))

        // Act
        val found = repository.findById(saved.id!!)

        // Assert
        assertNotNull(found)
        assertEquals(saved, found)
    }

    @Test
    fun `findById returns null for non-existent id`() {
        // Arrange
        val nonExistentId = UserId(999999L)

        // Act
        val result = repository.findById(nonExistentId)

        // Assert
        assertNull(result, "Should return null for non-existent ID")
    }

    // --- findByEmail ---

    @Test
    fun `findByEmail returns the entity for existing email`() {
        // Arrange
        val saved = repository.save(User(null, "Email Test", "findme@example.com"))

        // Act
        val found = repository.findByEmail("findme@example.com")

        // Assert
        assertNotNull(found)
        assertEquals(saved, found)
    }

    @Test
    fun `findByEmail returns null for non-existent email`() {
        // Act
        val result = repository.findByEmail("nonexistent@example.com")

        // Assert
        assertNull(result)
    }

    @Test
    fun `findByEmail is case-insensitive`() {
        // Arrange
        val saved = repository.save(User(null, "Case Test", "CaseTest@Example.com"))

        // Act
        val found = repository.findByEmail("casetest@example.com")

        // Assert
        assertNotNull(found)
        assertEquals(saved.id, found.id)
    }

    // --- findAll ---

    @Test
    fun `findAll returns empty list when repository is empty`() {
        // Act
        val result = repository.findAll()

        // Assert
        assertTrue(result.isEmpty())
    }

    @Test
    fun `findAll returns all saved entities`() {
        // Arrange
        val user1 = repository.save(User(null, "User 1", "user1@example.com"))
        val user2 = repository.save(User(null, "User 2", "user2@example.com"))
        val user3 = repository.save(User(null, "User 3", "user3@example.com"))

        // Act
        val result = repository.findAll()

        // Assert
        assertEquals(3, result.size)
        assertTrue(result.contains(user1))
        assertTrue(result.contains(user2))
        assertTrue(result.contains(user3))
    }

    // --- delete ---

    @Test
    fun `delete removes the entity`() {
        // Arrange
        val saved = repository.save(User(null, "To Delete", "delete@example.com"))

        // Act
        repository.delete(saved.id!!)

        // Assert
        assertNull(repository.findById(saved.id!!))
    }

    @Test
    fun `delete is idempotent`() {
        // Arrange
        val saved = repository.save(User(null, "Idempotent Delete", "idemp@example.com"))
        val id = saved.id!!

        // Act - delete twice
        repository.delete(id)
        repository.delete(id) // should not throw

        // Assert
        assertNull(repository.findById(id))
    }

    @Test
    fun `delete for non-existent id does not throw`() {
        // Arrange
        val nonExistentId = UserId(999999L)

        // Act & Assert - should not throw
        repository.delete(nonExistentId)
    }

    // --- count ---

    @Test
    fun `count returns zero for empty repository`() {
        // Act
        val count = repository.count()

        // Assert
        assertEquals(0, count)
    }

    @Test
    fun `count returns correct number after saves and deletes`() {
        // Arrange
        val user1 = repository.save(User(null, "Count 1", "count1@example.com"))
        repository.save(User(null, "Count 2", "count2@example.com"))
        repository.save(User(null, "Count 3", "count3@example.com"))
        repository.delete(user1.id!!)

        // Act
        val count = repository.count()

        // Assert
        assertEquals(2, count)
    }

    // --- exists ---

    @Test
    fun `exists returns true for existing id`() {
        // Arrange
        val saved = repository.save(User(null, "Exists Test", "exists@example.com"))

        // Act
        val exists = repository.exists(saved.id!!)

        // Assert
        assertTrue(exists)
    }

    @Test
    fun `exists returns false for non-existent id`() {
        // Arrange
        val nonExistentId = UserId(999999L)

        // Act
        val exists = repository.exists(nonExistentId)

        // Assert
        assertEquals(false, exists)
    }
}
```

## Key Contract Patterns Demonstrated

### 1. Save Contract
- Returns entity with generated ID
- Updates existing entity (same ID)
- Throws for invalid input
- Throws for business rule violations (duplicate email)

### 2. Find Contract
- Returns entity for existing key
- Returns null for non-existent key
- Handles case-insensitivity where specified

### 3. Delete Contract
- Removes the entity
- Is idempotent (can be called multiple times)
- Does not throw for non-existent entities

### 4. Query Contract
- Returns empty collection when no data
- Returns all matching entities
- Count matches actual entity count

## Notes

- Each method's contract is documented in its own test group
- Tests use descriptive names that read as specification statements
- No implementation details leak into the tests
- `TODO("Provide implementation")` placeholder allows specs to compile but fail meaningfully
- Error conditions are explicitly specified with expected exception types
