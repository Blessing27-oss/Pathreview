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


## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
All sub-tasks from PLAN.md are complete. Reproduced the bug (13 failures confirmed), then fixed all 13 failing tests by changing `mock_result = AsyncMock()` to `mock_result = Mock()` — keeping `db.execute` as `AsyncMock` since the service awaits it, but making the result object a plain `Mock` so `.scalars().first()` and `.scalars().all()` resolve correctly. Also corrected a secondary bug in `test_list_reviews_ordered_by_created_at` where `assert_called_once()` was wrong because `list_reviews` calls `db.execute` twice. Added `AsyncSession` type annotations to `review_service.py` and a mypy override for `tests.*` to satisfy the pre-commit hook (consistent with `make typecheck` already excluding `tests/` by design). All 19 tests now pass.

**Next steps:**
Open a draft PR, get peer feedback, and finalise the PR description.

**Blockers:**

---

### Check-in 2 (end of week)

**PR link:** [link to your submitted pull request]

**Branch:** [the branch name you worked on, e.g. `fix/123-short-description`]

**What you built:**
[1–3 sentences summarizing what your fix does and how it works]

**Tests added or updated:**
[Which test files did you touch? What do they cover?]

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

**Draft PR feedback received from:** [name or Slack handle, or "none"]
