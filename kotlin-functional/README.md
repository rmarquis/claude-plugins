# Kotlin Functional-Idiomatic Plugin

A Claude Code plugin for generating **functional-first, idiomatic Kotlin** implementations with comprehensive KDoc documentation.

## Philosophy

**Three Core Principles:**

1. **Functional-First**: Favor immutability, pure functions, expression-oriented code, and higher-order functions
2. **Idiomatic Kotlin**: Leverage Kotlin's strengths (data classes, sealed classes, extension functions, scope functions, null safety)
3. **Documentation as Code**: KDoc is not an afterthought—it's integral to the implementation, documenting contracts, invariants, and design decisions

## Position in the Workflow

```
requirements → architecture → specification → kotlin-functional → implementation
     ↓              ↓               ↓                 ↓
  docs/         docs/          docs/          src/main/kotlin/
requirements/  architecture/  specifications/  (idiomatic implementation)
```

**Interface Points:**
- **Reads**: `docs/specifications/<feature>/` - contract, behavior, and property specs
- **Reads**: `docs/architecture/<feature>.md` - module structure, interfaces, responsibilities
- **Generates**: `src/main/kotlin/` - functional, idiomatic implementations with KDoc
- **Optional**: Can review existing code and suggest functional refactorings

## Commands

### `/implement-functional [spec-or-module]`

Generate functional Kotlin implementation from specifications.

**Process:**
1. Locates specifications in `docs/specifications/<feature>/`
2. Reads architecture from `docs/architecture/`
3. Analyzes contracts, behaviors, and properties
4. Generates functional implementation with:
   - Immutable domain models (data classes, sealed hierarchies, value classes)
   - Pure functions with type-safe error handling (Result types)
   - Extension functions for domain operations
   - Comprehensive KDoc documenting contracts, invariants, and postconditions
5. Outputs to `src/main/kotlin/<package>/` with TODOs for I/O operations

**Examples:**
```bash
/implement-functional user-auth
/implement-functional payment-processing
/implement-functional com.example.orders
```

### `/review-functional [file-or-dir]`

Review code for functional and idiomatic Kotlin patterns.

**Process:**
1. Scans Kotlin files from specified path
2. Analyzes against functional/idiomatic checklist
3. Generates review report with:
   - Overall functional/idiomatic score
   - Issues categorized by severity
   - Concrete refactoring suggestions with before/after code
   - Positive patterns (celebrating good code!)
4. Offers interactive refactoring with user approval

**Examples:**
```bash
/review-functional src/main/kotlin/userauth
/review-functional src/main/kotlin/userauth/UserRepository.kt
```

## Functional Patterns Supported

### 1. Algebraic Data Types (ADTs)
- Sealed classes and sealed interfaces for exhaustive when-expressions
- Data classes for immutable domain models
- Value classes for type-safe primitives (zero overhead)

### 2. Immutable Domain Models
- `val` properties by default
- `copy()` for state transformations
- Smart constructors with validation
- Explicit `init` blocks for invariants

### 3. Functional Error Handling
- `Result<T>` for type-safe error handling
- Sealed error hierarchies for granular error cases
- No exceptions for expected failures

### 4. Higher-Order Functions
- Functions as first-class values
- Behavior composition without inheritance
- Strategy pattern via function parameters

### 5. Extension Functions
- Domain operations as extensions
- Behavior organization without class pollution
- Pipeline-friendly APIs

### 6. Expression-Oriented Programming
- `when` expressions (not statements)
- Single-expression functions
- Minimal mutable state

### 7. Scope Functions
- `let`, `run`, `with`, `apply`, `also` for readable pipelines
- Side effects isolated to specific scopes

### 8. Lazy Sequences
- Efficient transformations without intermediate collections
- Infinite sequences with `generateSequence`

## KDoc Documentation Standards

Every public declaration includes KDoc with:

1. **Summary**: One-sentence description
2. **Detailed Description**: How it works, design decisions
3. **Preconditions**: What must be true before calling (via `## Preconditions`)
4. **Postconditions**: What is guaranteed after calling (via `## Postconditions`)
5. **Invariants**: Properties that always hold (via `## Invariants`)
6. **Side Effects**: Observable effects beyond return value (via `## Side Effects`)
7. **Examples**: Code examples for complex usage (via `@sample`)
8. **Related References**: Links to related functions/types (via `@see`)

## Integration with Other Plugins

### With `software-specification`
- Reads kotlin-test specs to understand contracts
- Implements interfaces that specs test against
- Shares domain models between test and implementation

### With `software-architecture`
- Uses module structure from architecture docs
- Architecture's "deep modules" aligns with functional encapsulation
- Module boundaries become package boundaries

## Anti-Patterns Detected

1. ❌ Mutable data structures → ✅ Immutable transformations
2. ❌ Unnecessary nullable types → ✅ Result types or non-null guarantees
3. ❌ Exception-based flow control → ✅ Result or sealed error hierarchies
4. ❌ Imperative loops → ✅ map, filter, fold, reduce
5. ❌ Inheritance hierarchies → ✅ Interfaces with extensions or higher-order functions
6. ❌ Missing or weak KDoc → ✅ Comprehensive KDoc with contracts
7. ❌ Hidden side effects → ✅ Pure functions or explicit side effect documentation
8. ❌ Unnecessary null checks → ✅ Semantic use of nullable types

## Success Criteria

A well-implemented Kotlin codebase should:

- ✅ **Immutable by default**: 90%+ of properties are `val`
- ✅ **Expression-oriented**: Minimal use of `var` and imperative loops
- ✅ **Type-safe errors**: Result types or sealed hierarchies instead of exceptions
- ✅ **Comprehensive KDoc**: Every public declaration has KDoc with contracts
- ✅ **Null safety**: No unnecessary nullable types or `!!` operators
- ✅ **Functional composition**: Higher-order functions and extension functions used appropriately
- ✅ **Side effects isolated**: Pure functions separated from I/O operations
- ✅ **Readable**: Code reads like prose due to extension functions and scope functions

## Installation

1. Copy the `kotlin-functional/` directory to your Claude Code plugins location
2. The plugin will be automatically discovered by Claude Code
3. Use `/implement-functional` or `/review-functional` commands in your project

## License

MIT
