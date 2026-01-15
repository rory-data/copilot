# Testing Anti-Patterns

This reference covers common testing pitfalls and how to avoid them. Tests must verify real behaviour, not mock behaviour. Mocks are a means to isolate components, not the thing being tested.

**Core principle:** Test what the code does, not what the mocks do.

## The Iron Laws

1. **NEVER** test mock behaviour
2. **NEVER** add test-only methods to production classes
3. **NEVER** mock without understanding dependencies

---

## Anti-Pattern 1: Testing Mock Behaviour

**The violation:**

```python
# ❌ BAD: Mocking a repository and then testing that it returns the mocked value
def test_get_user_directly():
    mock_repo = Mock()
    mock_repo.get_user.return_value = {"id": 1, "name": "Fake User"}

    # We are verifying the mock returns what we told it to return
    # This tests NOTHING in our application logic.
    result = mock_repo.get_user(1)
    assert result["name"] == "Fake User"
```

**Why this is wrong:**

- You're verifying the mock works, not that your application works.
- The test tells you nothing about real behaviour.
- It provides false confidence.

**The correction:** "Are we testing the behaviour of a mock?"

**The fix:**

```python
# ✅ GOOD: Test the service logic that USES the mock
def test_user_service_logic():
    mock_repo = Mock()
    mock_repo.get_user.return_value = {"id": 1, "name": "John Doe"}

    service = UserService(mock_repo)
    # Test how the service processes the data from the repository
    display_name = service.get_formatted_name(1)

    assert display_name == "User: John Doe"
```

---

## Anti-Pattern 2: Test-Only Methods in Production

**The violation:**

```python
# ❌ BAD: destroy() or reset() added only for test cleanup
class Session:
    def __init__(self, user_id):
        self.user_id = user_id

    def destroy_for_test(self):
        """Only used in tests to clean up files."""
        # Cleanup logic...
        pass
```

**Why this is wrong:**

- Production classes are polluted with code that should never run in production.
- Dangerous if accidentally called.
- Violates separation of concerns.

**The fix:** Use fixtures or test utilities for cleanup.

```python
# ✅ GOOD: Test utilities or fixtures handle cleanup
@pytest.fixture
def session():
    s = Session(user_id=1)
    yield s
    # Cleanup happens HERE in the fixture, not in the class
    cleanup_session_files(s.user_id)
```

---

## Anti-Pattern 3: Mocking Without Understanding

**The violation:**

```python
# ❌ BAD: Mocking a method that has side effects the test depends on
def test_server_registration(monkeypatch):
    # Mocking 'write_config' prevents the file from being created
    monkeypatch.setattr("myapp.config.write_config", lambda path, data: None)

    register_server("server-1")
    # This fails because register_server depends on the file being there for validation!
    assert is_server_registered("server-1") is True
```

**Why this is wrong:**

- Over-mocking breaks the logic the test actually needs to exercise.
- Test fails mysteriously or passes for the wrong reasons.

**The fix:** Mock at the correct level (e.g., the slow network call, not the local file write).

```python
# ✅ GOOD: Mock the external dependency, preserve local side effects
def test_server_registration_with_mock_api(monkeypatch):
    # Mock the slow API call, let local file writing happen normally (in-memory or temp)
    monkeypatch.setattr("myapp.api.notify_central_registry", Mock())

    register_server("server-1")
    assert is_server_registered("server-1") is True
```

---

## Anti-Pattern 4: Incomplete Mocks

**The violation:**

```python
# ❌ BAD: Partial mock - only fields you think you need
mock_response = Mock()
mock_response.json.return_value = {
    "status": "success",
    "data": {"user_id": 1}
    # Missing: 'metadata' field that downstream code expects
}
```

**Why this is wrong:**

- **Partial mocks hide assumptions.** You only mocked what you knew about.
- Downstream code accessing `response.json()["metadata"]` will crash.
- Tests pass but the system fails in integration/production.

**The Iron Rule:** Mock the COMPLETE data structure as it exists in reality.

**The fix:**

```python
# ✅ GOOD: Mirror real API completeness
mock_response.json.return_value = {
    "status": "success",
    "data": {"user_id": 1},
    "metadata": {"request_id": "abc-123"} # Include all standard fields
}
```

---

## Anti-Pattern 5: Integration Tests as Afterthought

**The violation:**

- Implementation is claimed "complete" before any tests (unit or integration) exist.
- "Ready for testing" sent to QA/Partners without automated verification.

**The fix:** Follow TDD or ensure tests are part of the "Definition of Done".

1. Write failing test.
2. Implement to pass.
3. Refactor.
4. **THEN** claim complete.

---

## When Mocks Become Too Complex

**Warning signs:**

- Mock setup is longer than the test logic.
- You are mocking everything to get the test to pass.
- Test breaks every time the implementation details change.

**Consider:** Integration tests with real components are often simpler and more reliable than complex mocks.

---

## Quick Reference

| Anti-Pattern                  | Fix                                    |
| ----------------------------- | -------------------------------------- |
| Assert on mock return values  | Test the logic using the mock          |
| Test-only methods in prod     | Move cleanup to fixtures/utilities     |
| Mocking without understanding | Understand side effects before mocking |
| Incomplete mocks              | Mirror real API structure completely   |
| Tests as afterthought         | TDD - tests first                      |

## Red Flags

- Checking if a mock exists but not what it does.
- `if "pytest" in sys.modules:` or similar in production code.
- Mocking "just to be safe".
- Can't explain why a mock is needed.
