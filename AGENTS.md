# Repository guidance

Unless asked, don't inspect reference or modify examples.
Use the most-specific `AGENTS.md`; for package work, read `packages/<name>/AGENTS.md` first.

Turborepo pnpm workspace; packages use strict TypeScript; Vitest tests are colocated.
For schema-backed execution, put deterministic input/output constraints (shape, types, ranges, formats, cross-field invariants) in schemas, not `execute`. Keep runtime/external checks (authorization, existence, conflicts, API responses) in `execute`.
Use literal model names/IDs from `docs/src/plugins/remark-model-tokens/models.ts` in changesets/comments; placeholders are not replaced.

Use the narrowest package build/test/lint/typecheck; run unit/integration before E2E. Prefer targeted `pnpm --filter` or `pnpm turbo build --filter` commands; avoid root setup/build scripts unless needed. Fresh clone: `pnpm install`, then build relevant dependencies. Unresolved workspace imports usually mean dependencies need building; some integration tests need `pnpm i --ignore-workspace`.

Features/new packages need docs. For docs, follow `docs/AGENTS.md` and styleguides. After code changes, read `@.mastracode/commands/changeset.md`.

Architecture: `packages/core/src`; `mastra/` config/DI; `agent/`, `tools/`, `memory/`, `workflows/`, `storage/` are modular framework components.

Read applicable `@.claude/commands/`: `changeset`, `commit`, `gh-new-pr`, `gh-pr-comments`, `make-moves`.
Read applicable `@.claude/skills/`: `playground-msw-tests` (primary for playground/UI), `e2e-tests-studio` (secondary), `mastra-docs`, `react-best-practices`, `tailwind-v4`, `mastra-frontend`, `mastra-smoke-test`, `smoke-test`.

## Review conventions

Maintainers enforce these by hand on every PR. Every rule lives in exactly one place: repo-wide rules here, package contracts in that package's `AGENTS.md`, and `.coderabbit.yaml` `path_instructions` only restate the one or two that a reviewer needs while reading that directory's diff. Each line ends with the PRs it was mined from.

Errors and logging:

- Log through the registered Mastra logger, never `console.*`; reserve `error` for terminal exhaustion and `warn` for a recovered attempt. (PR #24437, #24441, #16507)
- Preserve the original `MastraError` across processor, tripwire and stream boundaries, keeping `name`, `message` and `stack` as plain fields, never a blob. (PR #24444, #23969, #19739, #24150)
- Return the status that means what it says — 429 for quota, 404 for unrouted — never 200 with an error body. (PR #21160, #20579, #24154)
- Catch and log every fire-and-forget promise through the instance logger, and let spawn or child errors surface instead of being swallowed. (PR #16309, #16482, #20328)
- Fail closed in credential and tenant paths: no process-global or env fallback, and reject cross-tenant references before any provider request. (PR #19638, #19921, #19577, #20579)

Tests (Vitest, colocated with the source):

- Prove a regression test load-bearing by stating it fails on `main` or with the fix reverted, and review harnesses as production code. (PR #20523, #19940, #24344, #24368)
- Never delete or loosen an existing assertion to make a suggestion fit; a pinned assertion is the contract under review. (PR #24476, #14843, #22177)
- Gate an async assertion on a real readiness event or `vi.waitFor`, never on a fixed sleep or timer. (PR #20926, #22942, #23696)
- Name a test for exactly what it asserts, make the fixture reach that path, and assert the observable outcome. (PR #19225, #18999, #23990, #14843)
- Restore global state: env in a `finally`, and `afterEach` calling `vi.unstubAllEnvs()`, `vi.unstubAllGlobals()` and `vi.restoreAllMocks()`. (PR #22394, #23327, #22888)
- Derive test types from the exported type, route handler or `satisfies`; never `as any` on a fixture or input. (PR #19299, #19225, #21432, #22558)

Pull requests:

- Keep a PR to one logical change; unrelated edits get their own PR and pre-existing defects get their own issue. (PR #22558, #23466, #19429, #22559)
- Decline a finding with a technical rationale that cites `main` (file:line or "pre-existing on main") and names the follow-up. (PR #14453, #20471, #22960, #20261)
- Reply to every bot and human review thread before merge, and keep the PR body describing what actually ships. (PR #17710, #19701, #20848)
- Open no PR while its feature issue carries `status: needs triage` or `status: needs approval`; it is auto-closed. (`CONTRIBUTING.md`; PR #24471, #24282)
- Carry no changeset on a test-only PR, and omit the usage example when the option is accepted-but-ignored. (PR #20523, #20473, #23978, #24275)

Formatting is oxfmt, not prettier: `pnpm lint:format` runs `oxfmt --check` over `.ts`, `.md`, `.yaml` and more; `pnpm oxfmt:changed` formats what you touched.
