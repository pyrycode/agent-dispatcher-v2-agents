# Agent Dispatcher v2 — Driver Repo

Driver repo that dispatches agents to build [`pyrycode/agent-dispatcher-v2`](https://github.com/pyrycode/agent-dispatcher-v2). The dispatcher itself runs from `dispatcher/` (git submodule pinned to a `pyrycode/agent-dispatcher` v1 SHA).

This is the same shape as `pyrycode/agents` — per-agent CLAUDE.md prompts (po/, architect/, developer/, code-review/, documentation/), `bin/` launcher scripts, `.env` config — but pointed at the v2 source repo instead of the Go daemon.

## Status: ported, not yet v2-ified

The 5 agent CLAUDE.md files are direct ports from `pyrycode/agents` as of 2026-05-10. They contain Go-specific examples and patterns (e.g. `go test -race`, `go vet`, references to "the Go daemon") that don't apply to the TypeScript v2 target. **An early ticket will audit and adapt** — for now, the agents will see Go references in their prompts but build commands that resolve via the dispatcher's `SALVAGE_GATES` env var (TS-flavored: `pnpm typecheck;pnpm test`).

The valuable accumulated lessons (sizing rules, anti-rationalization, file-overlap checks, salvage flow, predicate-completeness, label-shape audits, etc.) all transfer unchanged.

## Target build commands

| Operation | Command (run in `agent-dispatcher-v2/`) |
|---|---|
| Typecheck | `pnpm typecheck` |
| Tests | `pnpm test` |
| Lint | `pnpm lint` |
| Format | `pnpm format` |

`SALVAGE_GATES` in `.env` defaults to `pnpm typecheck;pnpm test` for the v2 target.

## Operator interface

Run from this repo's root:

- `bin/pyry-start` — launch dispatcher (loads `.env`, syncs submodule, exports `AGENTS_REPO_PATH`, holds a per-target lock at `<TARGET_REPO_PATH>/.dispatcher.lock`)
- `bin/pyry-drain` — graceful stop
- `bin/pyry-restart` — drain + start
- `bin/pyry-status` — is the dispatcher running?
- `bin/pyry-logs` — tail the live dispatch log
- `bin/pyry-test` / `bin/pyry-typecheck` — run dispatcher tests / tsc (against the submodule, not the v2 target)

## Important — read me before editing prompts

Three rules that survived the v1 → v2 lessons synthesis. They override any phrase-matching in the ported CLAUDE.md files.

### 1. Belt-and-suspenders means different fabric

Every "agent does X" rule needs a deterministic dispatcher-side safety net for X. Two stochastic rules verifying each other share the same failure mode. When adding a new agent rule, ask: *"what deterministic check enforces this if the agent forgets?"*

### 2. Labels are the truth, prose is decoration

When the agent must signal something to the dispatcher, the canonical signal is the label — not a comment, not a PR body section. Agents that update only prose silently bypass the dispatcher's interpretation. Mandatory: every state transition the dispatcher reads has a corresponding label-write requirement.

### 3. Strict rules over precise rules

Stochastic agents rationalize past threshold rules ("this counts as one logical change"). Absolute rules age better. The v2 target ships **XS / S only — no M, no "Why M, not split" paragraph**. Splits are the architect's exclusive call.

## Pipeline shape

Same as v1: `Inbox → Backlog → In Architecture → In Development → In Code Review → In Documentation → Done`.

- **Inbox** is human-gated. Anyone creates issues there. The human promotes to Backlog when ready for PO.
- **PO** refines (does not create from raw requests).
- **Architect** writes specs to `docs/specs/architecture/{ticket}-{name}.md` AND splits oversized tickets back to PO via `needs-rework:po`.
- **Developer** ships code + tests, opens a PR.
- **Code Review** posts findings; failing review = `needs-rework:developer` label (the canonical signal — prose is decorative).
- **Documentation** synthesizes into `docs/knowledge/`.

## Self-hosting roadmap

The v2 source repo cannot dispatch its own development on day 1 (no salvage, no streaming, no GH client). The plan:

1. **Phase 0 (now):** v1 dispatcher (this submodule) drives v2 development
2. **Phase 1:** ~ticket #15 — v2 has enough core to run a sandbox dispatch against a toy repo
3. **Phase 2:** ~ticket #25 — v2 self-hosts (this `dispatcher/` submodule flips to point at `pyrycode/agent-dispatcher-v2`)

Switching the submodule pointer in step 2 is the cutover moment. v1 stays around as the fallback runtime.

## See also

- v2 target source: [`pyrycode/agent-dispatcher-v2`](https://github.com/pyrycode/agent-dispatcher-v2)
- v1 dispatcher (current submodule): [`pyrycode/agent-dispatcher`](https://github.com/pyrycode/agent-dispatcher)
- v1 driver (reference for the port): `/Users/juhanailmoniemi/WorkSpace/Projects/pyrycode/agents/`
