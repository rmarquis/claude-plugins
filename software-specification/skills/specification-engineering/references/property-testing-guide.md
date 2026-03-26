# Property-Based Testing Guide

A guide to identifying properties, writing generators, and implementing property-based specifications using only `kotlin.random` (no external dependencies).

## What is Property-Based Testing?

Property-based testing verifies that **invariants hold for all inputs**, not just specific examples. Instead of writing:

```kotlin
@Test fun `save then find returns the user`() {
    val user = User(name = "John", email = "john@example.com")
    val saved = repository.save(user)
    assertEquals(saved, repository.findById(saved.id))
}
```

Write:

```kotlin
@Test fun `roundtrip property - save then find returns equivalent user`() {
    repeat(100) {
        val user = random.user() // random valid user
        val saved = repository.save(user)
        assertEquals(saved, repository.findById(saved.id))
    }
}
```

This tests the **property** (roundtrip equivalence) with many random inputs, increasing confidence and discovering edge cases.

## Identifying Properties

### Roundtrip Properties

Operations that convert data and back should return equivalent values.

**Pattern**: `decode(encode(x)) == x` or `load(save(x)) == x`

**Where to look**:
- Serialization/deserialization (JSON, XML, binary)
- Repository save/load operations
- Encoding/decoding (Base64, URL encoding)
- Format conversions (string parsing, date formatting)
- Compression/decompression

**Examples**:
```kotlin
// Serialization roundtrip
decode(encode(entity)) == entity

// Repository roundtrip
repository.findById(repository.save(entity).id) == savedEntity

// String parsing roundtrip
parse(format(date)) == date
```

### Idempotence Properties

Operations that, when applied multiple times, have the same effect as applying once.

**Pattern**: `f(f(x)) == f(x)` or `f(x); f(x)` has same state as `f(x)`

**Where to look**:
- Delete operations
- Normalization functions (trim, lowercase, sort)
- Cache operations (ensure cached, invalidate)
- Set operations (add to set multiple times)
- Cleanup/reset operations

**Examples**:
```kotlin
// Delete is idempotent
repository.delete(id)
repository.delete(id) // should not throw, state unchanged

// Normalization is idempotent
normalize(normalize(text)) == normalize(text)

// Sort is idempotent
sort(sort(list)) == sort(list)
```

### Commutativity Properties

Operations where the order of arguments doesn't affect the result.

**Pattern**: `f(a, b) == f(b, a)`

**Where to look**:
- Set operations (union, intersection)
- Mathematical operations (addition, multiplication)
- Merge operations
- Comparison functions (min, max)

**Examples**:
```kotlin
// Set union is commutative
setA.union(setB) == setB.union(setA)

// Merge is commutative (if designed that way)
merge(configA, configB) == merge(configB, configA)
```

### Associativity Properties

Operations where grouping doesn't affect the result.

**Pattern**: `f(f(a, b), c) == f(a, f(b, c))`

**Where to look**:
- String concatenation
- List concatenation
- Reduction operations

**Examples**:
```kotlin
// Concatenation is associative
(a + b) + c == a + (b + c)
```

### Invariant Properties

Conditions that must always be true after any operation.

**Pattern**: `invariant(state)` is always true

**Where to look**:
- State validity (accounts have positive balance, counts are non-negative)
- Collection constraints (sorted order maintained, no duplicates in set)
- Business rules (order total equals sum of line items)
- Temporal ordering (created date before modified date)

**Examples**:
```kotlin
// Count is never negative
repository.count() >= 0

// Order total matches line items
order.total == order.items.sumOf { it.price * it.quantity }

// Sorted list remains sorted after add
sortedList.add(item)
sortedList == sortedList.sorted()
```

### Inverse Properties

Operations that undo each other.

**Pattern**: `g(f(x)) == x` where g is the inverse of f

**Where to look**:
- Encrypt/decrypt
- Compress/decompress
- Encode/decode
- Add/remove operations

**Examples**:
```kotlin
// Encrypt and decrypt are inverses
decrypt(encrypt(plaintext, key), key) == plaintext

// Add and remove are inverses (for unique items)
list.add(item)
list.remove(item)
list == originalList
```

## Writing Generators

### Basic Generators

Extension functions on `Random` that produce random values:

```kotlin
import kotlin.random.Random

// String generators
fun Random.alphaChar(): Char = ('a'..'z').random(this)

fun Random.alphaString(length: IntRange = 1..50): String =
    (1..nextInt(length.first, length.last + 1))
        .map { alphaChar() }
        .joinToString("")

fun Random.alphaNumericChar(): Char =
    (('a'..'z') + ('A'..'Z') + ('0'..'9')).random(this)

fun Random.alphaNumericString(length: IntRange = 1..50): String =
    (1..nextInt(length.first, length.last + 1))
        .map { alphaNumericChar() }
        .joinToString("")

// Number generators
fun Random.positiveInt(max: Int = Int.MAX_VALUE - 1): Int =
    nextInt(1, max + 1)

fun Random.nonNegativeInt(max: Int = Int.MAX_VALUE - 1): Int =
    nextInt(0, max + 1)

fun Random.positiveLong(max: Long = Long.MAX_VALUE - 1): Long =
    nextLong(1, max + 1)

fun Random.money(max: Double = 10000.0): Double =
    (nextDouble() * max * 100).toLong() / 100.0

// Boolean
fun Random.boolean(): Boolean = nextBoolean()

// Enum
inline fun <reified T : Enum<T>> Random.enum(): T =
    enumValues<T>().random(this)
```

### Domain-Specific Generators

Compose basic generators into domain objects:

```kotlin
// Email generator
fun Random.email(): String =
    "${alphaString(3..10)}@${alphaString(3..8)}.${alphaString(2..3)}"

// Phone generator
fun Random.phone(): String =
    "+1${(1..10).map { nextInt(0, 10) }.joinToString("")}"

// Date generator (within reasonable range)
fun Random.pastDate(): LocalDate =
    LocalDate.now().minusDays(nextLong(1, 3650))

fun Random.futureDate(): LocalDate =
    LocalDate.now().plusDays(nextLong(1, 3650))

// Domain objects
fun Random.userId(): UserId = UserId(positiveLong())

fun Random.user(): User = User(
    id = userId(),
    name = alphaString(1..50),
    email = email(),
    createdAt = pastDate()
)

fun Random.address(): Address = Address(
    street = "${positiveInt(1, 9999)} ${alphaString(5..20)} St",
    city = alphaString(5..15),
    state = alphaString(2..2).uppercase(),
    zip = positiveInt(10000, 99999).toString()
)

fun Random.order(): Order = Order(
    id = OrderId(positiveLong()),
    customerId = userId(),
    items = orderItems(1..5),
    status = enum<OrderStatus>()
)

fun Random.orderItem(): OrderItem = OrderItem(
    productId = ProductId(positiveLong()),
    quantity = positiveInt(1, 10),
    unitPrice = money(1.0, 999.99)
)

fun Random.orderItems(countRange: IntRange = 1..5): List<OrderItem> =
    (1..nextInt(countRange.first, countRange.last + 1))
        .map { orderItem() }
```

### Constrained Generators

Generate values that satisfy constraints:

```kotlin
// Valid email (stricter format)
fun Random.validEmail(): String {
    val local = alphaNumericString(3..20)
    val domain = alphaString(3..10)
    val tld = listOf("com", "org", "net", "io").random(this)
    return "$local@$domain.$tld"
}

// Age within valid range
fun Random.adultAge(): Int = nextInt(18, 120)
fun Random.childAge(): Int = nextInt(0, 18)

// Non-empty list
fun <T> Random.nonEmptyList(maxSize: Int = 10, generator: Random.() -> T): List<T> =
    (1..nextInt(1, maxSize + 1)).map { generator() }

// List with specific size
fun <T> Random.listOfSize(size: Int, generator: Random.() -> T): List<T> =
    (1..size).map { generator() }

// Unique list (no duplicates by key)
fun <T, K> Random.uniqueList(
    maxSize: Int = 10,
    keySelector: (T) -> K,
    generator: Random.() -> T
): List<T> {
    val seen = mutableSetOf<K>()
    val result = mutableListOf<T>()
    repeat(maxSize * 3) { // try enough times
        if (result.size >= maxSize) return result
        val item = generator()
        val key = keySelector(item)
        if (key !in seen) {
            seen.add(key)
            result.add(item)
        }
    }
    return result
}
```

### Edge Case Generators

Explicitly include edge cases:

```kotlin
// String with edge cases mixed in
fun Random.stringWithEdgeCases(): String = when (nextInt(10)) {
    0 -> "" // empty
    1 -> " " // single space
    2 -> "   " // multiple spaces
    3 -> alphaString(1..1) // single char
    4 -> alphaString(100..200) // long string
    else -> alphaString(1..50) // normal
}

// Number with edge cases
fun Random.intWithEdgeCases(): Int = when (nextInt(10)) {
    0 -> 0
    1 -> 1
    2 -> -1
    3 -> Int.MAX_VALUE
    4 -> Int.MIN_VALUE
    else -> nextInt()
}
```

## Property Test Structure

### Basic Pattern

```kotlin
class ModulePropertySpec {

    private val random = Random(seed = 42) // deterministic

    // Generators defined here

    @Test
    fun `property name - description of what must hold`() {
        repeat(100) {
            // Generate
            val input = random.generateInput()

            // Act
            val result = operation(input)

            // Assert invariant
            assertTrue(invariant(result), "Failed for input: $input")
        }
    }
}
```

### Roundtrip Example

```kotlin
@Test
fun `roundtrip - serialize then deserialize returns equivalent`() {
    repeat(100) {
        val original = random.user()
        val serialized = serializer.serialize(original)
        val deserialized = serializer.deserialize(serialized)
        assertEquals(original, deserialized, "Failed for: $original")
    }
}
```

### Idempotence Example

```kotlin
@Test
fun `idempotence - normalizing twice equals normalizing once`() {
    repeat(100) {
        val input = random.alphaString()
        val once = normalizer.normalize(input)
        val twice = normalizer.normalize(once)
        assertEquals(once, twice, "Not idempotent for: $input")
    }
}
```

### Invariant Example

```kotlin
@Test
fun `invariant - order total always equals sum of line items`() {
    repeat(100) {
        val order = random.order()
        val expectedTotal = order.items.sumOf { it.quantity * it.unitPrice }
        assertEquals(expectedTotal, order.calculateTotal(), 0.01,
            "Total mismatch for order: $order")
    }
}
```

### Stateful Property Example

```kotlin
@Test
fun `invariant - repository count matches actual items`() {
    val repository = InMemoryRepository()

    repeat(100) {
        // Random operation
        when (random.nextInt(3)) {
            0 -> repository.save(random.entity())
            1 -> repository.findAll().randomOrNull()?.let { repository.delete(it.id) }
            2 -> { /* no-op */ }
        }

        // Invariant must hold after every operation
        val count = repository.count()
        val actualSize = repository.findAll().size
        assertEquals(actualSize, count, "Count mismatch")
        assertTrue(count >= 0, "Negative count")
    }
}
```

## Best Practices

### Use Deterministic Seeds

```kotlin
// Good: Reproducible
private val random = Random(seed = 42)

// Bad: Different every run
private val random = Random.Default
```

Deterministic seeds make failures reproducible. When a test fails, the same seed will produce the same sequence of random values.

### Include Input in Failure Messages

```kotlin
// Good: Shows what failed
assertEquals(expected, actual, "Failed for input: $input")

// Bad: No context
assertEquals(expected, actual)
```

### Start with Fewer Iterations

```kotlin
// Good: Fast feedback during development
repeat(100) { /* ... */ }

// Increase for CI/thorough testing
repeat(1000) { /* ... */ }
```

### Test One Property Per Test

```kotlin
// Good: Focused property
@Test fun `roundtrip - save then load`() { /* ... */ }
@Test fun `idempotence - delete twice`() { /* ... */ }

// Bad: Multiple properties mixed
@Test fun `various properties`() {
    // roundtrip test
    // idempotence test
    // invariant test
}
```

### Name Tests as Property Statements

```kotlin
// Good: Describes the property
`roundtrip - encode then decode returns original`
`idempotence - trim applied twice equals trim once`
`invariant - balance never negative after withdrawal`

// Bad: Describes implementation
`test encoding`
`test trim function`
`test withdrawal`
```

## Common Property Categories by Module Type

### Repository/Storage
- Roundtrip: save then find
- Idempotence: delete
- Invariant: count >= 0, findAll size matches count

### Serialization/Encoding
- Roundtrip: encode/decode, serialize/deserialize
- Invariant: encoded output is valid format

### Validation
- Invariant: valid input always accepted
- Invariant: invalid input always rejected
- Idempotence: validation result same on repeated calls

### State Machines
- Invariant: only valid transitions allowed
- Invariant: state always in defined set
- Idempotence: repeated events in stable state

### Collections/Aggregates
- Invariant: computed values match item values
- Commutativity: merge operations (if applicable)
- Associativity: combine operations (if applicable)

### Caching
- Idempotence: get from cache multiple times
- Roundtrip: put then get
- Invariant: cache size within limits
