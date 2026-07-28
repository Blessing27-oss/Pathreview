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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [https://github.com/Blessing27-oss/Pathreview/commit/48ed1ff8feafd46cbfbb55a0f3d8eebc04554494](https://github.com/Blessing27-oss/Pathreview/commit/48ed1ff8feafd46cbfbb55a0f3d8eebc04554494)

**Reproduction summary:**
Ran `pytest tests/unit/test_review_service.py -q` with the virtual environment activated and observed 13 failures. Tests calling `get_review` crashed with `AttributeError: 'coroutine' object has no attribute 'first'` and tests calling `list_reviews` crashed with `AttributeError: 'coroutine' object has no attribute 'all'`, confirming that `mock_result = AsyncMock()` causes `scalars()` to return a coroutine instead of a plain result object.

**PLAN.md link:** [https://github.com/Blessing27-oss/Pathreview/blob/fix/158-unit-tests-misconfigure-async-mocks/PLAN.md](https://github.com/Blessing27-oss/Pathreview/blob/fix/158-unit-tests-misconfigure-async-mocks/PLAN.md)

**Walkthrough video (recommended):  [www.loom.com/share/cad333687a314bc398567439426f9246](https://www.loom.com/share/cad333687a314bc398567439426f9246)**
