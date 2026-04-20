---
description: "Verify tech stack versions against live registries before code generation"
---

# Version Guard — Tech Stack Verification

Verify that locked library versions are current and that code generation will use
up-to-date API patterns. This command fires automatically before `/speckit.plan`
and `/speckit.implement` via hooks.

## Execution Steps

1. **Locate the tech stack decision record**:
   - Check for `docs/reference/tech-stack-decision-record.md` in the repo root
   - If not found, check `package.json` files at the repo root, `frontend/`, and `backend/` for dependency versions
   - If neither exists, output a warning and skip (do not block the workflow)

2. **For each locked dependency**, fetch the latest stable version from the npm registry:
   - Fetch `https://registry.npmjs.org/{package}/latest`
   - Compare the locked major version against the latest stable major version
   - Report the result in a table:

   | Package | Locked | Latest Stable | Status |
   |---------|--------|---------------|--------|
   | react   | 18.3.1 | 19.x.x       | ⚠️ Behind |
   | vite    | 6.0.0  | 6.2.x        | ✅ Current |

3. **For any package where the locked version may be newer than your training data**:
   - Fetch the official migration guide or changelog:
     - React: `https://react.dev/blog`
     - Vite: `https://vite.dev/blog`
     - Azure SDKs: use Microsoft docs MCP tools
     - Other packages: fetch the GitHub releases page or CHANGELOG.md from the repo
   - Summarize key API changes, new patterns, and deprecated features in 3-5 bullets

4. **Output a version guard report** summarizing:
   - Packages checked and their status
   - Any API patterns that differ from training-data-era defaults
   - Specific warnings (e.g., "React 19 replaces `useFormStatus` with `useActionState`")

5. **Do NOT block the workflow**. The report is informational — the agent should use
   the fetched documentation (not training data) for any flagged packages during
   subsequent plan/implement steps.

## Graceful Degradation

- If `web_fetch` or registry calls fail (network issues, rate limiting): warn and skip
- If no `package.json` or tech stack record exists: skip silently
- If all packages are current: output "✅ All locked versions are current" and proceed
