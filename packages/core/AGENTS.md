Build from root: pnpm build:core
Test from root: pnpm test:core
Typecheck from root: pnpm --filter ./packages/core check
If focused core Vitest runs fail to resolve @internal/test-utils/setup, run pnpm build:core first so internal workspace build artifacts are available
If you change Zod compatibility behavior, also run pnpm test:core:zod and pnpm --filter ./packages/core typecheck:zod-compat

Most tests live under packages/core/src/
Run focused processor, harness, agent, or loop tests before broader validation when those areas change

Keep changes here surgical; many packages depend on core

Mastra exposes a per-run scratch space (`runScope`) keyed by `runId` for non-serializable runtime state (MessageList, processor states, converted tools, loop options). Access it via `mastra.__createRunScope(runId)` / `__getRunScope(runId)` and typed `RunScopeKey<T>` keys from `mastra/run-scope.ts`. It is refcounted alongside `__registerInternalWorkflow`, never persisted, never published over pubsub, and dies with the run. Do not put runScope values on step input/output schemas — those cross the wire and must stay JSON-safe (Date/Error/Map/Set/GeneratedFile are handled by the codec at the `UnixSocketPubSub` boundary; live handles and closures are not).

## Conventions

Repo-wide rules (logging, errors, tests, PR hygiene) are in the root `AGENTS.md` § Review conventions; this section holds only the core contracts on top of them. Enforced by hand in review and not inferable from the code. Each line ends with the PRs it was mined from.

Streaming, processors, retry and abort:

- Withhold a rejected retry attempt's text from the final output; blanking a tripwired step's `text` is the intended contract, not a bug. (PR #24476, #24443, #24444)
- Scope abort to the segment, not the run id; never widen an abort check to reject every enqueue for that run. (PR #24479, #24174, #23696)
- Keep `MastraFinishReason` distinct: `tripwire` means a processor rejected the output, `aborted` means the caller stopped the run; never collapse one into another. (PR #24476, #23969, #24444)
- Make every processor hook and output accessor agree on what `result.text` contains; one accessor must never contradict another about the same result. (PR #24476, #24444, #24059)
- Express cancellation with a standard `AbortSignal` or `readable.cancel()` and a typed abort error; never a bespoke stop method, never a tripwire. (PR #24241, #24451, #23969)
- Never mutate or reorder existing history or system messages; append a new user message at the end so the provider prompt cache survives. (PR #15416, #14232, #22481, #18653)
- Build built-in behaviour on the same public registry an external author would use; no hardcoded special cases, no internal-only APIs. (PR #16459, #22177, #18384)
- Never add a guard that masks an upstream bug, and never fail silently; surface the error or abort the stream. (PR #19183, #18639, #14653, #23696)
- Bound every iteration, collection and query with a named limit, and give each network or subprocess call a default timeout. (PR #20579, #23524, #19135, #18908)

API and compatibility:

- Treat a published `.d.ts` field and every package export as public API; narrowing or removing one is breaking even if unreleased. (PR #23675, #20138, #17896, #12020)
- Make a new field on a shared type optional and nullable for older peers; needing a new core API means bumping peerDeps. (PR #20677, #12295, #12687)
- Do not widen a contract or change a default inside a bugfix; new behaviour is opt-in and refactors get their own PR. (PR #22872, #22878, #16922, #13634)
- State an option's enforcement exceptions and unsupported backends in its docs and types, not only the happy contract. (PR #18999, #24336, #23033)
