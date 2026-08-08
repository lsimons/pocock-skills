# Skills

**Agent skills for real engineering.**

Fork of [mattpocock/skills](https://github.com/mattpocock/skills), stripped down to skills only.

See [`CLAUDE.md`](./CLAUDE.md) for conventions (invocation policy, how skills reference each other) and
the upstream sync procedure.

## Setup

Run `/setup-matt-pocock-skills` once per repo you use these skills in. It will:

- Ask which issue tracker you want to use (GitHub, Linear, or local files)
- Ask what labels you apply to tickets when you triage them (`/triage` uses labels)
- Ask where you want to save any docs it creates

## Why these skills exist

- **Misalignment** — the agent builds something that isn't what you meant. Fix: a **grilling session**,
  where the agent interviews you until the plan is actually resolved. Use
  [`/grill-me`](./skills/grill-me/SKILL.md) for non-code decisions,
  [`/grill-with-docs`](./skills/grill-with-docs/SKILL.md) for code (adds domain-model and ADR upkeep).
- **Verbosity and jargon drift** — the agent and the codebase don't share a vocabulary, so everything
  takes more words than it should. Fix: a `CONTEXT.md` glossary, built and sharpened by
  [`/grill-with-docs`](./skills/grill-with-docs/SKILL.md) and [`/domain-modeling`](./skills/domain-modeling/SKILL.md).
- **Code that doesn't work** — no feedback loop, so the agent is flying blind. Fix:
  [`/tdd`](./skills/tdd/SKILL.md) for a red-green-refactor loop, and
  [`/diagnosing-bugs`](./skills/diagnosing-bugs/SKILL.md) for a disciplined diagnosis loop on hard bugs.
- **Accumulating mud** — agents can write code fast enough to accelerate entropy, not just output. Fix:
  [`/to-spec`](./skills/to-spec/SKILL.md) quizzes you on affected modules before writing a spec, and
  [`/improve-codebase-architecture`](./skills/improve-codebase-architecture/SKILL.md) periodically surveys
  the codebase for deepening opportunities (a survey, not a rescue — it finds candidates, it doesn't fix them).

## Reference

Skills split on one axis — who can invoke them. **User-invoked** skills are reachable only when you type
them (e.g. `/grill-me`); their job is to orchestrate. **Model-invoked** skills can be invoked by you _or_
reached for automatically by the agent when the task fits; they hold the reusable discipline. A
user-invoked skill may invoke model-invoked skills, but never another user-invoked one.

### Engineering — daily code work

**User-invoked**

- **[ask-leo](./skills/ask-leo/SKILL.md)** — Ask which skill or flow fits your situation. A router over the user-invoked skills in this repo.
- **[grill-with-docs](./skills/grill-with-docs/SKILL.md)** — Grilling session that also builds your project's domain model, sharpening terminology and updating `CONTEXT.md` and ADRs inline.
- **[triage](./skills/triage/SKILL.md)** — Move issues through a state machine of triage roles.
- **[improve-codebase-architecture](./skills/improve-codebase-architecture/SKILL.md)** — Scan a codebase for deepening opportunities, present them as a visual HTML report, then grill through whichever one you pick.
- **[setup-matt-pocock-skills](./skills/setup-matt-pocock-skills/SKILL.md)** — Configure this repo for the engineering skills (issue tracker, triage labels, domain doc layout). Run once per repo before using the other engineering skills.
- **[to-spec](./skills/to-spec/SKILL.md)** — Turn the current conversation into a spec and publish it to the issue tracker. No interview — just synthesizes what you've already discussed.
- **[to-tickets](./skills/to-tickets/SKILL.md)** — Break any plan, spec, or conversation into a set of tracer-bullet tickets, each declaring its blocking edges — written as text in a local file, or as native blocking links on a real tracker.
- **[implement](./skills/implement/SKILL.md)** — Build the work described by a spec or set of tickets, driving `/tdd` at pre-agreed seams and closing out with `/code-review` before committing.
- **[wayfinder](./skills/wayfinder/SKILL.md)** — Plan a huge chunk of work, more than one agent session can hold, as a shared map of decision tickets on the issue tracker — resolve them one at a time until the way to the destination is clear.

**Model-invoked**

- **[prototype](./skills/prototype/SKILL.md)** — Build a throwaway prototype to answer a design question — a single shareable HTML file for state/logic questions, or several radically different UI variations toggleable from one route.
- **[diagnosing-bugs](./skills/diagnosing-bugs/SKILL.md)** — Disciplined diagnosis loop for hard bugs and performance regressions: build a feedback loop that goes red on this bug → minimise → hypothesise → instrument → fix → regression-test.
- **[research](./skills/research/SKILL.md)** — Investigate a question against high-trust primary sources and capture the findings as a cited Markdown file in the repo, run as a background agent.
- **[tdd](./skills/tdd/SKILL.md)** — Test-driven development with a red-green-refactor loop. Builds features or fixes bugs one vertical slice at a time.
- **[domain-modeling](./skills/domain-modeling/SKILL.md)** — Actively build and sharpen a project's domain model — challenge terms against the glossary, stress-test with edge-case scenarios, and update `CONTEXT.md` and ADRs inline.
- **[codebase-design](./skills/codebase-design/SKILL.md)** — Shared discipline and vocabulary for designing deep modules: a lot of behaviour behind a small interface, placed at a clean seam, testable through that interface.
- **[code-review](./skills/code-review/SKILL.md)** — Two-axis review of the diff since a fixed point: **Standards** (does it follow the repo's coding standards, plus a Fowler smell baseline?) and **Spec** (does it faithfully implement the originating issue/spec?), run as parallel sub-agents so neither pollutes the other.
- **[resolving-merge-conflicts](./skills/resolving-merge-conflicts/SKILL.md)** — Work through an in-progress git merge or rebase conflict hunk by hunk, resolving by intent traced to each side's primary source, then finish the operation — never `--abort`.
- **[wizard](./skills/wizard/SKILL.md)** — Generate an interactive bash wizard that walks a human through steps only they can perform: provisioning infrastructure, setting up credentials or CI secrets, walking an unfamiliar third-party dashboard, or running a one-off migration or cutover.

### Productivity — daily non-code workflow tools

**User-invoked**

- **[grill-me](./skills/grill-me/SKILL.md)** — Get relentlessly interviewed about a plan or design until every branch of the design tree is resolved.
- **[handoff](./skills/handoff/SKILL.md)** — Compact the current conversation into a handoff document so another agent can continue the work.
- **[teach](./skills/teach/SKILL.md)** — Teach the user a new skill or concept over multiple sessions, using the current directory as a stateful teaching workspace.
- **[to-questionnaire](./skills/to-questionnaire/SKILL.md)** — Turn a decision you can't answer alone into a Markdown questionnaire for the one person who can — filled in async, or together over a meeting. It grills you about the send (who it's for, what you need back), not the subject.
- **[wait-what](./skills/wait-what/SKILL.md)** — Fire this the moment a message doesn't land. The agent re-pitches it with the context you're missing, in plain English, using your `CONTEXT.md` vocabulary.

**Model-invoked**

- **[grilling](./skills/grilling/SKILL.md)** — Interview the user relentlessly about a plan, decision, or idea until every branch of the design tree is resolved. The reusable interview primitive behind `grill-me`, `grill-with-docs`, `triage`, `wayfinder` and `improve-codebase-architecture`.
- **[writing-for-agents](./skills/writing-for-agents/SKILL.md)** — Writing documents for agents: skills, AGENTS.md/CLAUDE.md, and any doc an agent reaches by a pointer.

## Coexisting with sbp/agents.md

This pack is designed to be installed alongside [schubergphilis/agents.md](https://github.com/schubergphilis/agents.md), a separate agentskills.io-compliant pack aimed at mission-critical engineering. Both packs can live in `skills/` side by side without conflict — no directory names collide.

**Prefer the matching `sbp-*` skill over the equivalent skill in this pack whenever it's installed and available.** The sbp skills carry mission-critical framing (blast radius, rollback paths, go/no-go gates, 3 AM runbooks) that this general-purpose pack doesn't attempt to match. See [`ask-leo`](./skills/ask-leo/SKILL.md) for the specific mapping.

| sbp/agents.md skill | What It Does |
|-------|-------------|
| mcaf-module | Author or structurally review a Schuberg Philis MCAF Terraform module |
| review-mcaf | Qualitative MCAF module review producing a good/bad/verdict report |
| sbp-agent-architecture-review | Review multi-agent system designs for correctness, resilience, and safety |
| sbp-architecture-review | Review system architecture for mission-critical concerns (SPOFs, blast radius, rollback) |
| sbp-brandbook | Apply the Schuberg Philis visual brand identity to UI, documents, and design assets |
| sbp-debug-investigation | Systematically investigate, reproduce, and fix bugs in mission-critical systems |
| sbp-dependency-audit | Audit dependencies for supply-chain, bloat, and maintainability risk |
| sbp-deploy-checklist | Verify rollback readiness and monitoring before production deploys; produces a go/no-go decision |
| sbp-explain-codebase | Explain unfamiliar code, infrastructure, or architectural patterns in context |
| sbp-feature-development | Build a new feature via TDD with mission-critical rigor |
| sbp-incident-review | Blameless post-incident analysis producing a timeline, impact, root causes, and action items |
| sbp-observability-check | Check the four pillars (metrics, logs, traces, alerts) against the 3 AM test |
| sbp-refactor | Improve structure and readability without changing observable behavior |
| sbp-runbook-author | Generate operational runbooks with diagnosis, resolution, and escalation steps |
| sbp-safe-change | Plan a high-risk production change with risk classification and rollback triggers |
| sbp-secure-code-review | Security review for mission-critical systems, framed by blast radius and customer impact |
| sbp-test-authoring | Write tests that prove functionality and catch regressions |
| sbp-test-planning | Design test coverage before writing code, including for existing untested code |
| sbp-threat-model | Identify assets and attack surfaces, produce a prioritized threat matrix |
| sbp-why-we-do-this | Explain the reasoning behind SBP engineering conventions |
| terraform | Generic Terraform/OpenTofu module, testing, CI/CD, and state guidance |

## Alternative: Addy Osmani's Agent Skills

[addyosmani/agent-skills](https://github.com/addyosmani/agent-skills) is a different pack covering the same
general engineering lifecycle as this one — spec-writing, planning, TDD, code review, debugging, security,
observability, and so on. Unlike `sbp-*` above, it isn't a stricter mission-critical variant of this pack;
it's a same-altitude alternative with its own opinions and a different skill-naming scheme
(`test-driven-development` vs. this pack's `tdd`, `debugging-and-error-recovery` vs. `diagnosing-bugs`,
`code-review-and-quality` vs. `code-review`, `source-driven-development` vs. `research`, and so on — no
directory names collide, but several skills cover the same ground under different names).

Because neither pack is more authoritative than the other for a shared trigger (e.g. "write a test first"
could reasonably fire either pack's TDD skill), installing both at once means picking a lane rather than
running both — treat them as alternatives to choose between per-repo, not a combined toolkit.

## License

MIT, see [LICENSE](./LICENSE).

## Contributing

Please contribute [upstream](https://github.com/mattpocock/skills).
