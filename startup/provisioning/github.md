# Provisioning: GitHub Repository

**Idempotency:** Check if repo exists before creating.

---

## Step 1: Initialize git

```bash
ls .git 2>/dev/null && echo "GIT_EXISTS" || echo "GIT_MISSING"
```

If missing:
```bash
git init
```

## Step 2: Check if GitHub repo exists

```bash
gh repo view {org}/{slug} --json name 2>/dev/null && echo "REPO_EXISTS" || echo "REPO_MISSING"
```

## Step 3: Create GitHub repo

If missing:
```bash
gh repo create {org}/{slug} --{visibility} --source . --push
```

If the user specified a personal account (no org), use:
```bash
gh repo create {slug} --{visibility} --source . --push
```

**Note:** `--source .` tells `gh` to use the current directory. `--push` does the initial push.

## Step 4: Set default branch protections (optional)

For production repos, consider:
```bash
# Skip for now — can be configured later via /ship
```

## Output

Log:
- Git: initialized / already existed
- GitHub repo: created at https://github.com/{org}/{slug} / already existed
