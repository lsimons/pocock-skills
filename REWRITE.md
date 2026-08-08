# Rewrite: strip this fork to essentials

This repo is a fork of [mattpocock/skills](https://github.com/mattpocock/skills) kept
for personal use, not for redistribution. The upstream repo is built to be *published*:
it ships a Claude Code plugin, a changesets-driven release process, a public docs site
(aihero.dev), and README content aimed at recruiting new users. None of that applies to
a personal fork — there is no one to publish a release to, no plugin marketplace listing
to maintain, no newsletter to point at. Your job is to strip the repo down to what
actually earns its keep for one person's daily use: the skills themselves, plus a single
cross-agent config file, plus ordinary repo boilerplate.

Read this whole file before changing anything.

## How to work

- **Propose a plan before executing it.** List the full current file tree first, group
  files by what they're for, and write up a stripping plan. Get that plan approved
  before deleting or moving anything.
- **Stage, don't commit, until asked.** Use `git add` so the diff is reviewable, but
  don't create commits until told to. Work in a small number of reviewable batches
  (e.g. "drop distribution/release tooling" as one batch, "make skills self-contained"
  as another) rather than one giant diff.
- **Don't guess on judgment calls — ask.** Several decisions below are genuinely
  open (which buckets to keep, whether to keep the router skill, whether to keep
  `.out-of-scope/` as a decision log). Ask a focused question rather than picking
  silently. It's fine to recommend an option when you ask.
- **Verify claims about the repo, don't assume they still hold.** File lists, skill
  counts, and bucket contents below are a snapshot — re-derive them from the actual
  repo state before acting on them.

## What must remain

- `skills/` — the actual skill content. This is the entire point of the fork.
- A **single** cross-agent instructions file (`CLAUDE.md`/`AGENTS.md` — check which one
  is presently the real file and which is the symlink; keep exactly one real file and
  one symlink alias, don't create two independent files that can drift).
- Ordinary boilerplate: `README.md`, `LICENSE`, `.gitignore`, `CONTEXT.md` (if it
  still describes this repo's own domain language for skill-authoring, not a template
  meant for end users' projects).

Everything else is a candidate for removal — justify keeping it, not the reverse.

## What to strip, and why

Go through the repo's current top-level and `.agents/`/`.claude-plugin/` contents and
classify each item. As of the last time this was reviewed, the following categories of
cruft existed — confirm they still exist before removing, since the repo may have
changed:

- **Plugin distribution machinery** — `.claude-plugin/marketplace.json`,
  `.claude-plugin/plugin.json`, and any equivalent for other agents. These exist so
  strangers can install this as a versioned, read-only bundle. A personal fork is
  edited in place; there's no one subscribing to it. Drop them, and drop any ADR whose
  entire subject is plugin-shipping strategy (e.g. one weighing a Claude Code plugin
  against a deferred Codex plugin) — it's a decision record for a problem that no
  longer exists once the plugin is gone.
- **Release/versioning tooling** — changesets config (`.changeset/`), `CHANGELOG.md`,
  `package.json`/`package-lock.json` (unless something in the repo has a real runtime
  dependency on them — check before deleting), and any script whose only job is
  keeping a plugin manifest's version in sync with `package.json`. There are no
  consumers pinning to version numbers in a personal fork.
- **Install/promotional surface** — any install-instructions doc (multiple install
  paths for different agents, "30-second setup" framing), newsletter/marketing links,
  and README sections that exist to sell the repo to a stranger rather than to explain
  it to the one person using it. Rewrite the README to describe what the skills do and
  how to reach for them; drop the sales pitch.
- **Published-docs surface** — a `docs/` tree whose pages are written for a public
  website (check for a stated URL pattern like a per-skill published page) rather than
  for someone reading the repo. If every promoted skill has a mirrored `docs/<bucket>/
  <name>.md`, and that doc's content is substantially just an expanded version of the
  skill's own `SKILL.md`, drop the `docs/` tree and the doc-writing template that
  produces it. Keep the substance (the "what it does / when to use it / common
  questions" framing is genuinely useful) as guidance for how to write a good
  `SKILL.md`, not as a second artifact to keep in sync.
- **Plugin-curation bookkeeping.** If skills are organized into "promoted" vs.
  "non-promoted" buckets because a plugin manifest could only ship an explicit list,
  that rule loses its reason to exist once the plugin does. Decide what the buckets
  (e.g. `engineering/`, `productivity/`, `misc/`, `in-progress/`, `deprecated/`) are
  *actually* for once nothing is being curated for external shipment — organizational
  clarity is still a fine reason to keep folders, but don't keep a promoted/
  non-promoted split, a plugin manifest skill list, or a link-checking script that only
  exists to keep that list honest.
- **Public-issue-tracker decision logs.** A directory of "why we won't build X" notes
  that cite external issue numbers is a maintainer's answer to public feature
  requests. Ask whether it's worth keeping as a personal decision log (context for
  *why* something isn't there) or should go — it has no audience once the repo isn't
  fielding outside requests.
- **CI workflows** that test plugin installation, or any other automation whose sole
  purpose is validating the now-removed distribution surface.

## Making skills self-contained

Check every `SKILL.md` for references to material outside its own skill directory —
shared checklists, a shared `references/` folder at the repo root, or prose that
points at `docs/` pages you're about to delete. Skills should be self-contained
directories: a skill's supporting files live under its own directory, one level deep
from `SKILL.md` (see the [Agent Skills spec](https://agentskills.io/specification)).

- If a skill links to a checklist or reference file used by only that skill, keep it
  under that skill's own directory.
- If multiple skills draw on the same shared checklist that currently lives outside
  any single skill's directory, there is no spec-compliant shared location — duplicate
  the file into each consuming skill's own directory instead of a symlink or a shared
  top-level folder (symlinks break the moment a skill is distributed or zipped
  individually). Record which files are duplicated and where, in the cross-agent
  config file, so future edits get copied to every copy instead of silently drifting.
- Cross-skill relationships that are expressed as "run skill X" in prose are fine and
  don't need to change — that's invocation, not a file reference.

## Consolidating into one cross-agent file

If per-agent-specific metadata exists alongside each skill (for example, a per-skill
YAML file carrying one agent's UI metadata or an invocation-policy flag mirrored from
the skill's own frontmatter), decide whether that duplication is still worth the
upkeep once you're not shipping a multi-agent plugin. If you're only using one or two
agents day to day, consider folding the invocation-policy distinction (which skills
are reachable only by explicitly naming them vs. reachable by the model on its own)
directly into each `SKILL.md`'s frontmatter and dropping the parallel per-agent files,
rather than keeping two files in sync for agents you don't use.

Whatever install/config guidance survives (how to point an agent at this repo's
skills, any one-time per-project setup step) belongs in the single cross-agent config
file, written so it works regardless of which agent is reading it — not as a slash
command, since not every agent supports those.

## Set up an upstream sync procedure

This fork will keep diverging from `mattpocock/skills` in structure once you strip it,
which means upstream improvements to individual skills' *content* need to be
hand-ported rather than merged. Once the stripping is done and committed:

1. Add the upstream remote if it's missing and fetch it:
   ```
   git remote get-url upstream >/dev/null 2>&1 || git remote add upstream https://github.com/mattpocock/skills.git
   git fetch upstream
   ```
2. Add a "Syncing with Upstream" section to the cross-agent config file describing the
   procedure: how to find what changed upstream since the last sync (a git tag marking
   the last processed upstream commit, falling back to `git merge-base` on first run),
   how to walk changed `skills/<name>/` directories one at a time and port substantive
   content changes (not structural ones that don't apply here), and how to handle
   changes to any file you duplicated per the self-containment step above (port the
   change to every duplicate).
3. Explicitly list what never gets ported: anything in the categories under "What to
   strip, and why" above — this fork deliberately removed that surface, so upstream
   changes to it aren't relevant.
4. After the first sync-procedure commit, if the fork is genuinely caught up with
   upstream at that point, tag the commit (e.g. `upstream-sync-point`) and push the
   tag, so the next sync has a real starting point instead of falling back to the
   merge-base every time.

## Order of operations

This matches the shape that worked well doing this kind of stripping before — treat it
as a default, not a rigid sequence, and check in with the user between batches:

1. Inventory the repo, propose the strip plan, get it approved.
2. Execute the strip in one staged batch; leave it for review before committing.
3. Fix self-containment (skill-owned references, duplicated shared checklists) as a
   separate batch, since it usually surfaces follow-up questions the first pass
   didn't.
4. Consolidate any remaining per-agent duplication into the single cross-agent file.
5. Commit what's been reviewed and approved so far.
6. Add the upstream sync procedure, and only tag `upstream-sync-point` once you've
   confirmed the fork is actually caught up with `upstream/main`.
7. Align skill conventions with `~/git/mdd/mdd`, as its own pass (see below).
8. Check compatibility with `~/git/sbp/agents.md` and document coexistence, as its own
   pass (see below).

## Aligning with mdd conventions

`~/git/mdd/mdd` is a real project you use these skills on, with its own established
conventions (spec workflow, cross-referencing rules, toolchain) documented in its
`AGENTS.md`, `README.md`, and `docs/`. Where a skill's default behavior would produce
output that doesn't fit those conventions, the skill should defer to them rather than
impose its own — you're the one hitting the friction, so it's worth fixing once instead
of overriding it by hand every session.

Do this as two steps, not one, and get sign-off on the findings before changing
anything — the specific adjustments below are a starting point from a prior pass over a
different skill set, not a checklist to apply blindly to this one:

1. **Review.** Read mdd's `AGENTS.md`, `README.md`, and `docs/` conventions, then read
   through this repo's skills (particularly the spec/planning workflow skills — e.g.
   `to-spec`, `to-tickets`, `wayfinder` — and anything that discusses ADRs, decision
   records, or domain documentation, e.g. `domain-modeling`, `grill-with-docs`) looking
   for concrete incompatibilities: a skill's default file location or template
   conflicting with mdd's own numbered/indexed convention, a skill modeling a
   cross-referencing pattern mdd's `AGENTS.md` forbids, toolchain examples that assume
   the wrong package manager or task runner, or vocabulary mismatches. Report what you
   find before changing anything — some of this may turn out to already be handled by
   a skill's own "match existing project convention first" clause, in which case there's
   nothing to fix.
2. **Align, once you have sign-off on which findings to act on.** Likely candidates,
   based on what mdd actually does and what a prior alignment pass against a similar
   skill set found worth changing:
   - If a skill defaults to a spec/plan location or template of its own, make it defer
     to mdd's actual convention (a numbered, indexed doc corpus under `docs/spec/`) as
     the preferred default when nothing else is already established, while still
     falling back to whatever a project has already established — never introduce a
     second competing scheme.
   - If a skill's examples model citing specs/ADRs from inside code comments or
     docstrings as the default pattern, and mdd's convention is references should flow
     one way (docs cite code, not the reverse), make the one-way direction the
     skill's default without necessarily forbidding the reverse outright — that's a
     stricter, project-specific rule mdd itself enforces, not something every project
     using this skill needs.
   - If a skill maintains its own ADR-style decision-record location distinct from a
     project's existing numbered/status-tracked doc corpus, note that the two can be
     the same system with different names, so a skill doesn't accidentally suggest
     standing up a second, parallel one.
   - Update toolchain examples (CI, git hooks, test runner references, lint/format
     commands) so they show `mise` as the task-runner front door, `pnpm` for
     JavaScript/TypeScript examples (matching `~/git/lsimons/lsimons-template-ts`), and
     `uv`/`ruff` for Python examples (matching `~/git/lsimons/lsimons-template-py`) —
     read both template repos rather than assuming their toolchain hasn't moved on.
     Don't add a tool neither template actually uses.
   - Leave alone anything that's a deliberate, unrelated choice this fork already made
     on purpose (e.g. a commit-message convention, or a stance on a specific
     integration) — check with the user before changing anything outside the scope of
     "align with mdd" rather than opportunistically tidying nearby content.
   - If duplicated reference files (per the self-containment step above) are among the
     things you change, update every duplicate, not just the first one you touch.

## Checking compatibility with sbp/agents.md

`~/git/sbp/agents.md` ([schubergphilis/agents.md](https://github.com/schubergphilis/agents.md))
is a separate, independently maintained pack of mission-critical-engineering skills
(`sbp-*`-prefixed, plus a few unprefixed ones) that may be installed alongside this
fork's skills in the same agent. Its own docs claim coexistence is supported. Check
whether that claim actually holds for this repo's skills, and document how the two
packs should behave together:

1. **Review.** Read `~/git/sbp/agents.md`'s baseline `AGENTS.md`, its skill list, and a
   sample of its skills, and compare against this repo's skills for:
   - **Routing capture** — if this repo has a router/dispatch skill (e.g. `ask-matt`)
     with a closed list of only its own skills, installing both packs means the router
     will silently send work to this repo's more generic skill instead of an
     available, more rigorous `sbp-*` equivalent (e.g. security review, threat
     modeling, debugging), unless the user names the `sbp-*` skill explicitly. This is
     the highest-severity risk to check for and fix.
   - **Divergent standards** — e.g. a differing bar for what counts as an acceptable
     change (general improvement vs. a zero-defect, hard-gate standard), which could
     cause waffling if both packs' skills are in context for the same task.
   - **Lifecycle/vocabulary mismatches** — if this repo organizes skills by a named
     set of phases or buckets and `sbp-*` skills use different phase/lifecycle
     labels in their own frontmatter, note that an agent told to hand off at a phase
     boundary using one vocabulary may have nothing to map to in the other.
   - **Redundant-but-not-conflicting overlaps** — skills in both packs addressing
     similar problems (e.g. security review, debugging, documentation-of-decisions)
     without actually contradicting each other; these are fine to leave as-is, just
     worth naming so it's clear neither needs to change.
   - **Frontmatter/metadata differences** that don't actually block coexistence (e.g.
     one pack's frontmatter carries extra fields the other lacks).
2. **Document coexistence**, once you've confirmed there's no hard incompatibility
   (directory names not colliding, both markdown-first and Skill-tool-invoked): add a
   section to the README (and the cross-agent config file, and the router skill if one
   exists) listing `sbp-*`'s skills and stating that they take precedence over this
   pack's equivalents when both are installed, so mission-critical work routes to the
   more rigorous `sbp-*` skill rather than being silently captured by the generic one
   here. If a router/dispatch skill exists, give it enough awareness of `sbp-*`
   skill names that it doesn't need the user to name them explicitly to avoid capture.
