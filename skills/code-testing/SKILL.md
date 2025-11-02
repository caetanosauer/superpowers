---
name: code-testing
description: Use after implementing a feature component to verify it comprehensively - validates architecture through systematic testing of contracts, edge cases, and error conditions
---

# Code Testing

## Overview

Implement clean architecture first, then verify it comprehensively.

**Core principle:** Architecture drives design. Tests validate it. Both matter.

**When to use:** After implementing a complete, self-contained feature component.

## When to Use

**Use this skill when:**
- You've implemented a feature component (logical, self-contained functionality)
- The code works and stands alone
- You're ready to verify it comprehensively

**Examples of complete components:**
- A retry mechanism with exponential backoff
- An input validation function with error messages
- A data transformer that normalizes API responses
- A cache layer with eviction policy

**Don't use this skill for:**
- Exploratory prototypes (throw away, no tests needed)
- Configuration files or data
- Generated code
- Work in progress that isn't complete yet

**Announce at start:** "I'm using code-testing to verify this component."

## The Process

### Phase 1: Recognition

**Identify when you have a testable unit:**

Component Recognition Signals:
- ✓ Has a clear, single responsibility (does one thing well)
- ✓ Has defined inputs and outputs (clear API surface)
- ✓ Can be used independently (minimal coupling to other incomplete pieces)
- ✓ Handles its own edge cases internally
- ✓ Functions correctly in isolation

**Not Yet Complete:**
- Half of a feature (authentication without session management)
- Untested assumptions (assumes input is always valid)
- Dependent on unfinished code (calls function that doesn't exist yet)

**The Recognition Moment:**
When you think "this piece works and stands alone," invoke this skill.

### Phase 2: Analysis

**Analyze the component systematically before writing tests:**

#### Step 1: Define the Contract

Document:
- What does this component promise to do?
- What are the inputs and their constraints?
- What are the outputs and their guarantees?
- **What are the pre-conditions, post-conditions, and invariants?**

**Pre-conditions:** What must be true before calling this code?
**Post-conditions:** What is guaranteed to be true after execution?
**Invariants:** What properties remain true throughout execution?

#### Step 2: Document and Enforce Contracts

For each pre-condition, post-condition, and invariant, decide:

**Should this be enforced with assertions in the code?**
- Pre-conditions that catch programming errors → `assert()` or runtime check
- Invariants that must never be violated → assertions in the implementation
- Post-conditions that validate outputs → assertions before returning

**Should this be tested with test cases?**
- Boundary conditions and edge cases → test cases
- Error handling behavior → test cases
- Integration scenarios → test cases

**When to use assertions vs tests:**
- **Assertions:** Contract violations that indicate bugs in the caller or implementation
- **Tests:** Verify behavior is correct across valid inputs and expected error cases

#### Step 3: Identify Test Scenarios

Three categories to explore:

**Happy Paths:**
- What are the typical, expected use cases?
- What's the "normal" path through the code?

**Edge Cases & Boundaries:**
- Empty inputs, null values, undefined
- Minimum/maximum values
- Boundary conditions (0, 1, n-1, n)
- Unusual but valid inputs

**Error Conditions:**
- Invalid inputs
- Precondition violations
- External failures (network, filesystem, dependencies)
- What should happen when things go wrong?

#### Step 4: Reason About Coverage

Ask: "If I added these assertions and wrote these test cases, would they comprehensively verify this component?"
- Does each test prove something different?
- Are assertions catching contract violations?
- Are there gaps in coverage?

**Output:** A map of both assertions to add and test cases to write.

### Phase 3: Implementation

**Write the tests and add assertions:**

#### Step 1: Add Assertions to Implementation

For each identified pre-condition, post-condition, and invariant:

<Good>
```python
def process_items(items: list[Item]) -> Result:
    # Pre-condition assertion
    assert len(items) > 0, "process_items requires non-empty list"

    result = transform(items)

    # Post-condition assertion
    assert result.is_valid(), "result must be valid after processing"
    return result

def balance_tree(node: Node) -> Node:
    # ... balancing logic ...

    # Invariant assertion
    assert is_balanced(node), "tree must remain balanced after operation"
    return node

def calculate_discount(price: float, rate: float) -> float:
    result = price * (1 - rate)

    # Post-condition assertion
    assert 0 <= result <= price, "discount cannot increase price or go negative"
    return result
```
Clear messages, catches contract violations
</Good>

**Assertion Guidelines:**
- Use language-specific assertion mechanisms (assert, debug_assert, etc.)
- Write clear assertion messages explaining what failed
- Place pre-conditions at function entry
- Place post-conditions before return statements
- Place invariant checks after state-changing operations

#### Step 2: Write Test Cases

For each test scenario identified in analysis:

<Good>
```python
def test_retries_failed_operations_three_times():
    """Retry mechanism attempts operation 3 times before giving up."""
    attempts = 0

    def failing_operation():
        nonlocal attempts
        attempts += 1
        if attempts < 3:
            raise ConnectionError("Failed")
        return "success"

    result = retry_operation(failing_operation)

    assert result == "success"
    assert attempts == 3

def test_rejects_empty_email():
    """Validation rejects empty email with clear error message."""
    result = validate_form({"email": ""})

    assert result.is_error()
    assert result.error == "Email required"
```
Clear names, test one behavior, real code
</Good>

<Bad>
```python
def test_validation():
    """Test validation."""
    mock = Mock()
    mock.email.return_value = ""
    result = validate_form(mock)
    mock.email.assert_called()
```
Vague name, tests mock not code
</Bad>

**Good Test Qualities:**
- Clear descriptive names ("rejects empty email", not "test1")
- Test one behavior per test
- Use real code, avoid mocks unless necessary
- Arrange-Act-Assert structure

#### Step 3: Organize Tests by Category

Structure tests to mirror your analysis:
- Happy path tests first
- Edge cases and boundaries
- Error conditions last

This makes it clear what's covered and what's missing.

### Phase 4: Verification

**Verify tests and assertions actually work:**

#### Step 1: Run the Tests

Execute your test suite for the component:

```bash
# Language-agnostic - use your project's test runner
pytest path/to/test_component.py      # Python
cargo test component_tests            # Rust
go test ./component                   # Go
```

**Confirm:**
- All tests pass
- No errors or warnings in output
- Test output is clean and clear

#### Step 2: Reason About Test Quality

For each test, ask:
- Does this test actually verify the behavior I intended?
- Could this test pass even if the implementation is wrong?
- Am I testing the right thing, or just testing implementation details?

**Red flags:**
- Test passes but you're not sure why
- Test is very similar to implementation code
- Test uses mocks for everything
- Test name doesn't match what it actually tests

#### Step 3: Verify Assertions Work

**For critical assertions, consider:**
- Temporarily violating the pre-condition to ensure assertion fires
- Check that assertion messages are clear and actionable
- Ensure assertions are enabled in development/test builds

**You don't need to break everything** - use judgment about which assertions are critical enough to verify.

#### Step 4: Report Findings

If tests reveal design issues:
- Point out the problem clearly
- Suggest specific fixes
- **Do not proactively fix** - wait for confirmation
- Examples: "This function is hard to test because it does three things. Consider splitting into X, Y, Z."

**Final Check:**
- Component works? ✓
- Tests prove it? ✓
- Tests catch bugs? ✓
- Assertions enforce contracts? ✓

## Common Rationalizations

**If you catch yourself thinking these thoughts, STOP:**

| Excuse | Reality |
|--------|---------|
| "The code works, tests would just slow me down" | Working now ≠ works after changes. Tests prevent regressions. |
| "This is too simple to need tests" | Simple code breaks. Tests take minutes, debugging takes hours. |
| "I'll add tests later when I have time" | Later never comes. Untested code becomes legacy immediately. |
| "Manual testing is faster for this" | Manual testing is faster once. Automated testing is faster forever. |
| "Adding assertions will hurt performance" | Most languages disable assertions in production builds. Development catches bugs early. |
| "This component is hard to test, must be the testing approach" | Hard to test often = design issue. Point it out, don't work around it. |
| "I need to ship quickly, tests can wait" | Shipping untested code = shipping bugs. Fix costs multiply with time. |
| "Tests are comprehensive enough, don't need assertions too" | Tests verify behavior, assertions catch contract violations. Different purposes. |

**Red Flags - You're Skipping Discipline:**
- Marking work complete without running tests
- Writing one happy-path test and calling it done
- Skipping edge cases "because they're unlikely"
- Avoiding assertions "to keep code clean"
- "I tested it manually, it works"

**All of these mean: Go back and follow the skill properly.**

## Integration with Other Skills

- Use `systematic-debugging` if tests reveal bugs
- Use `brainstorming` before implementing if design is unclear
- This skill complements (not replaces) architectural thinking

## Summary

**The Four Phases:**

1. **Recognition:** Identify complete, self-contained component
2. **Analysis:** Define contract, identify pre/post conditions, invariants, test scenarios
3. **Implementation:** Add assertions to code, write comprehensive test cases
4. **Verification:** Run tests, reason about quality, verify critical assertions

**Key Principle:**
Architecture drives design. Tests validate it. Both matter. This skill ensures validation happens comprehensively without constraining architectural decisions.
