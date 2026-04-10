## Plan: Add Minimal FastAPI Backend Tests

Add a separate top-level `tests/` directory with a small pytest-based backend suite for the FastAPI app. The recommended approach is to keep the first pass narrow: add pytest as a test dependency, create shared fixtures for `TestClient` and in-memory activity reset, and cover the highest-value smoke paths for the current API without broadening into frontend or static-file testing. Structure each test using the AAA pattern so setup, request execution, and assertions remain clearly separated and easy to extend.

**Steps**
1. Add test tooling support in `/workspaces/skills-getting-started-with-github-copilot/requirements.txt` by including `pytest` alongside the existing FastAPI stack so the repo can run the new suite consistently.
2. Create `/workspaces/skills-getting-started-with-github-copilot/tests/` as the dedicated backend test directory and add `/workspaces/skills-getting-started-with-github-copilot/tests/conftest.py` for shared fixtures. This fixture layer should:
- expose a FastAPI `TestClient` for `src.app:app`
- deep-copy the module-level `activities` store before each test and restore it afterward so POST/DELETE tests do not leak state into one another
3. Add `/workspaces/skills-getting-started-with-github-copilot/tests/test_app_routes.py` for the minimal non-mutating smoke paths. Each test should follow AAA explicitly:
- Arrange: prepare the client and any expected values
- Act: call the route once
- Assert: check redirect target or response payload
This file should cover:
- `GET /` returns the redirect to `/static/index.html`
- `GET /activities` returns a successful response and includes expected activity data shape for at least one known activity
4. Add `/workspaces/skills-getting-started-with-github-copilot/tests/test_app_activity_mutations.py` for the minimal mutating smoke paths. Each test should follow AAA explicitly:
- Arrange: set up the target activity, participant email, and baseline state
- Act: perform one signup or unregister request
- Assert: verify both the response message and the updated in-memory participant list
This file should cover:
- successful signup adds a participant and returns the expected message
- successful unregister removes a participant and returns the expected message
5. Run focused verification with pytest on the new files first, then the full suite. This step depends on steps 1-4.
6. If any failures come from import-path or fixture isolation issues, fix them locally without broadening the scope into additional endpoints or frontend behavior.

**Relevant files**
- `/workspaces/skills-getting-started-with-github-copilot/src/app.py` — source of the FastAPI `app`, the mutable `activities` store, and the four current route handlers to target first
- `/workspaces/skills-getting-started-with-github-copilot/requirements.txt` — add `pytest` so the new backend suite is installable in the repo environment
- `/workspaces/skills-getting-started-with-github-copilot/pytest.ini` — already sets `pythonpath = .`, which should allow tests to import `from src.app import app, activities` from a top-level `tests/` directory
- `/workspaces/skills-getting-started-with-github-copilot/tests/conftest.py` — new shared fixture module for `TestClient` and state reset
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_app_routes.py` — new smoke tests for redirect and activity listing
- `/workspaces/skills-getting-started-with-github-copilot/tests/test_app_activity_mutations.py` — new smoke tests for signup and unregister behavior

**Verification**
1. Install dependencies if needed, then run `pytest -q tests/test_app_routes.py`.
2. Run `pytest -q tests/test_app_activity_mutations.py` to confirm state-reset fixtures isolate the POST/DELETE behavior.
3. Run `pytest -q` from `/workspaces/skills-getting-started-with-github-copilot` as a final repo-level check.

**Decisions**
- Included scope: a separate backend `tests/` directory, pytest setup, shared fixtures, minimal smoke coverage for the existing FastAPI routes, and explicit AAA structure within each new test.
- Excluded scope: frontend tests, UI automation, static asset assertions beyond the root redirect target, and broader edge-case/error-path coverage for now.
- Assumption: keeping `tests/` at the repo root is preferred over colocated tests under `src/`.
- Assumption: `pytest.ini` with `pythonpath = .` is sufficient for imports from `src.app` without adding package boilerplate.
- Decision: prefer straightforward AAA tests over heavily parameterized or overly compact test bodies for this initial suite, because readability is the priority.

**Further Considerations**
1. If you want stronger backend confidence immediately after this minimal pass, the next expansion should be duplicate-signup and missing-activity error tests because those are the main business-rule branches in `/workspaces/skills-getting-started-with-github-copilot/src/app.py`.
2. If dependency hygiene matters, an alternative is to move test-only packages out of `requirements.txt` into a separate dev requirements file, but that is outside the current minimal scope.
