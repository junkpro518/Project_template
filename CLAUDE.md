# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Spec-First Development

**No feature code without a spec.**

Use the Spec Kit workflow available in this project.

Before implementing any new feature, major refactor, database change, API change, UI flow, or architectural decision, the following must exist:
1. Spec
2. Clarification answers, if there are ambiguities
3. Technical plan
4. Task breakdown

Treat the spec files as the single source of truth.

Any decision not covered in the specs must be handled in one of two ways:
- Ask the user before proceeding, when the decision affects product behavior, architecture, data model, security, cost, or user experience.
- Document it under an **Assumptions** section, when it is a minor implementation detail.

Small non-behavioral changes may be made without a full spec, such as:
- Fixing typos
- Formatting
- Renaming comments
- Updating documentation wording
- Running tests
- Fixing clearly broken imports caused by recent changes

These exceptions must not change product behavior, database schema, API contracts, security rules, or architecture.

## 6. Workflow Compliance

**Follow the full workflow. Skip nothing for meaningful work.**

Before starting any work, verify that Spec Kit is installed in the project. If it is not installed, stop and notify the user before proceeding.
https://github.com/github/spec-kit.git

Spec Kit must always be kept on the latest available update. Before starting any work that depends on Spec Kit, run `specify self upgrade` first. If the update cannot be verified or completed, stop and notify the user before proceeding.

Use the Spec Kit skills available in the project.

Start by reviewing:
1. Constitution
2. Existing specs
3. Current task scope
4. Existing implementation

Before any feature work, the project must have:
- **Constitution** — the project principles → `/speckit-constitution`. If `.specify/memory/constitution.md` is still the unfilled template, stop and create it first.
- **Converge** — when entering a codebase that already has code but no specs, run `/speckit-converge` once so the existing work is captured as tasks before anything new is planned.

Then execute the workflow in order:
1. **Specify** — create the feature spec → `/speckit-specify`
2. **Clarify** — resolve ambiguities before planning → `/speckit-clarify`
3. **Plan** — define the technical approach → `/speckit-plan`
4. **Agent Context Update** — sync the chosen stack into the agent context file, immediately after planning → `/speckit-agent-context-update`
5. **Checklist** — validate that requirements are complete, clear, and consistent → `/speckit-checklist`
6. **Tasks** — break work into verifiable steps → `/speckit-tasks`. For a feature with a UI, include one e2e test task per user flow (Section 11).
7. **Tasks to Issues** — publish the task breakdown as GitHub issues, immediately after tasks → `/speckit-github-taskstoissues` (the `github` extension; the core `/speckit-taskstoissues` is deprecated)
8. **Analyze** — cross-artifact consistency check before writing code → `/speckit-analyze`
9. **Implement** — write code only after the previous steps are clear → `/speckit-implement`. For a feature with a UI, run its e2e tests after the security review (Section 11).
10. **Converge** — assess the code against the spec and append any unbuilt work as tasks → `/speckit-converge`. A feature with a UI is not complete while its e2e tests are missing or failing (Section 11).

Every step is mandatory. No step may be skipped, reordered, or merged into another for new features, major changes, or unclear work. There are no optional steps.

A step that produces no findings still counts as executed. When clarify finds no ambiguities, or analyze finds no inconsistencies, record that result and move to the next step — running a step and finding nothing is not the same as skipping it.

Converge may append new tasks. When it does, return to step 9, implement them, then converge again. The feature is complete when converge finds nothing remaining.

### Security review after every coding phase

«تُستخدم بعد كل مرحلة من مراحل كتابة الكود» — the `security-audit` skill (`.claude/skills/security-audit`, from Cloudflare) runs after every phase that writes code: each `/speckit-implement` run and each implement → converge loop.
- Use its **guidance mode**: a focused review of the code that phase added or changed — trust boundaries, input handling, auth, secrets, data isolation. Fix confirmed findings before running converge; record "no findings" when there are none (the step still counts as executed).
- Run the **complete six-phase audit** only when the user asks for it or before a release: it launches many agents and costs much more.
- Follow the skill's execution-safety rules as written: never probe deployed services, production data or paid APIs; execute target code only in a sandbox.

**Note on `security-audit`:**
- Run the skill with a model that accepts defensive code review and does not refuse the request. Examples mentioned by Ahmed Nagdy in his video: GLM 5.3 and Kimi.
- If the current model refuses, do not bypass or work around the refusal. Switch the model for this task only, and tell the user.
- Review only the user's own code.

If the user asks to implement immediately but the required spec artifacts are missing, stop and create or request the missing spec artifacts first.

### Always guide the user through the workflow

The user is not a developer and reads mostly from a phone. Throughout the work, tell the user in simple Arabic:
- **Where we are:** the current step out of the 10 (e.g. «الخطوة 3 من 10: Plan») and what was just completed.
- **What this step is for** and what it produces, in one or two short sentences.
- **How to use it:** the exact command to run next (e.g. `/speckit-clarify`), what to give it, and what the user needs to decide or approve.
- **What comes next** after this step.

Do this at the start and end of every step, and whenever the user seems lost. The same applies to the extension commands when used: `assess` (idea evaluation before specify) and `bug` (assess → fix → test), and to the e2e check (Section 11). Never assume the user remembers the steps or the command names.

## 7. Claude Code Plugins

After the project structure, architecture, and required tools are defined, Claude Code must review the official Claude plugins directory:
https://github.com/anthropics/claude-plugins-official

Claude Code must only install plugins that directly match the approved project tools and workflow. Do not install plugins speculatively.

Before installing, updating, or using a plugin, verify that it is trusted, relevant, and documented in the project plan.

## 8. Instruction Sync: CLAUDE.md ↔ AGENTS.md

`CLAUDE.md` and `AGENTS.md` must stay aligned on core project rules.

Core rules include:
- Project overview
- Architecture decisions
- Spec-first workflow
- Testing commands
- Security rules
- Code style
- Deployment rules
- Prohibited actions

Tool-specific instructions may differ when necessary.

When changing one file, review the other file and update any shared rules so both agents follow the same project standards.

## 9. Build on What Exists

**Reuse before reinventing. Study prior art before designing.**

- Study how established products solve the problem before designing a solution. Adopt their proven patterns and conventions rather than inventing an approach from scratch.
- Prefer established, well-maintained libraries when they reduce overall complexity or improve reliability. Do not reimplement common functionality without a clear reason.
- Lean on the dependencies already in the project before writing your own implementation or adding packages. Do not assume a library lacks a capability without checking its documentation and types.

## 10. Incremental Delivery, Durable Architecture

**Ship something that works, then grow it. Decide the shape for the long term.**

- Grow the system in layers. Start from the smallest version that works end to end, and add each new capability on top of a product that already works. Never trade a working product for unfinished complexity.
- Make architectural decisions for the long term. Do not accept a stopgap that only works for now and is meant to be replaced later.

This does not override Section 2. Simplicity governs how a component is implemented; long-term thinking governs the boundaries, contracts, and data shapes between components.

## 11. End-to-End Testing (e2e)

**A feature with a UI is not done until an end-to-end test proves it works in the running app.**

- **Framework:** [e2e](https://github.com/tester-army/e2e.git): agentic end-to-end testing for web and mobile apps. You describe a goal in natural language, an agent drives the app, and exact assertions check the result. Apache-2.0, still pre-1.0 (APIs can change between minor releases).
- **Skill:** use the `e2e` skill in `.claude/skills/e2e` (copied from the source repo; the pinned commit and the review note are in its `SOURCE.md`). When the installed package differs from the copy, prefer `npx e2e guide <topic>`.
- **When it applies:** any feature whose spec has a user-facing flow in a web or mobile UI. Backend-only, CLI, library, documentation, and non-behavioral changes (the Section 5 exceptions) do not need it.
- **Where it sits in the workflow:** Tasks (step 6) includes one e2e test task per user flow, written from the spec's acceptance scenarios. Implement (step 9) runs those tests after the security review. Converge (step 10) treats a feature with a UI as incomplete while its e2e tests are missing or failing.
- **Setup:** if the project has no e2e yet, ask the user before adding it, then follow the skill's `setup` reference (`npx e2e init`). Never add another test framework beside it without asking.
- **Telemetry off:** the e2e CLI sends anonymous usage data. Every project sets `E2E_TELEMETRY_DISABLED=1` (it is already in `.env.example`).
- **Keys and cost:** agent steps need a model and its authentication, so keys live only in `.env` and in the e2e config's `credentials` and `secrets`, never in test code. Tests without agent steps need no model. Keep the replay cache on while fixing a test; use `--no-cache` only for the final confirming run.
- **Safety:** run e2e only against the project's own app, locally or on staging, with test accounts. Never against production, real user data, or paid services.
- **Feedback to the e2e team:** never run `npx e2e feedback` without `--dry-run` first and the user's agreement. Never file an issue on their repository yourself.
- **Guide the user:** as in Section 6, tell the user in simple Arabic when this check starts and what it found.

## 12. Subagents and Model Choice

**Cheap models on cheap work. Keep the judgment calls on the main model.**

Source: Anthropic, "Which Claude model should you build with?" (https://claude.dev/blog/which-claude-model-should-you-build-with/, 2026-11-09). The lineup and prices change, so re-read the guide before changing these rules.

**What a subagent is for**
- A subagent runs in its own context window and hands back a summary. Its tokens bill at the rate of the model it runs on.
- Use it for verbose work (reading many files, logs, test output) so the main model's context stays lean. A lean context matters more the bigger the main model is.

**Which model for which work**

| Work | Model |
|---|---|
| Search files, read logs, run tests, summarize | Haiku 5.5 or Sonnet 5.5 |
| High-volume classification, routing, extraction | Haiku 5.5 |
| Code edits and judgment calls (the main session) | Opus 5.5 at `medium` (the default) |
| Hardest work, major refactors, leading many subagents | Fable 5.2 |

- Give a small model only work where a wrong answer is cheap to spot. If it misreads a search result, the main model opens the wrong file and pays for it.
- The main model keeps the judgment calls and checks what a subagent returns before acting on it.

**How to set the model**
- In the subagent definition (`.claude/agents/<name>.md`), set `model: haiku` or `model: sonnet` in the frontmatter. `CLAUDE_CODE_SUBAGENT_MODEL` sets the default for subagents that don't specify one.
- Example, `.claude/agents/log-reader.md`:
  ```markdown
  ---
  name: log-reader
  description: Reads test output and logs, and reports each failing case. Use after a test run.
  tools: Read, Grep, Glob, Bash
  model: haiku
  ---
  Read the logs you're given. For each failure, report the file, the test name and the first error line. Don't edit any files.
  ```
- The built-in `Explore` subagent runs on the session's model, not always on Haiku. To keep exploration cheap, define your own subagent named `Explore` with `model: haiku`.
- The `haiku` alias needs Claude Code v2.1.293 or later to resolve to Haiku 5.5 on the Anthropic API (on Bedrock, Google Cloud and Foundry it still resolves to Haiku 4.5). Pin it with `ANTHROPIC_DEFAULT_HAIKU_MODEL` if needed.
- Do not make `opusplan` the default without measuring it: it puts the code edits on Sonnet, the reverse of the rule above.

**Bulk work bigger than one context window**
- A strong lead splits the task, hands the pieces to many cheaper workers in parallel (the guide's test used up to 25 Sonnet 5.5 workers), and merges the results. In that test it cost about a third less than the lead alone, with about the same score.
- Skip this for one chain of dependent steps, or work that fits in one context. There the planning and merging cost more than they save.

**When a model isn't good enough, in this order**
1. Give it a way to check its own work: a test, a build, or a script that calls the endpoint.
2. Raise effort one level.
3. Move up one model, starting at a lower effort than before. Past `high`, one more effort level can cost as much as a bigger model, so compare both.

**Switching models**
- The prompt cache belongs to the model that wrote it. Switch at a natural break, run `/compact` first, or start a fresh session with a short written plan.
- A move down a tier loses the earlier reasoning. Moving from Fable 5.2 to any other model drops it.

**Tell the user**
- As in Section 6, when launching subagents tell the user in simple Arabic: how many, which model, and why. What comes back from a subagent is translated to Arabic before it reaches the user.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## Shared toolbox (server)

Before adding skills or tools to this project, read `TOOLBOX.md`: pick only what this project needs from `~/toolbox` (`kit search`, `kit add`). Deploy from `docker-compose.example.yml` — memory limit, login in front of public sites, secrets only in `.env`.

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->
