---
name: groundtruth
description: >-
  Makes a documentation surface tell the truth, thoroughly — audit it against PRIMARY reality (the
  running system, the code, git), fix every drifted claim, fill the gaps reality has that the docs
  don't, re-probe after each fix (never trust your own edit), and propagate corrections across every
  sibling surface (README / CLAUDE.md / in-app help pages / config comments) so they agree. Use when
  docs may be stale, lying, or incomplete. Triggers (plain-English first): "are the docs accurate",
  "the README/CLAUDE.md is stale", "update the docs to match reality", "the docs are out of date",
  "make the docs accurate", "check the docs against the code", "fix the stale docs", then the
  shorthand "groundtruth" / "true up the docs". For finalizing or shipping a change just made (version
  bump + regression test), use the seal skill instead — groundtruth is the standalone truth-up of an
  existing surface, not a release step.
---

# /groundtruth — make the docs tell the truth, thoroughly

Take a documentation surface — a README, CLAUDE.md, an in-app help page, a public docs
site, a config's comments, a whole repo's docs — and make it an **accurate and
complete** reflection of reality. Two moves, in order: **fix what's false** (drift
to truth), then **add what's missing** (gaps reality has that the docs don't
mention). Verify against PRIMARY sources — the running system, the code, git —
never against other docs (a doc citing a doc that's also wrong just compounds the
lie). It edits DOCS only: where the doc and the code disagree, it reports the
contradiction, it doesn't pick a winner.

**Why this exists:** stale docs are worse than missing docs — missing docs make
you look; stale docs make you trust a lie. A README naming a default model that the deploy changed underneath it. A CLAUDE.md host
table pointing at a machine decommissioned weeks ago. A config
comment saying "sanitized placeholders" with the real values right under it.
Blue/green labels that stopped matching which side is live. Each was a doc that
actively misled because nobody ran the audit→truth pass while the context was
fresh. This skill IS that pass, as a first-class, repeatable move.

## Input

- What to true up: a path, a repo, "CLAUDE.md", "the project README", "the docs
  site", "this system's docs". If omitted, infer the surface from the session's
  most recent work. If there's no target AND no session work to infer from (cold
  start), **ask the user which surface** — don't guess.

## Procedure

### 1. Scope the surface(s)

List every doc that describes the target — they travel in packs. For a repo:
README(s), CHANGELOG, in-repo guides, `*.yaml` / `*.env.example` + their comments,
in-app help pages, the public docs site, version stamps. For a system:
its CLAUDE.md section, the host map, the deploy recipe. Name them before touching
any.

- **Public docs site:** its source lives in-repo (e.g. Starlight `./docs/*.mdx`
  with the sidebar in `astro.config.mjs` — a new page needs BOTH the file and the
  sidebar entry; check how the repo registers a new page). Verify the *live* site
  with WebFetch, the *source* with Read.

### 2. Inventory the checkable claims — into a written ledger

Read each surface and pull out the **falsifiable assertions** — things that can be
right or wrong against reality — into a **claims ledger** (a TodoWrite list or a
scratch markdown table; never hold it only in your head, or you'll silently drop
claims — the exact drift this skill exists to kill). One row per claim:
`claim text | surface+location | source-of-truth probe | verdict | corrected value`.
Claim kinds:
- facts: IPs, hostnames, ports, versions, file paths, default values, config keys
- behavior: "X runs Y", "the default is Z", "endpoint A returns B", command lines
- status: "shipped/live", "planned", "deprecated", "TODO"
- cross-refs: "see file Q", links, "as documented in R"

### 3. Verify each claim against PRIMARY reality

For every claim, check the source of truth — the actual thing, not another doc.
**Authority ladder (use the highest reachable):**
- (a) the running system: SSH/curl the box, `/health`, `ollama list`, `systemctl`,
  WebFetch the live site
- (b) the code: read the function / default / route it describes
- (c) git: `git log`, `git rev-parse`, what actually shipped and when

Mark each claim: **TRUE** / **STALE** (reality differs) / **UNVERIFIABLE** /
**MISSING** (reality has it, docs don't). Rules when a source is unreachable:
- **System down?** Drop to code+git as primary; fix everything they can settle;
  mark only genuine runtime-behavior claims UNVERIFIABLE. **Never** substitute
  another doc for a primary source.
- **No git / not a repo?** Skip git; rely on system+code.
- **None of (a)(b)(c) reachable?** STOP and tell the user — don't edit on vibes.

**Scale:** for a surface bigger than ~one screen of claims, or with multiple
independent subsystems, dispatch parallel verifiers (Task subagents), one per
section, each returning its ledger rows; synthesize into the one ledger. Small
surface → verify inline.

### 4. Fix what's false (truth before prose)

Correct every STALE claim to match reality. This is the core, and it comes BEFORE
writing anything new — never paper fresh prose over a lie. **groundtruth edits
DOCS only.** When the doc and the code disagree, do not pick a winner: record it
as a finding for the user (the doc may be the bug, or the code may be —
fixing the code is /seal's job). Likewise STOP and surface real-vs-placeholder
data or a claim that's a deliberate aspiration, not a bug.

### 5. Fill the gaps for thoroughness

Add what reality has that the docs don't: undocumented behavior, a new config key,
a whole subsystem nobody wrote up, the non-obvious trap. **Completeness test:** a
competent stranger could, from these docs alone, (a) install/run it, (b) hit every
documented surface/route, and (c) recover from the named traps. Walk that
operate-from-cold checklist and confirm each item is covered. Add what's
load-bearing; skip the obvious; don't pad.

### 6. Reconcile across surfaces

Propagate each correction to every sibling surface so they agree — CLAUDE.md, the
README, in-app help pages, config comments. A fix in one and not the
others just relocates the lie. Bump / refresh version stamps and "last updated"
markers so nothing advertises a stale age.

### 7. Verify, then deliver

- **Re-probe** the fixed claims against primary reality — confirm the edit is now
  true; don't trust your own keystrokes.
- **Secret/disclosure scan** before anything leaves: use the repo's standing tool
  if one exists (a leak or secret scanner the repo already has);
  otherwise grep the diff for IPs, tokens, internal hostnames, person-names. Public
  surfaces use CAPABILITY language — never internal hostnames (`git.internal.example`),
  tenant/host names, IPs, or admin credentials.
- **Deliver by surface risk** (same commit/secret-scan discipline as /seal step 6):
  - private repo with a clear owner → commit in the repo's own voice; push only if the user has asked for pushes in this session, otherwise stop at the commit and say so.
  - shared CLAUDE.md, a public docs site, or anything that **auto-deploys on
    push-to-main** → show the user the diff and get the nod FIRST.
  - no-git surface → save the edit in place and report the change set; if it
    deploys from a separate source repo, edit the SOURCE and note that.
- **Report** terse: what was STALE (now true), what was MISSING (now added), what's
  UNVERIFIABLE, and anything you STOPPED on for the user.

## Notes

- **vs /seal:** /seal finalizes ONE change (audit → doc → **version bump** →
  **regression gate** → commit). /groundtruth trues an EXISTING surface against
  reality and runs standalone, on docs no single change produced. They're kin —
  seal's step-1 audit is a groundtruth pass narrowed to one change's surfaces, and
  seal points here for large/multi-surface audits — but seal owns the version bump
  and code test; this owns the claim-ledger and cross-surface propagation.
- **If the audit is the closing step of a change you're shipping** (the user also
  says seal / ship / lock-in), that's /seal — groundtruth is the standalone
  truth-up.
- **Primary over secondary, always.** Verify against the running system / the code
  / git, never against another doc. Compounding lies is how one dead host name survives in
  three docs at once.
- **STALE > MISSING in severity.** Fix lies before filling gaps. A missing line is
  a gap; a wrong line is a trap.
- **Don't invent.** UNVERIFIABLE stays flagged, not guessed. "I couldn't reach the
  box" beats a confident wrong edit.
- Pairs with /seal.
