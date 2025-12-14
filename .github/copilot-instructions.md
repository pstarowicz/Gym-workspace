
## GymGuard / GymSetTracker — Workspace Copilot Instructions

Purpose
-------

This file gives workspace-wide guidance for Copilot and contributors working across the two projects in this workspace: `GymGuardUI` (the test automation framework) and `GymSetTracker` (the application under test). The key rule: GymGuardUI tests must reference app-provided stable attributes (for example `data-test-id`) when creating locators so locators are stable and consistent across runs.

Why this matters
-----------------

- The application (`GymSetTracker`) exposes stable selector attributes in its frontend. Tests must not invent or assume arbitrary DOM IDs. Instead, they should reference attributes the app renders (for example `data-test-id` or element `id`).
- Consistent locators reduce flakiness and make the test framework reusable in CI and local development.

Locator & page naming rules (workspace-level)
---------------------------------------------

1. Use test-data IDs as the primary source of truth.
	- Pattern: `data-test-id` attributes should be in the form `<entity>-<id>` where `<entity>` is a stable name (exercise, workout, user, record) and `<id>` is the canonical id or stable token provided by the application.
	- Example: `data-test-id="exercise-42"` — the test should use this value when locating the exercise row or card for id 42.

2. Page object / page name conventions must mirror the app's page responsibilities.
	- Use these canonical page names when generating POM classes and test names: `LoginPage`, `RegisterPage`, `DashboardPage`, `ExercisesPage`, `WorkoutsPage`, `PersonalRecordsPage`, `ProfilePage`.
	- Test classes should use `*Tests` suffix (e.g., `ExercisesTests`).

3. Locator formatting conventions (stable, machine-friendly):
	- Prefer `data-test-id` attributes: `[data-test-id="<entity>-<id>"]`.
	- If `id` must be used, prefer `id="<entity>-<id>"`.
	- Avoid fragile CSS that depends on position or styling (e.g., :nth-child) for primary selectors.

4. Locator generation guidance for Copilot when asked to produce code:
	- Resolve the test-data id from GymSetTracker first — do not fabricate an id.
	- If the user did not provide the id in the prompt, ask for the id or indicate where to fetch it (see 'Where test-data lives').
	- Generate locators that include the entity name and id: e.g., `By.cssSelector("[data-test-id='exercise-" + exerciseId + "']")` in Java, or `page.locator('[data-test-id="exercise-' + exerciseId + '"]')` in Playwright.

Where to find stable IDs (quick pointers)
----------------------------------------

- The application DOM and frontend components are the source of stable selector attributes; there is no canonical database file containing selector ids. Look for attributes the app renders (e.g., `data-test-id`) in `GymSetTracker/frontend/src/` or inspect the running UI.
- If a stable attribute is missing, coordinate with the app team to add `data-test-id` attributes following the `<entity>-<id>` pattern.

Examples (Java / Selenium style)
--------------------------------

- Locator using test-data id (assume variable `exerciseId`):

	- By.cssSelector("[data-test-id='exercise-" + exerciseId + "']")

- Page object naming: `ExercisesPage.java` with method `openExerciseById(String exerciseId)` that uses the selector above.

Guidance for Copilot-generated code (concise checklist)
-----------------------------------------------------

-- Do: use `data-test-id` / `id` that includes the canonical or app-provided stable id/token.
- Do: create a small helper method to build selectors from entity + id to avoid duplication (e.g., `selectorFor(entity, id)`).
- Don't: invent numeric ids or hard-code values that are not provided by the application or agreed with the app team.

Examples of prompts Copilot may receive and how to respond
--------------------------------------------------------

- Bad prompt response: Generate a selector `#exercise-123` without checking if `123` exists.
-- Good prompt response: If the prompt contains `exerciseId=42` generate a locator using `exercise-42`. If the prompt lacks the id, ask for it or request the canonical id from the app team or inspect the running UI.

Notes on cross-repo modifications
--------------------------------

-- If tests need a `data-test-id` attribute that the app does not currently expose, propose a small frontend change in a PR to `GymSetTracker` that adds `data-test-id` using the canonical `<entity>-<id>` scheme on the relevant element(s). Include test coverage that verifies the attribute exists.

Verification & QA
-----------------

-- After Copilot generates page objects or locators, run a quick grep in the workspace for common patterns (`data-test-id=`, `exercise-`, `workout-`) to ensure the generated code references the canonical id patterns.

Contact / follow-ups
--------------------

If uncertain about a specific id or mapping, open an issue or ask a maintainer. When making changes to GymSetTracker to support tests (e.g., adding `data-test-id` attributes), keep the changes minimal and include a short explanation in the PR.

Completion summary
------------------

This file enforces one key rule: tests must reference app-provided stable attributes (for example `data-test-id`) when producing locators and page names. Use the pointers above to find those attributes, and update framework tests or app markup accordingly if a stable attribute is missing.
