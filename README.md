---
name: groundtruth
type: tool
status: experimental
license: MIT
version: 0.1.0
summary: A Claude skill that checks a README, config comments or docs site against the running system, the code and git, fixes what is false, then adds what is missing.
updated: 2026-10-03
verified: 2026-10-03
---

# groundtruth

Stale docs are worse than missing docs. Missing docs make you look. Stale docs make you trust a lie.

`groundtruth` takes a documentation surface and makes it accurate and complete, in two moves and in this order: fix what is false, then add what is missing. It checks claims against primary sources (the running system, the code, git) and never against another document, because a doc citing a doc that is also wrong only compounds the lie.

It edits docs only. Where a doc and the code disagree, it reports the contradiction. It does not decide which one is the bug.

## The procedure

1. **Scope.** List every doc that describes the target: READMEs, changelog, in-repo guides, config files and their comments, in-app help pages, the public docs site, version stamps. Name them before touching any.
2. **Ledger.** Pull every falsifiable claim into a written table, one row each: claim text, location, the probe that settles it, verdict, corrected value. Facts (hosts, ports, versions, paths, defaults), behavior ("X runs Y"), status ("shipped", "planned"), and cross-references. The ledger is written down, not held in memory, so claims are not silently dropped.
3. **Verify** each claim against the highest source you can reach:
   - the running system (SSH, curl, `/health`, `systemctl`, the live site);
   - the code (read the function, default or route the doc describes);
   - git (what actually shipped, and when).

   Each claim gets a verdict: TRUE, STALE (reality differs), UNVERIFIABLE, or MISSING (reality has it, the docs do not). If the system is down it falls back to code and git, and marks only genuine runtime claims UNVERIFIABLE. It never substitutes another doc.
4. **Fix what is false.** Every STALE claim is corrected before any new prose is written.
5. **Fill the gaps.** Add undocumented behavior, new config keys, whole subsystems nobody wrote up, the non-obvious trap. The test: could a competent stranger install it, hit every documented surface, and recover from the named traps using only these docs?
6. **Reconcile across surfaces.** Propagate each correction to every sibling surface so they agree. A fix in one place and not the others only moves the lie.
7. **Re-probe, then deliver.** Re-check the fixed claims against reality. Scan the change for secrets and internal details before anything leaves. Then deliver according to risk: commit and push for a private repo with a clear owner, show the diff first for anything that deploys on push.

The final report is short: what was STALE (now true), what was MISSING (now added), what is UNVERIFIABLE, and anything it stopped on.

## Illustrative ledger

The shape of the working table (invented rows):

| Claim | Where | Probe | Verdict | Corrected |
|---|---|---|---|---|
| `/ask` uses model A | README:41 | read `config.py` default | STALE | model B |
| Port 8080 | docs/run.md:12 | `ss -ltn` on the host | TRUE | |
| Phase 3 is "planned" | README:90 | `git log` shows it shipped | STALE | shipped in 1.4.0 |
| `--dry-run` flag | (nothing) | `--help` lists it | MISSING | added to docs/usage.md |

## Rules it keeps

- Primary over secondary, always.
- A stale claim is more severe than a missing one. Fix lies before filling gaps.
- Do not invent. "I could not reach the host" beats a confident wrong edit.
- Public surfaces get capability language, never internal hostnames, IPs or tokens.
- If nothing primary is reachable, stop and say so. It does not edit on a guess.

## Install

Copy `skills/groundtruth` into `~/.claude/skills/` (or a project's `.claude/skills/`). A plugin manifest is included in `.claude-plugin/plugin.json`.

Installed as a plugin the skill is namespaced, for example `/groundtruth:groundtruth`; copied into `skills/` it is `/groundtruth`. Claude also picks it up from the description, so you do not need to type the command. Ask Claude: "are the docs accurate?", "the README is stale", or "true up the docs".

Pairs with [seal](https://github.com/mediagato/seal), which finishes a single change; `groundtruth` is the standalone truth-up of an existing surface.

## Status

Experimental. It is a procedure for Claude to follow, not a program, so how well it works depends on the model and on what the session can actually reach.

## License

MIT. See [LICENSE](LICENSE).

Made at [MEDiAGATO](https://github.com/mediagato).
