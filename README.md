# aim-policy

An **org policy** repo for [`aim`](https://github.com/JasperHG90/agent-integrations-manager), the package manager for AI-assistant tooling (skills, sub-agents, MCP servers, and rules).

This repo holds a single, self-contained [`policy.toml`](policy.toml) that governs which repos and artifacts a project may install, which layout profiles are allowed, and how artifact content is risk-scanned. Projects bind to it so the whole team enforces one mandated policy, pinned and CI-enforced.

## What's in `policy.toml`

| Section | Purpose |
| --- | --- |
| `[repos]` | Block source repos by normalized URL or alias. |
| `[artifacts]` | Block specific skills / agents / rules / MCP servers. |
| `[profiles]` | Allow-list of layout profiles (`claude`, `gemini`). |
| `[archetypes]` | Allow-list of instruction archetypes (empty = all). |
| `[risk]` | Risk-scanning config: local ONNX screen + DSPy LLM judge. |
| `[[rule]]` | Custom risk rules the judge evaluates artifacts against. |

The current policy runs both the local injection/jailbreak `classifier` and the `llm_judge` (`gemini/gemini-flash-lite-latest`) in `block` mode at a `high` threshold, with overrides allowed.

## Using this policy

Bind a project to this repo so `aim` resolves the org policy instead of a local one:

```sh
aim policy bind https://github.com/JasperHG90/aim-policy
```

This writes `[policy] scope = "org"` into the project's `aim.toml` and pins the repo URL, resolved commit SHA, and a content hash into `aim.lock.toml`. Refresh after the policy changes upstream:

```sh
aim policy refresh
```

### CI enforcement

The real enforcement boundary is code review + CI on the committed lockfile. Validate a project against the mandated policy in CI — it fetches this policy fresh and fails the build on any violation:

```sh
aim policy validate --policy https://github.com/JasperHG90/aim-policy
```

## Editing the policy

`policy.toml` is the single source of truth. Edit it, commit, and bound projects pick up the change on their next `aim policy refresh` (or automatically within the 24h cache TTL). Keep it self-contained — `aim` expects one `policy.toml` with the same sections as an inline `[policy]` table.

See the [`aim` governance docs](https://github.com/JasperHG90/agent-integrations-manager#governance--risk-scanning) for the full policy schema and risk-scanning details.
