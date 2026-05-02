---
description: "Guidelines for GitHub Copilot to write comments to achieve self-explanatory code with less comments."
applyTo: "**/*.py"
---

# Code Commenting

Write code that speaks for itself. Comment only to explain **why**, not what.

## Avoid These Comment Types

**Obvious comments:**

```python
# Bad: States the obvious
counter = 0  # Initialise counter to zero
counter += 1  # Increment counter by one
```

**Redundant comments:**

```python
# Bad: Repeats the code
def get_user_name(user):
    return user.name  # Return the user's name
```

**Outdated comments:**

```python
# Bad: Comment doesn't match the code
# Calculate tax at 5% rate
tax = price * 0.08  # Actually 8%
```

## Write These Comment Types

**Complex business logic:**

```python
# Good: Explains why this specific calculation
# Progressive tax brackets: 10% up to $10k, 20% above
tax = calculate_progressive_tax(income, rates=[0.1, 0.2], thresholds=[10_000])
```

**Non-obvious algorithms:**

```python
# Good: Explains the algorithm choice
# Floyd-Warshall for all-pairs shortest paths —
# we need distances between every node pair, not just one source
for k in range(n):
    for i in range(n):
        for j in range(n):
            dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
```

**Regex patterns:**

```python
# Good: Explains what the pattern matches
# Email: username@domain.tld — does not allow IP-address domains
EMAIL_RE = re.compile(r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$")
```

**API constraints or gotchas:**

```python
# Good: Explains external constraint
# GitHub API: 5,000 requests/hour for authenticated users
await rate_limiter.wait()
response = await client.get(github_api_url)
```

**Public APIs (docstrings):**

```python
def calculate_compound_interest(
    principal: float,
    rate: float,
    time: float,
    compound_frequency: int = 1,
) -> float:
    """Calculate compound interest using the standard formula.

    Args:
        principal: Initial amount invested.
        rate: Annual interest rate as a decimal (e.g., 0.05 for 5%).
        time: Time period in years.
        compound_frequency: How many times per year interest compounds.

    Returns:
        Final amount after compound interest.
    """
```

**Constants with non-obvious values:**

```python
MAX_RETRIES = 3        # Based on P99 network reliability measurements
API_TIMEOUT_MS = 5000  # Lambda max is 15s; leaving headroom for retries
```

## Annotations

```python
# TODO: Replace with proper auth after security review
# FIXME: Memory leak in production — investigate connection pooling
# HACK: Workaround for bug in requests v2.1.0 — remove after upgrade
# NOTE: Assumes UTC timezone for all calculations
# SECURITY: Validate input before use in query to prevent injection
```

## Anti-Patterns

```python
# Bad: Don't comment out code — use git instead
# def old_function(): ...
def new_function(): ...

# Bad: Don't maintain history in comments — that's what git log is for
# Modified by John 2023-01-15
# Fixed bug reported by Sarah 2023-02-03
```

## Decision Framework

Before writing a comment, ask:

1. Is the code self-explanatory? → No comment needed
2. Would a better name eliminate the need? → Rename instead
3. Does this explain WHY, not WHAT? → Good comment
4. Will this help a future maintainer? → Good comment

## Checklist

- [ ] Comments explain why, not what
- [ ] No commented-out code
- [ ] No changelog comments
- [ ] Public functions have docstrings
- [ ] Constants with non-obvious values are explained
