---
name: proof
version: 1.0.0
description: |
  Anchor agent decisions on the MultiversX blockchain before and after execution.
  Two-phase audit trail: certify the intent (WHY) before acting, certify the diff (WHAT) after.
  4W framework — WHO: Claude Code / ostack, WHAT: SHA-256 of decision or git diff,
  WHEN: immutable on-chain timestamp, WHY: action description and risk summary.
  Use before /ship, before any destructive operation (delete, major refactor, deploy),
  and after completing high-risk changes. Free trial: 10 certifications, no wallet needed.
  Proactively suggest before /ship and before any irreversible action.
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble (run first)

```bash
_UPD=$(~/.claude/skills/ostack/bin/ostack-update-check 2>/dev/null || .claude/skills/ostack/bin/ostack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.ostack/sessions
touch ~/.ostack/sessions/"$PPID"
_SESSIONS=$(find ~/.ostack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.ostack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/ostack/bin/ostack-config get ostack_contributor 2>/dev/null || true)
_PROACTIVE=$(~/.claude/skills/ostack/bin/ostack-config get proactive 2>/dev/null || echo "true")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "PROACTIVE: $_PROACTIVE"
source <(~/.claude/skills/ostack/bin/ostack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
```

If `PROACTIVE` is `"false"`, do not proactively suggest ostack skills — only invoke
them when the user explicitly asks. The user opted out of proactive suggestions.

If output shows `UPGRADE_AVAILABLE <old> <new>`: read `~/.claude/skills/ostack/ostack-upgrade/SKILL.md` and follow the "Inline upgrade flow" (auto-upgrade if configured, otherwise AskUserQuestion with 4 options, write snooze state if declined). If `JUST_UPGRADED <from> <to>`: tell user "Running ostack v{to} (just updated!)" and continue.

## AskUserQuestion Format

**ALWAYS follow this structure:**
1. **Recommendation:** "Do X because Y." One sentence. Position first.
2. **Options:** A) ... B) ... — two options, three max if genuinely needed. When an option involves effort, show both scales: `(human: ~X / CC: ~Y)`

Skip AskUserQuestion entirely if there's a clear right answer — just do it and report.

Never hedge. Never pad with "great question." Never restate context the user already has. Compress ruthlessly. Trust the user to keep up.

## Conviction and Completeness — Often Wrong, Never in Doubt

AI makes the marginal cost of completeness near-zero. Do the complete thing. But the deeper question is whether you're completing the **right** thing.

### Say No to 1000 Things
Completeness without focus is waste. Before going deep, ask: is this the highest-leverage thing to build right now? If not, stop. The power of AI-assisted development is not doing everything — it's doing the right thing completely and fast. Say no to everything else.

### Iteration as Truth
Rapid collision with reality beats analysis. Ship the smallest thing that tests the riskiest assumption. Wrong fast is better than right slow. Conviction means committing to a direction, shipping it, and course-correcting from real signal — not hedging with half-implementations.

### Power Law
Concentrate effort, don't diversify. One complete feature beats three half-finished ones. One well-tested path beats broad shallow coverage. Find the 20% of work that drives 80% of value and do that completely.

---

# xProof: Agent Decision Anchoring

Anchor what your agent *decided* and what it *did* — on the MultiversX blockchain.
Two API calls. Two proofs. One immutable audit trail anyone can verify at xproof.app.

**The rule:** Before a critical action → anchor the *intent* (WHY). After → anchor the *result* (WHAT).

---

## Setup

```bash
echo ${XPROOF_API_KEY:+"XPROOF_READY"}
```

If blank: tell the user to get a free API key at **xproof.app** (10 certifications, no wallet, no credit card)
and set `XPROOF_API_KEY=pm_...` in their shell environment.

If `XPROOF_READY`: proceed to Phase 1.

---

## Phase 1 — BEFORE: Certify the Intent (WHY)

Call this before any critical action. Anchors the decision and timestamps the intent on-chain.

**Step 1 — Capture context:**

```bash
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
_RECENT_LOG=$(git log --oneline -5 2>/dev/null || echo "no-git")
_DIFF_STAT=$(git diff --cached --stat 2>/dev/null || git diff HEAD --stat 2>/dev/null || echo "no-diff")
_TODOS=$(cat TODOS.md 2>/dev/null | head -20 || echo "")
_SESSION_ID="ostack-$(date -u +%Y%m%dT%H%M%SZ)-$$"
_TIMESTAMP=$(date -u +%Y-%m-%dT%H:%M:%SZ)
_OSTACK_VERSION=$(cat ~/.claude/skills/ostack/VERSION 2>/dev/null || cat .claude/skills/ostack/VERSION 2>/dev/null || echo "unknown")
```

**Step 2 — Build the inputs hash** (what you analyzed before deciding):

```bash
_INPUTS_RAW=$(printf '%s\n%s\n%s\n%s\n%s' \
  "$_BRANCH" \
  "$_RECENT_LOG" \
  "$_DIFF_STAT" \
  "$_TODOS" \
  "${ACTION_DESCRIPTION}")
_INPUTS_HASH=$(printf '%s' "$_INPUTS_RAW" | sha256sum | cut -d' ' -f1)
```

**Step 3 — Determine action_type** from what Claude Code is about to do:

| Situation | action_type |
|-----------|-------------|
| /ship, push, merge, deploy | `code_deploy` |
| delete files, drop table, rm -rf | `data_access` |
| refactor, rename, restructure | `other` |
| generate content, write docs | `content_generation` |
| call external API | `api_call` |

**Step 4 — Assess risk_level:**

| Signal | risk_level |
|--------|-----------|
| Config, docs, README only | `low` |
| Feature code, tests | `medium` |
| Auth, payments, migrations, infra | `high` |
| Production deploy, data deletion, irreversible | `critical` |

**Step 5 — Certify the intent:**

```bash
_RESPONSE=$(curl -s -X POST https://xproof.app/api/audit \
  -H "Authorization: Bearer $XPROOF_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"agent_id\": \"claude-code-ostack\",
    \"session_id\": \"$_SESSION_ID\",
    \"action_type\": \"${ACTION_TYPE}\",
    \"action_description\": \"${ACTION_DESCRIPTION}\",
    \"inputs_hash\": \"$_INPUTS_HASH\",
    \"inputs_manifest\": {
      \"fields\": [\"git_branch\", \"recent_commits\", \"diff_stat\", \"todos\", \"user_intent\"],
      \"sources\": [\"git\", \"TODOS.md\", \"user_prompt\"],
      \"hash_method\": \"SHA-256 over concatenated newline-delimited fields\"
    },
    \"risk_level\": \"${RISK_LEVEL}\",
    \"risk_summary\": \"${RISK_SUMMARY}\",
    \"decision\": \"approved\",
    \"timestamp\": \"$_TIMESTAMP\",
    \"context\": {
      \"branch\": \"$_BRANCH\",
      \"ostack_version\": \"$_OSTACK_VERSION\",
      \"agent\": \"claude-code\",
      \"tool\": \"ostack\"
    }
  }")

_PROOF_BEFORE=$(echo "$_RESPONSE" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('proof_id','ERROR'))" 2>/dev/null)
_AUDIT_URL=$(echo "$_RESPONSE" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('audit_url',''))" 2>/dev/null)
_TX=$(echo "$_RESPONSE" | python3 -c "import json,sys; d=json.load(sys.stdin); b=d.get('blockchain',{}); print(b.get('explorer_url',''))" 2>/dev/null)
```

**Step 6 — Store and report:**

```bash
mkdir -p ~/.ostack/proofs
echo "$_SESSION_ID $_PROOF_BEFORE $_AUDIT_URL" >> ~/.ostack/proofs/proof-log.txt
```

Tell the user:

```
✓ Intent anchored on MultiversX
  WHO:  claude-code-ostack
  WHAT: inputs_hash $_INPUTS_HASH
  WHEN: $_TIMESTAMP
  WHY:  ${ACTION_DESCRIPTION}

  proof_before: $_PROOF_BEFORE
  audit:        $_AUDIT_URL
  blockchain:   $_TX
```

If `_PROOF_BEFORE` is `ERROR`: show the raw `$_RESPONSE` and stop. Do not proceed with the action.

---

## Phase 2 — AFTER: Certify the Result (WHAT)

Call this after the action completes. Anchors the git diff — proof of what actually changed.

**Step 1 — Hash the diff:**

```bash
_DIFF=$(git diff HEAD~1 2>/dev/null || git diff 2>/dev/null || echo "no-diff")
_DIFF_HASH=$(printf '%s' "$_DIFF" | sha256sum | cut -d' ' -f1)
_DIFF_LINES=$(echo "$_DIFF" | wc -l | tr -d ' ')
```

**Step 2 — Certify the result:**

```bash
_RESPONSE_AFTER=$(curl -s -X POST https://xproof.app/api/proof \
  -H "Authorization: Bearer $XPROOF_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"file_hash\": \"$_DIFF_HASH\",
    \"filename\": \"git-diff-$_SESSION_ID.patch\",
    \"author_name\": \"claude-code-ostack\"
  }")

_PROOF_AFTER=$(echo "$_RESPONSE_AFTER" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('proof_id','ERROR'))" 2>/dev/null)
_VERIFY_URL=$(echo "$_RESPONSE_AFTER" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('verify_url',''))" 2>/dev/null)
```

**Step 3 — Report the complete audit trail:**

```
✓ Result anchored on MultiversX
  WHAT: git diff — $_DIFF_LINES lines changed
  hash: $_DIFF_HASH

  proof_after: $_PROOF_AFTER
  verify:      $_VERIFY_URL

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AUDIT TRAIL (immutable, publicly verifiable)
  Intent (WHY): $_AUDIT_URL
  Result (WHAT): $_VERIFY_URL
  Both proofs anchored on MultiversX blockchain.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Include both URLs in the PR description when used with `/ship`.

---

## Verify a Proof

```bash
curl -s https://xproof.app/api/proof/$PROOF_ID | python3 -c "
import json, sys
d = json.load(sys.stdin)
print('File:    ', d.get('fileName',''))
print('Hash:    ', d.get('fileHash',''))
print('Anchored:', d.get('createdAt',''))
b = d.get('blockchain', {})
print('Network: ', b.get('network', 'MultiversX'))
print('TX:      ', b.get('explorer_url',''))
print('Verify:  ', d.get('verify_url',''))
"
```

---

## MCP Integration (Native Claude Code)

If you prefer native tool calls over curl, configure xProof as an MCP server in `~/.claude.json`:

```json
{
  "mcpServers": {
    "xproof": {
      "type": "http",
      "url": "https://xproof.app/mcp",
      "headers": {
        "Authorization": "Bearer ${XPROOF_API_KEY}"
      }
    }
  }
}
```

Claude Code will then have access to:
- `mcp__xproof__audit_agent_session` — Phase 1: certify intent
- `mcp__xproof__certify_file` — Phase 2: certify result
- `mcp__xproof__verify_proof` — verify any existing proof
- `mcp__xproof__get_proof` — retrieve proof details as JSON or Markdown
- `mcp__xproof__investigate_proof` — reconstruct full audit trail for a disputed action

Schema: `https://xproof.app/.well-known/agent-audit-schema.json`

---

## /ship Integration

Insert `/proof` naturally before and after `/ship`:

```
/plan-eng-review              # architecture review
/proof "Ship feature X — reason: Y"    # anchor intent → proof_before
/ship                         # tests, PR, push
/proof --after                # anchor the diff → proof_after
                              # paste both URLs in PR body
```

The PR body becomes a tamper-proof receipt: anyone can verify what Claude Code decided to ship — and what it actually changed — directly on the MultiversX blockchain.

---

## Pricing

| Tier | Certifications | Cost |
|------|---------------|------|
| Trial | 10 free | No wallet, no card |
| Credits | 100 | $5 USDC on Base |
| Credits | 1,000 | $40 USDC on Base |
| Credits | 10,000 | $300 USDC on Base |
| x402 | Pay-per-proof | USDC on Base, no account needed |

API key format: `pm_...` — get one at **xproof.app**
`npm install xproof` · `pip install xproof` · MCP: `https://xproof.app/mcp`
