---
name: find-skill
description: Finds and installs Agent Skills for Antigravity (IDE and CLI). 14 sources, ranked by GitHub stars. Examples: "find a skill for Docker", "search for testing skills", "show all React skills". Supports parameters: limit (how many to show), page (pagination), --all (show all results).
---

# Find Skill — skill discovery and installation

You are an expert skill finder with a local catalogue (14 sources, ranked by GitHub stars).

---

## Input parameters

The skill accepts free-form arguments. Parse them as follows:

| Format | Example | Meaning |
|--------|---------|---------|
| `<query>` | `docker` | Keyword search, show top 3 |
| `<query> --limit N` | `react --limit 10` | Show N results |
| `<query> --all` | `design --all` | Show ALL matching results |
| `<query> --page N` | `python --limit 5 --page 2` | Page N (with `limit` per page) |
| `<query> --agent <name>` | `docker --agent cursor` | Filter by agent: `claude`, `codex`, `opencode`, `cursor`, `any` |
| `--top N` | `--top 20` | Show top-N skills by stars across the entire catalogue |
| `--stats` | `--stats` | Catalogue stats by source and agent |
| `--category <cat>` | `--category design` | All skills in a category |

**Defaults:** limit=5, page=1, **agent=claude** (Antigravity reads the same SKILL.md format as Claude Code; the catalogue has no `antigravity` tag).

**Note on `--agent`:** by default only skills compatible with the SKILL.md format (tag `claude`) are shown. Use `--agent any` to remove the filter and see the full catalogue, `--agent cursor` to find skills for Cursor only, etc.

---

## Stage 0 — Check catalogue freshness

```bash
CACHE_FILE="$HOME/.claude/skills/find-skill/cache/catalogue.json"
LAST_UPDATE="$HOME/.claude/skills/find-skill/cache/last_update.txt"

if [ ! -f "$LAST_UPDATE" ]; then
  echo "Catalogue not initialized — update required"
  NEEDS_UPDATE=true
else
  LAST=$(cat "$LAST_UPDATE")
  NOW=$(date +%s)
  DIFF=$(( (NOW - LAST) / 86400 ))
  if [ "$DIFF" -gt 30 ]; then
    echo "Catalogue is stale ($DIFF days old) — update required"
    NEEDS_UPDATE=true
  else
    echo "Catalogue is fresh (updated $DIFF days ago)"
    NEEDS_UPDATE=false
  fi
fi
```

If `NEEDS_UPDATE=true` — run `~/.claude/skills/find-skill/update-skills-catalogue.sh`.

---

## Stage 1 — Understand the query

If the query is unclear — ask 1-2 questions:
- What stack / language / framework?
- What specific task is this for?

If the query is clear — proceed directly to stage 2.

---

## Stage 2 — Search the local catalogue (complement, not source of truth)

> Run **Stage 3 first**. The catalogue below caps each source and has blind spots — it returned zero for `mikrotik` while the live search returned 71.


```bash
cat ~/.claude/skills/find-skill/cache/catalogue.json | \
  python3 -c "
import json, sys, re

data = json.load(sys.stdin)
query = 'QUERY'.lower()
limit = LIMIT        # replace with number from parameter
page = PAGE          # replace with number from parameter
show_all = SHOW_ALL  # True/False
agent_filter = 'AGENT_FILTER'  # 'claude', 'codex', 'opencode', 'cursor', 'any'

# Source priority — scores proportional to GitHub stars (April 2026)
SOURCE_PRIORITY = {
    'Anthropic': 60,              # 105K stars — official Anthropic skills
    'skills.sh': 30,              # Vercel-curated catalogue, ~4K skills
    'hesreallyhim': 28,           # 39.9K stars — top awesome-list
    'ComposioHQ': 25,             # 49K stars — curated list
    'vercel-labs': 12,            # 24K stars — Vercel agent skills
    'VoltAgent-subagents': 8,     # 15.5K stars — Claude Code subagents
    'VoltAgent': 7,               # 13K stars — awesome agent skills
    'travisvn': 5,                # 10K stars — curated list
    'BehiSecc': 4,                # 8K stars — curated list
    'alirezarezvani': 4,          # 8K stars — large collection
    'heilcheng': 3,               # 3.5K stars — awesome agent skills
    'daymade': 3,                 # 744 stars — production-ready collection
    'mxyhi': 3,                   # 188 stars — ok-skills
}

# Search: name, description, tags + agent filter
results = []
for s in data['skills']:
    # Agent filter (if agent_filter != 'any')
    if agent_filter and agent_filter != 'any':
        skill_agents = s.get('agents', ['claude', 'codex'])  # legacy default
        if agent_filter not in skill_agents:
            continue

    score = 0
    name_l = s['name'].lower()
    desc_l = s.get('description', '').lower()
    tags_l = [t.lower() for t in s.get('tags', [])]

    # Query relevance
    if query == name_l:
        score = 100
    elif query in name_l:
        score = 50
    elif query in desc_l:
        score = 20
    elif any(query in t for t in tags_l):
        score = 10

    if score > 0:
        # Bonus for source (trust priority)
        source = s.get('source', '')
        source_bonus = SOURCE_PRIORITY.get(source, 0)

        # Bonus for stars (max +20)
        try:
            stars = int(str(s.get('stars', 0) or 0).replace(',','').replace('+',''))
        except:
            stars = 0
        stars_bonus = min(stars / 1000, 20)

        s['_score'] = score + source_bonus + stars_bonus
        s['_stars'] = stars
        s['_source_rank'] = source_bonus
        results.append(s)

# Sort: score desc (includes source + stars), then stars desc
results.sort(key=lambda x: (-x['_score'], -x['_stars']))

total = len(results)
if show_all:
    page_results = results
else:
    start = (page - 1) * limit
    page_results = results[start:start + limit]

total_pages = (total + limit - 1) // limit if not show_all else 1

output = {
    'total': total,
    'showing': len(page_results),
    'page': page if not show_all else 1,
    'total_pages': total_pages,
    'limit': limit,
    'agent_filter': agent_filter,
    'results': page_results
}
print(json.dumps(output, indent=2, ensure_ascii=False))
"
```

**How to use `agent_filter`:**
- Default for this file: `agent_filter = 'claude'` (same SKILL.md format as Antigravity)
- If user specified `--agent any` — show full catalogue (agent_filter = 'any')
- If specified `--agent cursor|codex|opencode` — substitute that value
- In `--stats` display distribution by agent

---

## Stage 3 — Live search — RUN THIS FIRST, ALWAYS

The local catalogue is a frozen copy: it caps each source and misses whole domains (vendor-specific network, hardware, niche stacks). **Query the live sources before trusting it.** Use the catalogue only to enrich or as fallback when the network is down.

### 3a. skills.sh — free, no key, no signup

```bash
curl -s --max-time 25 "https://skills.sh/api/search?q=QUERY" | python3 -c "
import json,sys
d = json.load(sys.stdin)
for s in d.get('skills', [])[:LIMIT]:
    print(f\"{s.get('installs',0):>6} inst  {s.get('id')}\")
"
```

Returns `id` as `owner/repo/skill`, plus `installs` — the only popularity signal that reflects real use. **Do not use the sitemap**: `https://skills.sh/sitemap.xml` answers 308 and yields nothing.

### 3b. GitHub code search — finds what no catalogue indexes

Before searching, read the domain terms declared by the repository in use. Each repo carries one
line in its `SKILLS.md`; it is what keeps the search from being capped by the catalogue's sources.

```bash
TERMOS=$(grep -m1 -oP '(?<=<!-- find-skill: termos = ).*(?= -->)' SKILLS.md 2>/dev/null)
```

The list is long on purpose — it is the whole surface of the project. **Do not loop over all of
it.** Pick the 3-5 terms that actually relate to the question, then run one query per chosen term
and merge. A question about backup takes `backup`, `rclone`, `sqlite`, `vault`; a question about
Ubuntu must not drag in Kotlin.

```bash
gh api "search/code?q=QUERY+filename:SKILL.md&per_page=20" \
  --jq '.items[] | "\(.repository.full_name) → \(.path)"' | sort -u

for t in <the 3-5 chosen terms>; do
  gh api "search/code?q=QUERY+$t+filename:SKILL.md&per_page=10" \
    --jq '.items[] | "\(.repository.full_name) → \(.path)"'
  sleep 7
done | sort -u
```

Unauthenticated search answers 401, so `gh` must be logged in. Code search allows 10 queries per
minute — that ceiling is why the terms are chosen, not swept.

This is what surfaces skills that live inside a product repo (`.claude/skills/`, `.agents/skills/`) and never reach any index.

---

## Stage 4 — Show results

**Never install without confirmation.**

Output format depends on the number of results:

### Trust levels (by source, stars as of April 2026):
```
Anthropic (105K)                              → Official (green)
skills.sh (Vercel-curated, ~4K entries)       → Official catalogue (green)
hesreallyhim (39.9K) / ComposioHQ (49K)       → Top awesome-list (green)
vercel-labs (24K)                             → Curated (yellow)
VoltAgent-subagents (15.5K) / VoltAgent (13K) → Curated (yellow)
travisvn (10K) / BehiSecc (8K)                → Curated (yellow)
alirezarezvani (8K) / heilcheng (3.5K)        → Community-vetted (orange)
daymade (744) / mxyhi (188)                   → Community (orange)
```

### Compact (up to 5 results):
```
Found N skills for "QUERY":

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. [name] (RECOMMENDED)
   Source      : Anthropic        | Trust: Official
   Description : [1-2 sentences]
   Stars       : [N]
   Repo        : [URL]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
2. [name]
   Source      : skills.sh        | Trust: Official catalogue
   ...

Install? (1, 2, all, or no)
```

### Table (6+ results):
```
Found N skills for "QUERY" (page PAGE/TOTAL_PAGES):

| #  | Name              | Source     | Trust   | Stars  | Description (brief)    |
|----|-------------------|------------|---------|--------|------------------------|
| 1  | skill-name        | Anthropic  | Offic.  | 9600   | Brief description...   |
| 2  | another-skill     | travisvn   | Curat.  | 5600   | Brief description...   |
| 3  | some-skill        | GitHub     | Verify  | 120    | Brief description...   |

Page PAGE of TOTAL_PAGES. Next: /find-skill QUERY --page NEXT
Install? (number, range 1-3, all, or no)
```

Results are automatically sorted: official and curated first, then community. Within the same source — sorted by stars.

---

## Stage 5 — Install after confirmation

```bash
bash "$HOME/.claude/skills/find-skill/scripts/install-skill.sh" "[REPO_URL]" --target antigravity
```

The skill lands in both global folders, unchanged:

- IDE / Antigravity 2.0: `~/.gemini/config/skills/[SKILL_NAME]/`
- CLI: `~/.gemini/antigravity-cli/skills/[SKILL_NAME]/`

For one project only, copy the folder to `<workspace>/.agents/skills/[SKILL_NAME]/` instead.

---

## Stage 6 — Confirm and explain

After installation:
1. Where the skill was installed
2. How to activate it (`/skill-name` or automatically)
3. An example of how to use it in the current project

---

## Special commands

| Query | Action |
|-------|--------|
| `update catalogue` | Run `update-skills-catalogue.sh` |
| `when was catalogue updated?` | Show date from `last_update.txt` |
| `show full catalogue` | All skills by category |
| `--top 20` | Top-20 by stars |
| `--stats` | Stats by source |
| `react --all` | All skills matching query |
| `python --limit 10 --page 2` | Page 2 with 10 results per page |

---

## Rules

- **Never install without confirmation**
- **Default 5 results**, but user can request more
- **Flag risks** if the source is unknown
- **Priority**: Anthropic (105K) > skills.sh (Vercel-catalog) > hesreallyhim (39.9K) > ComposioHQ (49K) > vercel-labs (24K) > VoltAgent-subagents (15.5K) > VoltAgent (13K) > travisvn (10K) > BehiSecc/alirezarezvani (8K) > heilcheng (3.5K) > daymade/mxyhi
- **Table format** for 6+ results to keep output compact

---

## Deactivation / removal

The catalogue and scripts stay in `~/.claude/skills/find-skill/` (shared by every agent). Only the skill folders are Antigravity's.

### Temporarily disable
```bash
for d in ~/.gemini/config/skills ~/.gemini/antigravity-cli/skills; do mv "$d/find-skill" "$d/find-skill-disabled"; done
```

### Re-enable
```bash
for d in ~/.gemini/config/skills ~/.gemini/antigravity-cli/skills; do mv "$d/find-skill-disabled" "$d/find-skill"; done
```

### Stop auto-update (cron)
```bash
crontab -l | grep -v "update-skills-catalogue" | crontab -
```

### Remove completely
```bash
bash ~/.claude/skills/find-skill/uninstall.sh --target antigravity
```
