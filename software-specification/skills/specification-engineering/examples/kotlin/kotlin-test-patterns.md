# kotlin-test Patterns

Idiomatic patterns for writing executable specifications using `kotlin-test`.

## Core Assertions

### Equality Assertions

```kotlin
import kotlin.test.assertEquals
import kotlin.test.assertNotEquals

// Basic equality
assertEquals(expected, actual)
assertEquals(expected, actual, "Custom failure message")

// Inequality
assertNotEquals(unexpected, actual)

// Floating point with tolerance
assertEquals(3.14, result, 0.01) // passes if within tolerance
```

### Nullability Assertions

```kotlin
import kotlin.test.assertNotNull
import kotlin.test.assertNull

// Assert not null and use the value
val result = assertNotNull(repository.findById(id))
assertEquals("expected", result.name) // result is smart-cast to non-null

// Assert null
assertNull(repository.findById(nonExistentId))
```

### Boolean Assertions

```kotlin
import kotlin.test.assertTrue
import kotlin.test.assertFalse

assertTrue(user.isActive)
assertTrue(user.isActive, "User should be active after registration")

assertFalse(user.isDeleted)
assertFalse(user.isDeleted, "User should not be deleted initially")
```

### Exception Assertions

```kotlin
import kotlin.test.assertFailsWith

// Assert specific exception type
assertFailsWith<IllegalArgumentException> {
    service.process(invalidInput)
}

// Capture exception for further assertions
val exception = assertFailsWith<ValidationException> {
    service.validate(badData)
}
assertEquals("INVALID_EMAIL", exception.code)
assertTrue(exception.message!!.contains("email"))
```

### Collection Assertions

```kotlin
import kotlin.test.assertTrue
import kotlin.test.assertEquals
import kotlin.test.assertContentEquals

// Size checks
assertEquals(3, results.size)
assertTrue(results.isEmpty())
assertTrue(results.isNotEmpty())

// Contains checks
assertTrue(results.contains(expectedItem))
assertTrue(results.containsAll(listOf(item1, item2)))

// Content equality (order matters)
assertContentEquals(listOf(1, 2, 3), results)

// Content equality (order doesn't matter)
assertEquals(setOf(1, 2, 3), results.toSet())
```

### Type Assertions

```kotlin
import kotlin.test.assertIs
import kotlin.test.assertIsNot

// Assert type and smart-cast
val result = assertIs<SuccessResult>(operation.execute())
assertEquals(42, result.value) // result is smart-cast to SuccessResult

// Assert not a type
assertIsNot<ErrorResult>(operation.execute())
```

## Test Organization Patterns

### Single Class, Multiple Methods

Simple approach for small specification sets:

```kotlin
class UserRepositoryContractSpec {

    @Test
    fun `save returns entity with generated id`() { /* ... */ }

    @Test
    fun `findById returns null for non-existent id`() { /* ... */ }

    @Test
    fun `delete is idempotent`() { /* ... */ }
}
```

### Grouped with Comments

Use comments to visually group related tests:

```kotlin
class PaymentServiceContractSpec {

    // --- processPayment ---

    @Test
    fun `processPayment succeeds for valid card`() { /* ... */ }

    @Test
    fun `processPayment fails for expired card`() { /* ... */ }

    @Test
    fun `processPayment fails for insufficient funds`() { /* ... */ }

    // --- refund ---

    @Test
    fun `refund succeeds for processed payment`() { /* ... */ }

    @Test
    fun `refund fails for already refunded payment`() { /* ... */ }
}
```

### Nested Inner Classes

Use nested classes for complex specification hierarchies:

```kotlin
class OrderSpec {

    inner class `when order is pending` {

        @Test
        fun `can add items`() { /* ... */ }

        @Test
        fun `can remove items`() { /* ... */ }

        @Test
        fun `can submit for processing`() { /* ... */ }
    }

    inner class `when order is submitted` {

        @Test
        fun `cannot add items`() { /* ... */ }

        @Test
        fun `can cancel`() { /* ... */ }

        @Test
        fun `can be shipped`() { /* ... */ }
    }

    inner class `when order is shipped` {

        @Test
        fun `cannot cancel`() { /* ... */ }

        @Test
        fun `can be delivered`() { /* ... */ }
    }
}
```

## Test Data Patterns

### Factory Methods

Create reusable test data factories:

```kotlin
class UserServiceSpec {

    // Default valid objects
    private fun validUser(
        name: String = "John Doe",
        email: String = "john@example.com",
        role: Role = Role.USER
    ) = User(name = name, email = email, role = role)

    private fun validCredentials(
        email: String = "john@example.com",
        password: String = "SecurePass123!"
    ) = Credentials(email = email, password = password)

    // Invalid object variants
    private fun userWithInvalidEmail() = validUser(email = "not-an-email")

    private fun userWithBlankName() = validUser(name = "")

    @Test
    fun `accepts valid user`() {
        val user = validUser()
        val result = service.create(user)
        assertTrue(result.isSuccess)
    }

    @Test
    fun `rejects user with invalid email`() {
        val user = userWithInvalidEmail()
        assertFailsWith<ValidationException> {
            service.create(user)
        }
    }
}
```

### Builder Pattern for Complex Objects

```kotlin
class OrderSpec {

    private class OrderBuilder {
        var customerId: Long = 1L
        var items: MutableList<OrderItem> = mutableListOf()
        var status: OrderStatus = OrderStatus.PENDING
        var shippingAddress: Address = defaultAddress()

        fun withItem(productId: Long, quantity: Int) = apply {
            items.add(OrderItem(productId, quantity))
        }

        fun withStatus(status: OrderStatus) = apply {
            this.status = status
        }

        fun build() = Order(
            customerId = customerId,
            items = items.toList(),
            status = status,
            shippingAddress = shippingAddress
        )

        private fun defaultAddress() = Address(
            street = "123 Test St",
            city = "Test City",
            zip = "12345"
        )
    }

    private fun order() = OrderBuilder()

    @Test
    fun `calculates total for order with items`() {
        val order = order()
            .withItem(productId = 1, quantity = 2)
            .withItem(productId = 2, quantity = 1)
            .build()

        assertEquals(expectedTotal, order.calculateTotal())
    }
}
```

### Random Generators for Property Tests

Extension functions on `Random` for generating test data:

```kotlin
import kotlin.random.Random

class UserPropertySpec {

    private val random = Random(seed = 42)

    // String generators
    private fun Random.alphaString(length: IntRange = 1..50): String =
        (1..nextInt(length.first, length.last + 1))
            .map { ('a'..'z').random(this) }
            .joinToString("")

    private fun Random.alphaNumericString(length: IntRange = 1..50): String =
        (1..nextInt(length.first, length.last + 1))
            .map { ('a'..'z') + ('A'..'Z') + ('0'..'9') }
            .flatten()
            .random(this)
            .let { (1..nextInt(length.first, length.last + 1)).map { _ -> it }.joinToString("") }

    private fun Random.email(): String =
        "${alphaString(3..10)}@${alphaString(3..8)}.${alphaString(2..3)}"

    // Number generators
    private fun Random.positiveInt(max: Int = 1000): Int =
        nextInt(1, max)

    private fun Random.positiveLong(max: Long = 1000L): Long =
        nextLong(1, max)

    private fun Random.money(max: Double = 10000.0): Double =
        (nextDouble() * max * 100).toLong() / 100.0 // Two decimal places

    // Domain object generators
    private fun Random.user(): User = User(
        id = UserId(positiveLong()),
        name = alphaString(1..50),
        email = email()
    )

    private fun Random.address(): Address = Address(
        street = "${positiveInt(999)} ${alphaString(5..20)} St",
        city = alphaString(5..15),
        zip = positiveInt(10000..99999).toString()
    )

    @Test
    fun `roundtrip - save then find returns equivalent user`() {
        repeat(100) {
            val user = random.user()
            // ... test logic
        }
    }
}
```

## Assertion Extension Functions

Create custom assertions for domain-specific verification:

```kotlin
// Custom assertion extensions
fun User.shouldBeActive() {
    assertTrue(this.isActive, "Expected user ${this.id} to be active")
}

fun User.shouldHaveRole(expected: Role) {
    assertEquals(expected, this.role, "Expected user ${this.id} to have role $expected")
}

fun <T> Result<T>.shouldBeSuccess(): T {
    assertTrue(this.isSuccess, "Expected success but was failure: ${this.exceptionOrNull()}")
    return this.getOrThrow()
}

fun <T> Result<T>.shouldBeFailure(): Throwable {
    assertTrue(this.isFailure, "Expected failure but was success: ${this.getOrNull()}")
    return this.exceptionOrNull()!!
}

// Usage in tests
@Test
fun `registered user is active with USER role`() {
    val result = service.register(validCredentials())
    val user = result.shouldBeSuccess()
    user.shouldBeActive()
    user.shouldHaveRole(Role.USER)
}
```

## Setup and Teardown Patterns

### Fresh Instance Per Test

kotlin-test creates a new test class instance for each test method:

```kotlin
class RepositorySpec {

    // Fresh instance for every test
    private val repository = InMemoryRepository()

    @Test
    fun `starts empty`() {
        assertEquals(0, repository.count())
    }

    @Test
    fun `count increases after save`() {
        repository.save(validEntity())
        assertEquals(1, repository.count())
    }
}
```

### Shared Expensive Setup

For expensive setup, use companion objects or lazy initialization:

```kotlin
class IntegrationSpec {

    companion object {
        // Shared across all tests in the class
        private val testDatabase by lazy { createTestDatabase() }
    }

    private val repository = DatabaseRepository(testDatabase)

    @Test
    fun `can save and retrieve`() {
        // Uses shared database
    }
}
```

### BeforeTest and AfterTest

```kotlin
import kotlin.test.BeforeTest
import kotlin.test.AfterTest

class StatefulServiceSpec {

    private lateinit var service: StatefulService

    @BeforeTest
    fun setup() {
        service = StatefulService()
        service.initialize()
    }

    @AfterTest
    fun cleanup() {
        service.shutdown()
    }

    @Test
    fun `test with initialized service`() {
        // service is initialized
    }
}
```

## Parameterized Test Pattern

Simulate parameterized tests with loops:

```kotlin
class ValidationSpec {

    @Test
    fun `rejects all invalid email formats`() {
        val invalidEmails = listOf(
            "no-at-sign",
            "@no-local-part.com",
            "no-domain@",
            "spaces in@email.com",
            "multiple@@signs.com"
        )

        invalidEmails.forEach { email ->
            assertFailsWith<ValidationException>("Expected rejection for: $email") {
                validator.validateEmail(email)
            }
        }
    }

    @Test
    fun `accepts all valid email formats`() {
        val validEmails = listOf(
            "simple@example.com",
            "user.name@example.com",
            "user+tag@example.com",
            "user@subdomain.example.com"
        )

        validEmails.forEach { email ->
            // Should not throw
            validator.validateEmail(email)
        }
    }
}
```

## Best Practices Summary

1. **Use backtick test names** - they read as natural language specifications
2. **One assertion concept per test** - multiple assertions are fine if testing one logical concept
3. **Use `assertNotNull` for smart-casting** - captures and returns the non-null value
4. **Use `assertFailsWith<T>`** - cleaner than try-catch with `fail()`
5. **Prefer factory methods over inline object creation** - makes tests more readable
6. **Use extension functions for custom assertions** - creates domain-specific testing DSL
7. **Use deterministic seeds in property tests** - enables reproducible failures
8. **Group related tests** - comments for simple cases, nested classes for complex hierarchies
