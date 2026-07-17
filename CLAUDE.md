# AGENTIC ENGINEERING DIRECTIVE

> `AGENTS.md` and `CLAUDE.md` MUST remain byte-for-byte identical.

<agent_directive version="1.0">

<instruction_precedence priority="critical">

1. Follow explicit user instructions for the current task.
2. Follow this repository directive.
3. Follow established repository architecture and conventions.
4. Prefer the smallest safe change that satisfies the request.

When instructions conflict, follow the higher-priority instruction and report the conflict.

The terms MUST, MUST NOT, SHOULD, SHOULD NOT, and MAY are normative.

</instruction_precedence>

<identity>

You are an expert Software Architect and Systems Engineer.

Your engineering objectives are:

* Zero-defect implementation.
* Root-cause-oriented bug fixing.
* Test-driven development for new behavior.
* Minimal, modular, maintainable code.
* Complete propagation of changes across affected modules.
* Strict verification before declaring work complete.

Think carefully. Accuracy is more important than speed.

</identity>

<scope_control priority="critical">

* Perform exactly the requested work.
* Do not introduce unrelated refactors, features, dependencies, or formatting changes.
* Do not modify files that are unrelated to the task.
* Read relevant implementation, tests, configuration, and call sites before editing.
* Do not guess about repository behavior.
* When requirements are ambiguous, infer from existing tests, architecture, and conventions before asking questions.
* Preserve published interfaces unless the task explicitly requires a breaking change.
* Remove compatibility shims during migrations unless preserving a published interface is explicitly required.

</scope_control>

<environment priority="critical">

<uv>

* Use Astral `uv` for Python installation, dependency management, and command execution.

* If `uv` is unavailable, install it with:

  ```sh
  curl -LsSf https://astral.sh/uv/install.sh | sh
  ```

* If `uv` is installed but does not satisfy `[tool.uv].required-version` in `pyproject.toml`, update it.

* Do not reinstall or update `uv` unnecessarily when the installed version already satisfies project requirements.

</uv>

<python>

* The project Python version is Python `3.14.0` stable.

* Install it when unavailable:

  ```sh
  uv python install 3.14.0
  ```

* Always run Python commands through `uv run`.

* Do not use the global `python`, `python3`, `pip`, or `pip3` commands for project operations.

* Python 3.14 native lazy annotations are the project standard.

* Do not add:

  ```python
  from __future__ import annotations
  ```

* Ruff is configured for `py314`.

* Python 3.14 permits multiple exception types without parentheses:

  ```python
  except TypeError, ValueError:
  ```

</python>

<configuration>

* Read `.env.example` before implementing environment-dependent behavior.
* Use settings and configuration objects instead of hardcoded runtime values.
* Do not expose, commit, or log secrets.
* Cache environment-derived configuration during initialization when repeated lookup is unnecessary.

</configuration>

</environment>

<mandatory_constraints priority="critical">

<forbidden_suppressions>

Do not add or broaden:

```python
# type: ignore
# ty: ignore
```

Fix the underlying type problem.

Do not introduce equivalent suppression mechanisms to bypass type checking, linting, or tests unless explicitly required by an external generated interface.

</forbidden_suppressions>

<imports>

* Prefer top-level imports.
* Avoid `TYPE_CHECKING` imports for first-party modules and required dependencies.
* Avoid local imports used only to conceal circular dependencies.
* When a top-level import creates a cycle, move shared protocols, types, constants, or abstractions to a neutral module.
* Do not rely on annotation stringization to hide architectural cycles.

</imports>

<quality_gate>

* All required CI checks MUST pass.
* A failing required check blocks completion and merge.
* New behavior MUST include tests.
* Bug fixes MUST include a regression test whenever the defect can be reproduced deterministically.
* Include edge cases, failure paths, and boundary behavior relevant to the change.
* Do not weaken, delete, skip, or mark tests as expected failures merely to make CI pass.

</quality_gate>

</mandatory_constraints>

<architecture priority="critical">

<shared_anthropic_logic>

Place provider-neutral Anthropic protocol logic in:

```text
src/free_claude_code/core/anthropic/
```

Providers MUST NOT import shared protocol utilities from another provider package.

Shared protocol behavior must have a neutral owner.

</shared_anthropic_logic>

<failure_ownership>

Maintain the following ownership boundaries:

* `core/` owns canonical failure semantics, domain-level failure representation, and SDK-independent redaction.
* Providers own SDK-specific and HTTP-specific failure classification.
* Providers own provider retry behavior.
* Protocol and API adapters own wire-level error types.
* Protocol and API adapters own commit-boundary serialization.

Do not leak SDK-specific exception types into core domain semantics.

</failure_ownership>

<abstraction>

* Apply DRY principles to meaningful duplicated behavior.
* Extract shared abstractions only when they have a coherent owner and stable responsibility.
* Prefer composition over copy-paste.
* Do not introduce speculative abstractions for one implementation.
* Keep base classes small and behaviorally cohesive.
* Do not place provider-specific fields in a shared base configuration.

Example:

```text
nim_settings
```

belongs in the relevant provider constructor or provider-specific configuration, not `ProviderConfig`.

</abstraction>

<encapsulation>

* Mutate internal state through methods owned by the object.
* Do not assign another object's private attributes directly.

Prefer:

```python
task_manager.set_current_task(task)
```

over:

```python
task_manager._current_task = task
```

</encapsulation>

<reasoning_behavior>

* Resolve client reasoning intent once at the application boundary.
* Provider adapters translate that normalized intent into documented provider capabilities.
* Do not branch on upstream model names, model versions, aliases, or naming patterns to select reasoning behavior.
* Capability decisions must be based on explicit configuration or documented provider capability data.

</reasoning_behavior>

<naming>

* Use platform-agnostic names in shared modules.
* Do not embed a specific transport or platform name in a general abstraction.

Prefer:

```text
PLATFORM_EDIT
```

over:

```text
TELEGRAM_EDIT
```

when the concept is shared across platforms.

</naming>

<migrations>

When moving or renaming modules:

1. Move ownership to the correct destination.
2. Update all first-party imports.
3. Update tests and fixtures.
4. Update configuration and documentation when applicable.
5. Remove obsolete modules and compatibility shims in the same change unless compatibility is explicitly required.
6. Search the repository for stale paths, names, and imports.

A migration is incomplete while both the old and new ownership structures remain without an explicit compatibility requirement.

</migrations>

<dead_code>

* Remove unused code, obsolete branches, abandoned compatibility layers, stale configuration, and hardcoded provider identifiers.
* Do not preserve dead code for hypothetical future use.
* Replace literals with existing configuration where the value represents runtime or provider state.

Prefer:

```python
settings.provider_type
```

over:

```python
"nvidia_nim"
```

</dead_code>

<performance>

* Use list accumulation plus `str.join()` for repeated string construction.
* Do not use repeated `+=` string concatenation inside substantial loops.
* Prefer iterative implementations where recursion depth may depend on external or unbounded input.
* Cache immutable initialization-time values when repeated computation or environment access is unnecessary.
* Do not sacrifice clarity for unmeasured micro-optimizations.

</performance>

</architecture>

<implementation_standards>

<simplicity>

* Write the simplest implementation that correctly satisfies the requirement.
* Keep functions and classes focused.
* Prefer explicit control flow over clever compression.
* Avoid unnecessary dependencies.
* Avoid duplicate sources of truth.
* Reuse existing repository abstractions when they are appropriate.
* Do not force reuse when the abstraction has the wrong responsibility.

</simplicity>

<correctness>

* Validate assumptions against code, tests, configuration, and runtime behavior.
* Handle error paths deliberately.
* Preserve atomicity at commit boundaries.
* Maintain redaction and privacy guarantees.
* Avoid partial state updates when an operation can fail midway.
* Keep retry behavior idempotent where possible.
* Do not catch broad exceptions unless the boundary genuinely owns all failures and preserves diagnostic context.

</correctness>

<testing>

Tests should cover the maximum practical behavior surface, including:

* Normal behavior.
* Boundary inputs.
* Invalid inputs.
* Failure classification.
* Retry behavior.
* State transitions.
* Serialization boundaries.
* Redaction behavior.
* Provider translation behavior.
* Regression cases.
* Relevant integration and smoke paths.

Prefer live smoke tests where they provide meaningful early detection and can run safely and deterministically.

Do not require live external services in the default unit-test suite unless the repository explicitly treats them as required CI dependencies.

</testing>

</implementation_standards>

<workflow priority="critical">

<step id="1" name="analyze">

Before editing:

1. Read the relevant source files.
2. Read directly related tests.
3. Trace callers and downstream consumers.
4. Inspect configuration and environment contracts.
5. Search for duplicated logic, legacy paths, and existing utilities.
6. Reproduce the bug when applicable.
7. Identify whether the requested change affects production files and therefore requires versioning.

Do not propose a fix based solely on an error message or isolated snippet when repository context is available.

</step>

<step id="2" name="plan">

Create an implementation plan that:

* Identifies the root cause or required behavior.
* Lists affected modules and ownership boundaries.
* Orders changes by dependency.
* Identifies tests to add or update.
* Determines whether a semantic version bump is required.
* Identifies required verification commands.
* Notes migration or compatibility implications.

Keep the plan proportional to the task.

</step>

<step id="3" name="execute">

During implementation:

* Fix the cause, not only the observed symptom.
* Make changes incrementally.
* Keep each edit internally coherent.
* Propagate changes to all affected imports, call sites, tests, configuration, and documentation.
* Remove obsolete implementation in the same change.
* Preserve unrelated behavior.
* Add or update tests alongside implementation.
* Do not claim to create commits unless repository access and explicit authorization permit commits.

</step>

<step id="4" name="verify">

Run focused checks while iterating, then run the complete local CI sequence before completion.

Preferred macOS/Linux command:

```sh
./scripts/ci.sh
```

Preferred Windows command:

```powershell
.\scripts\ci.ps1
```

The scripts require `uv` on `PATH`.

The local CI scripts run Ruff in repair mode before type checking and tests:

1. `ruff format`
2. `ruff check --fix`
3. `ty check`
4. `pytest`

Use subset controls when iterating:

```sh
./scripts/ci.sh --only <check>
./scripts/ci.sh --skip <check>
./scripts/ci.sh --dry-run
```

PowerShell equivalents:

```powershell
.\scripts\ci.ps1 -Only <check>
.\scripts\ci.ps1 -Skip <check>
.\scripts\ci.ps1 -DryRun
```

When debugging individual failures, use:

```sh
uv run ruff format
uv run ruff check --fix
uv run ty check
uv run pytest -v --tb=short
```

When verifying enforcement behavior matching GitHub CI, use:

```sh
uv run ruff format --check
uv run ruff check
```

Also run relevant smoke tests when the changed behavior crosses runtime, provider, API, CLI, installation, or integration boundaries.

Verification is incomplete until failures caused by the change are resolved.

</step>

<step id="5" name="inspect_diff">

Before completion:

1. Review the full diff.
2. Remove accidental edits.
3. Search for stale symbols and paths.
4. Confirm tests validate the intended behavior rather than implementation details alone.
5. Confirm no suppressions were introduced.
6. Confirm no production change is missing its required version bump.
7. Confirm `uv.lock` is synchronized when required.
8. Confirm both directive files remain identical when either was edited.

</step>

<step id="6" name="report">

Report:

* Files changed.
* Logic altered.
* Verification commands and outcomes.
* Residual risks.
* Any checks that could not be run and the exact reason.

Do not report a check as passing unless it was executed successfully.

</step>

</workflow>

<task_decision_rules>

<bug_fix>

For a bug fix:

1. Reproduce or precisely trace the defect.
2. Identify the root cause.
3. Add a regression test that fails before the fix when practical.
4. Implement the smallest root-cause correction.
5. Verify the regression test and relevant surrounding tests.
6. Check for the same defect pattern elsewhere.

</bug_fix>

<new_feature>

For a new feature:

1. Define observable behavior and boundaries.
2. Add tests before or alongside implementation.
3. Implement the minimum complete capability.
4. Preserve backward compatibility unless breaking behavior is explicitly required.
5. Add configuration and documentation only where the new behavior requires them.
6. Classify the required semantic version bump.

</new_feature>

<refactor>

For a refactor:

1. Preserve observable behavior.
2. Establish sufficient test coverage before risky structural changes.
3. Move ownership completely.
4. Remove replaced code.
5. Avoid adding compatibility layers unless explicitly required.
6. Use a PATCH version bump when production files change without user-visible capability changes.

</refactor>

<migration>

For a migration:

1. Identify the canonical new owner.
2. Update all producers and consumers.
3. Remove the old owner.
4. Search for stale names and paths.
5. Verify packaging and import behavior.
6. Document user action only when the migration is externally visible.

</migration>

</task_decision_rules>

<ci_contract priority="critical">

The repository has five required CI check categories:

1. Suppression and legacy-annotation enforcement.
2. Ruff formatting.
3. Ruff linting.
4. `ty` type checking.
5. Pytest.

They are represented in:

```text
scripts/ci.sh
scripts/ci.ps1
```

They are enforced by `tests.yml` using parallel jobs.

GitHub CI runs on:

```text
push
pull_request
merge_group
```

GitHub CI uses check-only Ruff commands:

```sh
uv run ruff format --check
uv run ruff check
```

Required status checks should include all exact statuses displayed by GitHub, such as:

```text
Ban suppressions and legacy annotations
ruff-format
ruff-check
ty
pytest
```

The displayed names may be prefixed with:

```text
CI /
```

Use the exact labels displayed in repository ruleset configuration.

Remove the obsolete required status named `ci` when it refers to the previous aggregate gate job.

</ci_contract>

<repository_protection>

Repository protection should use GitHub rulesets.

<main_integrity_ruleset>

The primary `main` integrity ruleset should:

* Be non-bypassable.
* Require pull requests.
* Require the merge queue.
* Require all mandatory status checks.
* Block direct pushes to `main`.
* Block force pushes to `main`.

</main_integrity_ruleset>

<review_ruleset>

A separate review-focused ruleset MAY permit `Alishahryar1` or administrators to bypass review requirements only.

A review bypass must not bypass the main integrity ruleset, required checks, merge queue, direct-push restrictions, or force-push restrictions.

</review_ruleset>

</repository_protection>

<versioning priority="critical">

<rule>

Every commit on `main` that changes a production file MUST include a semantic version bump in `[project].version` in `pyproject.toml` in the same commit.

Do not merge or push production changes without the required version and lockfile updates.

</rule>

<production_files>

Changes to these paths count as production changes:

```text
src/free_claude_code/api/
src/free_claude_code/application/
src/free_claude_code/cli/
src/free_claude_code/config/
src/free_claude_code/core/
src/free_claude_code/messaging/
src/free_claude_code/providers/
.env.example
pyproject.toml
scripts/install.sh
scripts/install.ps1
scripts/uninstall.sh
scripts/uninstall.ps1
scripts/ci.sh
scripts/ci.ps1
```

`pyproject.toml` counts as a production file when changes affect dependencies, scripts, packaging, runtime configuration, or the installation surface.

</production_files>

<non_production_files>

These paths do not require a version bump when changed alone:

```text
tests/
smoke/
README.md
assets/
AGENTS.md
CLAUDE.md
.github/
.gitignore
```

When one commit contains both production and non-production changes, the production-file rule applies.

</non_production_files>

<semver>

Use:

```text
MAJOR.MINOR.PATCH
```

Choose the bump as follows:

<patch>

Use PATCH for:

* Bug fixes.
* Internal refactors without user-visible behavior changes.
* Dependency updates.
* Packaging fixes.
* Installation fixes.
* Internal architecture corrections.
* Performance fixes that preserve public behavior.

Example:

```text
1.2.38 -> 1.2.39
```

</patch>

<minor>

Use MINOR for backward-compatible capabilities, including:

* New providers.
* New administrative fields.
* New CLI commands.
* New configuration options.
* New optional behavior.
* Additive API functionality.

Example:

```text
1.2.38 -> 1.3.0
```

</minor>

<major>

Use MAJOR for breaking changes, including:

* Removed or renamed environment variables.
* Incompatible API changes.
* Incompatible CLI changes.
* Incompatible default behavior.
* Removed published interfaces.
* Required migrations or user action.

Example:

```text
1.2.38 -> 2.0.0
```

</major>

When uncertain:

* Prefer PATCH for corrections and behavior-preserving work.
* Prefer MINOR for new backward-compatible capability.
* Use MAJOR only when users must adapt.

</semver>

<required_versioning_steps>

When a production file changes on `main`:

1. Classify the change.

2. Select PATCH, MINOR, or MAJOR.

3. Update `[project].version` in `pyproject.toml`.

4. Run:

   ```sh
   uv lock
   ```

5. Verify `uv.lock` reflects the package version.

6. Include implementation, version, and lockfile changes in the same commit.

</required_versioning_steps>

</versioning>

<tools>

* Prefer available built-in repository inspection and editing tools.
* Check tool availability before assuming a command or integration exists.
* Prefer targeted file reads and repository search over manually scanning unrelated files.
* Use shell commands when they are the most reliable way to inspect or verify repository state.
* Do not invent tool output.
* Do not claim that files, tests, commits, branches, pull requests, or external resources were changed unless the action was actually performed.

</tools>

<completion_contract priority="critical">

A task is complete only when all applicable conditions are satisfied:

* The requested behavior is implemented.
* The root cause is addressed.
* Affected code paths are updated.
* Relevant tests are added or updated.
* Required checks pass.
* Relevant smoke tests pass or any inability to run them is explicitly reported.
* No forbidden suppressions were introduced.
* No stale migration paths remain.
* Required versioning is complete.
* `uv.lock` is synchronized when required.
* `AGENTS.md` and `CLAUDE.md` remain identical.
* The final report is accurate and granular.

</completion_contract>

<response_contract>

Use the following final summary structure:

## Files Changed

List each changed file and its purpose.

## Logic Altered

Describe:

* The root cause or requested behavior.
* The implementation approach.
* Important ownership or architectural changes.
* Relevant edge cases.

## Verification Method

List the exact commands executed and their results.

Distinguish between:

* Passed checks.
* Failed checks.
* Checks not run.

Do not say that CI passed when the full required CI sequence was not run successfully.

## Residual Risks

List remaining risks, limitations, unverified external behavior, or follow-up work.

When there are no known residual risks, state:

```text
None.
```

</response_contract>

</agent_directive>
