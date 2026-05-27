---
description: "How to structure, test, commit, and scope code changes. Covers vertical slicing, TDD, change size, and scope discipline."
applyTo: "**"
---

# Implementation Discipline

## Slice vertically, not horizontally

Build one complete feature path per task: schema change + data access + endpoint/handler + test.
Do not build all schema changes first, then all data-access changes, then all endpoints.
Each slice must leave CI green and the system in a deployable state.

Dependency order within a slice: data model -> data access -> handler -> test -> commit.
Never commit a broken test. Never skip a test to make CI pass.

**Task sizing rule:** If a task would touch more than five files, split it into
two tasks. An agent performs best on small, focused changes.

## Test-first for new behaviour

Write the failing test before the implementation (Red-Green-Refactor).

**Test structure (AAA):**

```python
# Arrange: set up data and preconditions
# Act: call the function under test
# Assert: verify the outcome
```

**Naming:** `test_<unit>_<expected_behaviour>_<condition>`.
Example: `test_resolver_returns_empty_list_when_package_not_found`.

**Parametrize for input variation:**

```python
@pytest.mark.parametrize("input,expected", [
    ("case_a", result_a),
    ("case_b", result_b),
])
def test_behaviour_under_variation(input, expected):
    ...
```

**Mock at system boundaries only.** Mock the database, HTTP clients, filesystem,
and clock. Do not mock internal business logic in unit tests that are meant to
test that logic.

| Mock these                           | Do not mock these                        |
|--------------------------------------|------------------------------------------|
| Network / HTTP clients               | Pure transformation functions            |
| File system calls                    | Validation logic                         |
| External process calls               | Internal data-access classes under test  |
| Clock (`datetime.now()`) when needed | Dependency injection factories           |

**Anti-patterns:**

| Anti-pattern                          | Problem                                           |
|---------------------------------------|---------------------------------------------------|
| Testing implementation details        | Breaks on refactor without finding real bugs      |
| `pytest.skip` permanently             | Dead test masking a real issue                    |
| Shared mutable state across tests     | Tests pollute each other                          |
| Asserting only status code, not body  | Misses schema contract regressions                |

**Targeted test runs before full suite.** When iterating on a fix, run the
specific file first, then the full suite only once the targeted run is green:

```
pytest tests/unit/test_specific.py -x
```

## Change size

Target ~100 lines changed per commit. Up to ~300 lines is acceptable for a single
logical change. Over ~1000 lines: split the change before committing.

Every commit must:
1. Do one logical thing.
2. Leave CI green.
3. Follow conventional commit format: `type(scope): description`.

## Change summary

After completing any non-trivial change, record a summary in session notes or
the PR body:

```
CHANGES MADE:
- src/module/component.py: added create_pending()
- tests/unit/test_component.py: added 3 parametrize cases

THINGS I DID NOT TOUCH (intentionally out of scope):
- src/other/related.py: related but not part of this task

POTENTIAL CONCERNS:
- create_pending() assumes id is always present. Confirm with caller.
```

This pattern catches wrong assumptions early and proves scope discipline.

## Stopping rule

Stop and escalate if any of the following occur:

- The task has grown beyond five files and was not pre-approved as a large change.
- A test has been failing for more than two implementation attempts.
- An accepted architectural decision would need to be modified to proceed.
- A new dependency is required that is not already in the manifest.

## Dead code hygiene

After any refactor, scan for orphaned code. List it explicitly in the change
summary. Do not silently delete code. Flag it and ask before removing:

```
DEAD CODE IDENTIFIED:
- get_legacy_dep() in component.py -- replaced by create_pending()
-- Safe to remove?
```

## Scope discipline (Chesterton's Fence)

Before changing or removing any existing code, understand why it exists.
Check git history for context. If the reason is unclear, do not remove it --
flag it as a question.

Do not refactor code outside the scope of the current task. Mixed refactor-plus-
feature changes are harder to review and harder to revert. Submit them separately.
