# agent-dispatcher-v2-agents

Driver repo for [`pyrycode/agent-dispatcher-v2`](https://github.com/pyrycode/agent-dispatcher-v2) development. Holds:

- Per-agent CLAUDE.md prompts (`po/`, `architect/`, `developer/`, `code-review/`, `documentation/`)
- `bin/` launcher scripts
- `.env` config (gitignored — see `.env.example`)
- `dispatcher/` git submodule pinned to a v1 [`pyrycode/agent-dispatcher`](https://github.com/pyrycode/agent-dispatcher) SHA

The submodule will eventually flip to point at `pyrycode/agent-dispatcher-v2` once v2 is far enough along to self-host (planned around ticket #25 — see `CLAUDE.md`).

## Quick start

```bash
cp .env.example .env
$EDITOR .env  # fill in GITHUB_TOKEN, PROJECT_NUMBER

git submodule update --init  # pull dispatcher source

bin/pyry-start  # launch the dispatcher
```

See `CLAUDE.md` for full operating instructions.

## License

Apache-2.0.
