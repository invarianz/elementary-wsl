# Claude Code Project Configuration

## Git Identity

Always commit with the following identity — never use any other author:

- **Name:** invarianz
- **Email:** invarianztheorem@posteo.com

When creating commits, use:
```
git -c user.name="invarianz" -c user.email="invarianztheorem@posteo.com" commit ...
```

Never include Claude session links (e.g., `https://claude.ai/...`) in commit messages.

---

## Command: `sanity check`

Triggered when the user writes **"sanity check"**.

### Phase 1 — Analysis

Scan the codebase for each category below. Collect findings into a single list — do not fix anything yet.

1. **Dead code** — unused files, configs, or scripts.
2. **Duplicate / verbose code** — copy-pasted or near-identical logic that could be consolidated.
3. **TODO/FIXME/HACK comments** — leftover markers that may need attention or removal.
4. **Security** — hardcoded secrets, insecure file permissions, silently swallowed errors that hide failures.
5. **CI/CD issues** — workflow bugs, missing error handling, race conditions, incorrect triggers or permissions.

### Phase 2 — Fix

6. **Apply all fixes** found in Phase 1. Remove dead code outright — do not comment it out or add `// removed` markers.

### Report

Present a summary table of what was found and what was changed, grouped by category.
