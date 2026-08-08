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

[`ask-matt`](./skills/ask-matt/SKILL.md) maps every user-reachable skill and how they relate. Whenever you
add, rename, remove, or change how a user-reachable skill fits the flows, re-read `ask-matt`'s `SKILL.md`
and update it so the map stays accurate — a new skill it never mentions, or a stale one it still routes to,
is a router that lies.

## Writing a good SKILL.md

When a skill's job isn't obvious from its name, its `SKILL.md` should make two things clear early on:

- **What it does** — its one-sentence job, then the defining constraint: the single fact that makes it
  behave differently from the obvious default.
- **When to reach for it** — the trigger boundary, and where it's confusable with a sibling, what to use
  instead for that other case.

Optionally cover prerequisites (a workspace it writes into, prior setup another skill provides) and
whatever makes the skill's own approach click — its own vocabulary, the loop it runs, the artifact it
produces. Skip anything that doesn't apply; there's no fixed template.
