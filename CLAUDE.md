# pocock-skills

A personal fork of [mattpocock/skills](https://github.com/mattpocock/skills), kept for daily use across
Claude Code and Codex. It is edited in place, not published or distributed — there is no plugin, no
release process, and no public docs site here. See [README.md](./README.md) for what the skills do.

## Skill layout

Every skill is a self-contained directory directly under `skills/<name>/`, one level deep from its own
`SKILL.md` (see the [Agent Skills spec](https://agentskills.io/specification)). A skill's supporting files
(checklists, templates, scripts) live under its own directory — never in a shared top-level folder, and
never referenced by a relative path that reaches into another skill's directory or outside `skills/`.

If two skills would otherwise want the same shared reference file, there is no spec-compliant shared
location: duplicate the file into each consuming skill's own directory instead. There are currently no
duplicated files to track here — if you introduce one, list it below so future edits get copied to every
copy instead of silently drifting.

Cross-skill relationships expressed as "run the `/other-skill` skill" in prose are fine and don't need a
file reference — that's invocation, not a dependency on another skill's files.

`README.md` groups skills informally (Engineering, Productivity) for readability; those groupings don't
correspond to folders and don't gate anything — nothing curates a subset of `skills/` for distribution.

## Invocation policy

Every `SKILL.md` is either:

- **User-invoked** — reachable only by the human typing its name. Set `disable-model-invocation: true` in
  the frontmatter. The `description` is human-facing: a one-line summary read by a person browsing
  slash-commands, not a list of model trigger phrases.
- **Model-invoked** — reachable by model or user (the default: omit `disable-model-invocation`). The
  `description` is model-facing and keeps rich trigger phrasing ("Use when the user wants…, mentions…,
  asks for…") so auto-invocation fires. The test: _could the model usefully reach for this autonomously?_
  (Reuse is the reason to extract a skill, not the test.)

This frontmatter field is the single source of truth for invocation policy across every agent — there is
no separate per-agent metadata file to keep in sync. A user-invoked skill may invoke model-invoked skills,
but never another user-invoked one.

## The router

[`ask-leo`](./skills/ask-leo/SKILL.md) maps every user-reachable skill and how they relate. Whenever you
add, rename, remove, or change how a user-reachable skill fits the flows, re-read `ask-leo`'s `SKILL.md`
and update it so the map stays accurate — a new skill it never mentions, or a stale one it still routes to,
is a router that lies.

## Coexisting with sbp-* skills

[schubergphilis/agents.md](https://github.com/schubergphilis/agents.md) is a separate pack of
mission-critical-engineering skills (`sbp-*`-prefixed, plus `mcaf-module`, `review-mcaf`, `terraform`) that
may be installed alongside this pack in the same agent. No directory-name collisions, both are
markdown-first and Skill-tool-invoked — they coexist without conflict. Where an `sbp-*` skill overlaps one
here (code review, debugging, architecture review, testing, feature development), the `sbp-*` skill takes
precedence — it's the more rigorous, mission-critical-grade version. See
[`ask-leo`](./skills/ask-leo/SKILL.md)'s "When `sbp-*` skills are installed" section for the specific
mapping; keep that section in sync if either pack's skill list changes.

## Writing a good SKILL.md

When a skill's job isn't obvious from its name, its `SKILL.md` should make two things clear early on:

- **What it does** — its one-sentence job, then the defining constraint: the single fact that makes it
  behave differently from the obvious default.
- **When to reach for it** — the trigger boundary, and where it's confusable with a sibling, what to use
  instead for that other case.

Optionally cover prerequisites (a workspace it writes into, prior setup another skill provides) and
whatever makes the skill's own approach click — its own vocabulary, the loop it runs, the artifact it
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

2. Find what changed upstream since the last sync: `git log --stat <last-sync-point>..upstream/main`. Use
   the tag `upstream-sync-point` as `<last-sync-point>` if it exists; otherwise fall back to
   `git merge-base HEAD upstream/main`.

3. Walk the changed `skills/<name>/` directories one at a time (upstream paths are
   `skills/<bucket>/<name>/`; this fork's are `skills/<name>/`). For each, port substantive content
   changes — new guidance, fixed prose, behavior changes — and skip purely structural changes that don't
   apply here (a file moving between buckets, a `docs/` page edit, an `agents/openai.yaml` update).

4. If a change touches a file this fork duplicates per the self-containment rule above, port it to every
   duplicate, not just the first one you find.

5. After porting, move the `upstream-sync-point` tag to the commit you synced through and push it:

   ```
   git tag -f upstream-sync-point <commit>
   git push origin upstream-sync-point --force
   ```

**Never ported** — this fork deliberately removed this surface, so upstream changes to it aren't
relevant: `.claude-plugin/`, `.changeset/`, `CHANGELOG.md`, `package.json`/`package-lock.json`,
`scripts/sync-plugin-version.mjs`, the release GitHub workflow, `docs/` and its writing-docs template,
`.agents/install-block.md`, `.out-of-scope/`, per-skill `agents/openai.yaml`, the
promoted/non-promoted bucket split, and anything specific to the `misc/`/`in-progress/` skills this fork
dropped.
