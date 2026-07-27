## Week 7 — Issue selection

**Issue link:** [github.com/ascherj/pathreview/issues/158](https://github.com/ascherj/pathreview/issues/158)

**Issue title:** review_service unit tests misconfigure async mocks — 13 of 19 tests fail

**Tier:** [Yes ] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The test mocks are set up incorrectly for async database operations. The service code does await
  db.execute(...) and then calls .scalars().first() or .scalars().all() on the result. But the mock returns a coroutine
  object instead of a result object, so when the service tries to call .first() or .all() on it, it crashes with: AttributeError: 'coroutine' object has no attribute 'first'. A root causec is result.scalars() is mocked as an async method (returning a coroutine) when it should be a regular synchronous method returning a mock result object. A possible fix will be to use AsyncMock for db.execute() and a plain MagicMock for the result object — without modifying the service code
  itself.

**Branch name:** fix/158-unit-tests-misconfigure-async-mocks

**Setup confirmation:** [Yes ] App runs locally at localhost:5173

**Cohort ledger:** [Yes ] Issue added to cohort ledger

**Reproduction steps:**

1. Make sure the virtual environment is activated:
   ```
   source .venv/bin/activate
   ```
2. Run the failing tests:
   ```
   pytest tests/unit/test_review_service.py -q
   ```
3. Observed output: 13 failed, 6 passed
   - Tests calling `get_review` fail with:
     `AttributeError: 'coroutine' object has no attribute 'first'`
   - Tests calling `list_reviews` fail with:
     `AttributeError: 'coroutine' object has no attribute 'all'`

**Root cause observed:**
In each failing test, `mock_result` is created as `AsyncMock()`. Because `AsyncMock` makes every attribute access return a coroutine, calling `mock_result.scalars()` returns a coroutine instead of a plain object. The service code then calls `.first()` or `.all()` on that coroutine, which raises `AttributeError` since coroutines have no such methods.
