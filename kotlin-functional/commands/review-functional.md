# Command: review-functional

Review Kotlin code for functional programming patterns and idiomatic usage.

## Usage

```bash
/review-functional <file-or-dir>
```

## Arguments

- `<file-or-dir>`: Path to Kotlin file or directory to review
  - Can be a single file (e.g., `src/main/kotlin/UserService.kt`)
  - Can be a directory (e.g., `src/main/kotlin/userauth`)
  - Can be relative or absolute path

## Examples

```bash
# Review a single file
/review-functional src/main/kotlin/userauth/UserService.kt

# Review an entire module
/review-functional src/main/kotlin/userauth

# Review entire source directory
/review-functional src/main/kotlin
```

## What This Command Does

1. **Scans Code**:
   - Reads all `.kt` files from specified path
   - Recursively processes directories

2. **Analyzes Code**:
   - **Immutability**: Checks for `val` vs `var`, mutable collections
   - **Expression-Oriented**: Checks for statements vs expressions, imperative loops
   - **Error Handling**: Checks for Result types vs exceptions, error documentation
   - **Null Safety**: Checks for `!!` usage, unnecessary nullability
   - **Functional Composition**: Checks for higher-order functions, extensions
   - **KDoc Documentation**: Checks for comprehensive documentation with contracts

3. **Scores Code**:
   - Calculates score for each criterion (0-100)
   - Weights scores to calculate overall score
   - Assigns grade (Excellent/Good/Fair/Poor)

4. **Identifies Issues**:
   - **Critical**: Correctness issues (null safety violations, race conditions)
   - **Major**: Significant functional/idiomatic violations
   - **Minor**: Style and idiom improvements

5. **Generates Report**:
   - Creates `docs/reviews/functional-review-<timestamp>.md`
   - Includes:
     - Summary and scores
     - Issues with severity levels
     - Before/after refactoring examples
     - Positive patterns already in use
     - Recommendations (short/medium/long-term)
     - Refactoring opportunities

6. **Offers Interactive Refactoring**:
   - Option to apply suggested refactorings
   - File-by-file approval process
   - Shows diffs before applying changes

## Review Criteria

### 1. Immutability (25% weight)
- Properties declared with `val` (not `var`)
- Immutable collections used
- State transformations via `copy()` or new instances
- No in-place mutation

**Excellent**: 90%+ properties are `val`, no mutable collections

### 2. Expression-Oriented (15% weight)
- `when` expressions instead of statements
- Single-expression functions
- Minimal temporary variables
- Functional pipelines instead of loops

**Excellent**: Consistently expression-oriented, minimal statements

### 3. Error Handling (20% weight)
- `Result<T>` for expected failures
- Sealed interfaces for domain errors
- No exceptions for flow control
- Documented failure modes

**Excellent**: Type-safe error handling throughout, well-documented

### 4. Null Safety (10% weight)
- Non-null types by default
- No `!!` operator
- Nullable only where semantic
- Result types preferred over nullable returns

**Excellent**: No `!!`, semantic nullability, safe navigation

### 5. Functional Composition (15% weight)
- Higher-order functions for abstraction
- Extension functions for domain operations
- Function composition and chaining
- No inheritance hierarchies for behavior

**Excellent**: Extensive use of higher-order functions, rich extensions

### 6. KDoc Documentation (15% weight)
- KDoc on all public declarations
- Preconditions and postconditions documented
- Invariants documented for classes
- Side effects explicitly noted
- Examples for complex APIs

**Excellent**: Comprehensive KDoc with contracts, invariants, examples

## Report Structure

```markdown
# Functional Review: <module-name>

**Date**: 2025-01-29
**Reviewed**: src/main/kotlin/userauth
**Overall Score**: 79/100 (Good)

## Summary
[High-level assessment of code quality]

## Scores by Criterion
| Criterion | Score | Grade |
|-----------|-------|-------|
| Immutability | 85/100 | Good |
| Expression-Oriented | 70/100 | Fair |
| Error Handling | 90/100 | Excellent |
| Null Safety | 95/100 | Excellent |
| Functional Composition | 60/100 | Fair |
| KDoc Documentation | 75/100 | Good |
| **Overall** | **79/100** | **Good** |

## Issues Found

### Critical Issues (0)
[None found]

### Major Issues (3)

#### Issue 1: Mutable State in Domain Model
**File**: src/main/kotlin/User.kt:15
**Severity**: Major

**Current Code**:
```kotlin
data class User(
    val id: Long,
    var email: String,  // ❌ Mutable
    var role: Role
)
```

**Suggested Refactoring**:
```kotlin
data class User(
    val id: Long,
    val email: String,  // ✅ Immutable
    val role: Role
) {
    fun withRole(newRole: Role): User = copy(role = newRole)
}
```

**Rationale**: Mutable domain models lead to temporal coupling.

---

### Minor Issues (5)
[List of minor improvements]

## Positive Patterns
- ✅ Excellent use of sealed interfaces in PaymentService.kt
- ✅ Strong KDoc in OrderRepository.kt

## Recommendations

### Short-Term
1. Convert `var` to `val` in data classes
2. Replace exceptions with Result types

### Medium-Term
1. Extract loops into functional pipelines
2. Add extension functions for domain operations

### Long-Term
1. Separate pure logic from I/O effects
2. Introduce value classes for primitives

## Refactoring Opportunities
[List with effort estimates]
```

## Interactive Refactoring

After generating the report, the command offers:

```
Found 8 refactoring opportunities. Would you like me to:
1. Apply all minor refactorings automatically
2. Review and approve each refactoring individually
3. Focus on specific categories (e.g., only error handling)
4. Generate plan without applying changes

Choice: _
```

For each refactoring:
1. Shows diff (before/after)
2. Explains rationale
3. Waits for approval
4. Applies change if approved
5. Runs tests if available

## Anti-Patterns Detected

The review automatically detects these common anti-patterns:

1. **Mutable data structures** (`var`, `mutableListOf`)
2. **Nullable return for failure** (should use Result)
3. **Exception-based flow control** (should use Result/sealed errors)
4. **Imperative loops** (should use map/filter/fold)
5. **Inheritance for behavior** (should use higher-order functions)
6. **Missing or weak KDoc** (should include contracts)
7. **Hidden side effects** (should be documented)
8. **Unnecessary `!!`** (should use safe navigation)

## Scoring Algorithm

```
Overall Score =
  (Immutability × 0.25) +
  (Expression-Oriented × 0.15) +
  (Error Handling × 0.20) +
  (Null Safety × 0.10) +
  (Functional Composition × 0.15) +
  (KDoc × 0.15)

Grades:
  90-100: Excellent
  80-89:  Good
  70-79:  Fair
  60-69:  Poor
  <60:    Needs Improvement
```

## Options (Future)

```bash
# Generate report only, no interactive mode
/review-functional src/ --report-only

# Focus on specific criteria
/review-functional src/ --criteria=immutability,error-handling

# Set minimum score threshold
/review-functional src/ --min-score=80

# Output format
/review-functional src/ --format=json
```

## Output Files

### Review Report
- **Location**: `docs/reviews/functional-review-<timestamp>.md`
- **Format**: Markdown
- **Includes**: Scores, issues, suggestions, recommendations

### Refactoring Log (if changes applied)
- **Location**: `docs/reviews/refactoring-log-<timestamp>.md`
- **Format**: Markdown
- **Includes**: List of applied changes with diffs

## Integration with Other Tools

### With CI/CD
The review can be integrated into CI pipelines:

```yaml
# Example GitHub Action
- name: Review Kotlin Code
  run: claude /review-functional src/ --min-score=75 --report-only
```

### With Pre-commit Hooks
Can be used as a pre-commit check for code quality.

## Success Criteria

A successful review:
- ✅ Identifies concrete, actionable improvements
- ✅ Provides before/after code examples
- ✅ Explains rationale for suggestions
- ✅ Highlights existing good patterns
- ✅ Categorizes issues by severity and effort
- ✅ Offers interactive refactoring assistance

## Notes

- **Non-Destructive**: Review process doesn't modify code unless explicitly approved
- **Context-Aware**: Considers existing project patterns and conventions
- **Educational**: Explains *why* patterns are beneficial, not just *what* to change
- **Pragmatic**: Focuses on high-value improvements, not dogmatic purity
- **Positive**: Celebrates good code as much as identifying improvements

## Troubleshooting

**No issues found:**
- Your code is already highly functional and idiomatic!
- Consider reviewing specific modules for deep analysis

**Too many issues:**
- Focus on high-severity issues first
- Use interactive mode to tackle issues incrementally
- Consider generating a refactoring plan for team discussion

**Generated suggestions don't fit project:**
- Review considers context, but may need manual adjustment
- Use suggestions as starting points for discussion
- Adapt patterns to your specific requirements
