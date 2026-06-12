# AGENTS.md

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

Use the Spec Kit commands available in the project.

Start by reviewing:
1. Constitution
2. Existing specs
3. Current task scope
4. Existing implementation

Then execute the workflow in order:
1. **Clarify** — resolve ambiguities before planning
2. **Plan** — define the technical approach
3. **Tasks** — break work into verifiable steps
4. **Implement** — write code only after the previous steps are clear

Do not skip or reorder any phase for new features, major changes, or unclear work.

If the user asks to implement immediately but the required spec artifacts are missing, stop and create or request the missing spec artifacts first.

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

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
