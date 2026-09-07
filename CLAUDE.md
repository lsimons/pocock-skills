# Agent instructions for pocock-skills

> This file (`CLAUDE.md`) is the canonical agent configuration. `AGENTS.md` is a symlink to this file.

This file configures agents working on this repository itself. It is not meant to be copied into other projects or into a global agent configuration; the reusable assets are the skills in `skills/`, not this file.

## Prose style

No em-dashes anywhere in this repo's prose (`SKILL.md` files, `README.md`, `CLAUDE.md`, `CONTEXT.md`, commit messages). Where a sentence reaches for one, rewrite it instead with a comma, colon, period, parentheses, or a conjunction, whichever the sentence actually wants; never do a blind character substitution. This matches upstream and keeps future syncs from fighting over punctuation.

## Skill layout

Every skill is a self-contained directory directly under `skills/<name>/`, one level deep from its own
`SKILL.md` (see the [Agent Skills spec](https://agentskills.io/specification)). A skill's supporting files
(checklists, templates, scripts) live under its own directory, never in a shared top-level folder, and
never referenced by a relative path that reaches into another skill's directory or outside `skills/`.

If two skills would otherwise want the same shared reference file, there is no spec-compliant shared
location: duplicate the file into each consuming skill's own directory instead. There are currently no
duplicated files to track here. If you introduce one, list it below so future edits get copied to every
copy instead of silently drifting.

Cross-skill relationships expressed as "run the `/other-skill` skill" in prose are fine and don't need a
file reference: that's invocation, not a dependency on another skill's files.

`README.md` groups skills informally (Engineering, Productivity) for readability; those groupings don't
correspond to folders and don't gate anything. Nothing curates a subset of `skills/` for distribution.

## Invocation policy

Every `SKILL.md` is either:

- **User-invoked**: reachable only by the human typing its name. Set `disable-model-invocation: true` in
  the frontmatter. The `description` is human-facing: a one-line summary read by a person browsing
  slash-commands, not a list of model trigger phrases.
- **Model-invoked**: reachable by model or user (the default: omit `disable-model-invocation`). The
  `description` is model-facing and keeps rich trigger phrasing ("Use when the user wants…, mentions…,
  asks for…") so auto-invocation fires. The test: _could the model usefully reach for this autonomously?_
  (Reuse is the reason to extract a skill, not the test.)

This frontmatter field is the single source of truth for invocation policy across every agent. There is
no separate per-agent metadata file to keep in sync. A user-invoked skill may invoke model-invoked skills,
but never another user-invoked one.

A `description` containing a colon must be quoted, or the YAML front matter fails to parse.

## The router

[`ask-leo`](./skills/ask-leo/SKILL.md) maps every user-reachable skill and how they relate. Whenever you
add, rename, remove, or change how a user-reachable skill fits the flows, re-read `ask-leo`'s `SKILL.md`
and update it so the map stays accurate: a new skill it never mentions, or a stale one it still routes to,
is a router that lies.

## Coexisting with sbp-* skills

[schubergphilis/agents.md](https://github.com/schubergphilis/agents.md) is a separate pack of
mission-critical-engineering skills (`sbp-*`-prefixed, plus `mcaf-module`, `review-mcaf`, `terraform`) that
may be installed alongside this pack in the same agent. No directory-name collisions, both are
markdown-first and Skill-tool-invoked, so they coexist without conflict. Where an `sbp-*` skill overlaps one
here (code review, debugging, architecture review, testing, feature development), the `sbp-*` skill takes
precedence: it's the more rigorous, mission-critical-grade version. See
[`ask-leo`](./skills/ask-leo/SKILL.md)'s "Coexisting with sbp/agents.md" section for the specific
mapping; keep that section (and the matching table in README.md) in sync if either pack's skill list
changes.

## Writing a good SKILL.md

When a skill's job isn't obvious from its name, its `SKILL.md` should make two things clear early on:

- **What it does**: its one-sentence job, then the defining constraint: the single fact that makes it
  behave differently from the obvious default.
- **When to reach for it**: the trigger boundary, and where it's confusable with a sibling, what to use
  instead for that other case.

Optionally cover prerequisites (a workspace it writes into, prior setup another skill provides) and
whatever makes the skill's own approach click: its own vocabulary, the loop it runs, the artifact it
produces. Skip anything that doesn't apply; there's no fixed template.

## Syncing with upstream

This fork has diverged from [mattpocock/skills](https://github.com/mattpocock/skills) in structure
(flattened `skills/`, no plugin/release/docs machinery, no `agents/openai.yaml`), so upstream changes have
to be hand-ported per skill rather than merged wholesale.

1. Add the remote and fetch if not already set up:

   ```
   git remote get-url upstream >/dev/null 2>&1 || git remote add upstream https://github.com/mattpocock/skills.git
   git fetch upstream
   ```

2. Find what changed upstream since the last sync: `git log --stat <last-sync-point>..upstream/main`.
   `<last-sync-point>` is `git merge-base HEAD upstream/main`, which is accurate as long as step 6 below
   was run for the previous sync. The tag `upstream-sync-point` records the same commit as a readable
   label; if the two disagree, trust the merge-base and re-point the tag.

3. Walk the changed `skills/<name>/` directories one at a time (upstream paths are
   `skills/<bucket>/<name>/`; this fork's are `skills/<name>/`). For each, port substantive content
   changes (new guidance, fixed prose, behavior changes) and skip purely structural changes that don't
   apply here (a file moving between buckets, a `docs/` page edit, an `agents/openai.yaml` update).

   For a file this fork has never touched, take upstream's version wholesale. For one the fork has edited,
   `git merge-file --diff3` against the sync-point version resolves most of it mechanically; only the
   hunks the fork rewrote need a judgment call.

4. If a change touches a file this fork duplicates per the self-containment rule above, port it to every
   duplicate, not just the first one you find.

5. Commit the ported content on its own, so the port is reviewable as a content diff.

6. Then record the sync in git's history with a content-free merge, so the branch stops reporting as N
   commits behind upstream and the next sync's merge-base is correct:

   ```
   git merge -s ours upstream/main
   git tag -f upstream-sync-point <the upstream commit you synced through>
   git push origin main && git push origin upstream-sync-point --force
   ```

   `-s ours` keeps this fork's tree byte-for-byte and only adds the second parent. Run it **after** the
   content port, never instead of it: on its own it would silently declare upstream's changes absorbed
   while dropping them. The trade-off it accepts is that git will no longer surface unported upstream
   paths as outstanding work, so reviving one later (promoting an `in-progress/` skill, say) means copying
   it out of an upstream ref by hand.

**Never ported**, because this fork deliberately removed this surface, so upstream changes to it aren't
relevant: `.claude-plugin/`, `.changeset/`, `CHANGELOG.md`, `package.json`/`package-lock.json`,
`scripts/sync-plugin-version.mjs`, `scripts/link-skills.sh`, the release GitHub workflow, `docs/` and its
writing-docs template, `.agents/install-block.md`, `.out-of-scope/`, per-skill `agents/openai.yaml`, the
promoted/non-promoted bucket split, and anything specific to the `misc/`/`in-progress/` skills this fork
dropped.
