# spec-kit-version-guard

A [Spec Kit](https://github.com/github/spec-kit) extension that verifies locked tech stack versions against live registries before planning and implementation, ensuring LLM agents use up-to-date API patterns instead of stale training data.

## Problem

LLM agents generate code based on training data that may be months behind the latest library versions. When your project locks a dependency version newer than the model's training cutoff:

- Deprecated APIs get used instead of their replacements
- New features and patterns are missed
- Generated code may not compile or may use anti-patterns
- Silent regressions when API signatures have changed

## Solution

The Version Guard extension fires before `/speckit.plan` and `/speckit.implement`, fetching the latest stable versions from npm and comparing them against your locked versions. For any flagged packages, it fetches official migration guides and changelogs so the agent uses real documentation — not training data — for code generation.

## Installation

```bash
# From release
specify extension add version-guard --from \
  https://github.com/KevinBrown5280/spec-kit-version-guard/archive/refs/tags/v1.0.0.zip

# From main branch
specify extension add version-guard --from \
  https://github.com/KevinBrown5280/spec-kit-version-guard/archive/refs/heads/main.zip

# Development mode (local clone)
specify extension add --dev /path/to/spec-kit-version-guard
```

## Commands

| Command | Description | Modifies Files? |
|---------|-------------|-----------------|
| `speckit.version-guard.check` | Check locked tech stack versions against live npm registry and official docs | No — read-only |

## How It Works

1. **Locate the tech stack decision record**: Checks for `docs/reference/tech-stack-decision-record.md`, then falls back to `package.json` files at the repo root, `frontend/`, and `backend/`.

2. **Fetch latest versions**: For each locked dependency, fetches `https://registry.npmjs.org/{package}/latest` and compares:

   | Package | Locked | Latest Stable | Status |
   |---------|--------|---------------|--------|
   | react   | 18.3.1 | 19.x.x       | ⚠️ Behind |
   | vite    | 6.0.0  | 6.2.x        | ✅ Current |

3. **Fetch migration guides**: For packages where the locked version may be newer than training data, fetches official changelogs:
   - React: `https://react.dev/blog`
   - Vite: `https://vite.dev/blog`
   - Azure SDKs: Microsoft docs MCP tools
   - Others: GitHub releases page or CHANGELOG.md

4. **Output report**: Summarizes package status, API changes, and specific warnings (e.g., "React 19 replaces `useFormStatus` with `useActionState`").

5. **Non-blocking**: The report is informational — the workflow continues regardless.

## Hooks

| Hook | Command | Description |
|------|---------|-------------|
| `before_plan` | `speckit.version-guard.check` | Verify versions before planning |
| `before_implement` | `speckit.version-guard.check` | Verify versions before code generation |

## Graceful Degradation

- Network failures or rate limiting: warns and skips
- No `package.json` or tech stack record: skips silently
- All packages current: outputs "✅ All locked versions are current" and proceeds

## Requirements

- Spec Kit >= 0.2.0

## License

MIT
