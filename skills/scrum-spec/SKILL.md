---
name: scrum-spec
description: Use after superpowers:writing-plans has produced an implementation plan, to turn that plan into a scrum story (user stories plus test scope) derived entirely from the plan and publish it to GitHub or GitLab
---

# To Spec

Turn a written implementation plan into a scrum story issue. Run this AFTER `superpowers:writing-plans` — the plan (`docs/superpowers/plans/YYYY-MM-DD-<feature>.md`) is the single source of truth: derive the user stories and the testing scope entirely from the plan's content. Do NOT invent scope the plan does not contain; do NOT re-interview the user. Anything the plan leaves ambiguous goes into Further Notes.

**Announce at start:** "I'm using the scrum-spec skill to turn the plan into a story."

The story you publish is the parent issue `superpowers:scrum-tickets` breaks into tickets.

## Tracker

Publish to the project's issue tracker. Detect it; do not ask unless ambiguous:

1. **GitLab** — the `origin` remote points at a GitLab host, or the `glab` CLI is authenticated. Publish with `glab issue create`.
2. **GitHub** — the remote points at github.com, or the `gh` CLI is authenticated. Publish with `gh issue create`.
3. **Neither / offline** — write the story to `docs/superpowers/specs/YYYY-MM-DD-<feature-slug>.md` (the repo's spec location) and tell the user where it is.

If both CLIs are available and the remote does not disambiguate, ask which tracker to use.

## Scrum labels

Apply the project's Scrum label vocabulary (the team's board convention; English set shown):

| Kind | Labels | Rule |
|------|--------|------|
| Status (board column) | `S1-Todo` `S2-InProgress` `S3-Review` `S4-Done` | exactly one, always the current state |
| Type | `type::story` `type::task` `type::bug` `type::spike` `type::test-case` | this skill always publishes `type::story` |
| Priority | `P0-Urgent` `P1-High` `P2-Medium` `P3-Low` | at most one |
| Story points | `SP-1` `SP-2` `SP-3` `SP-5` `SP-8` `SP-13` | exactly one on a story, agreed with the user |
| Module grouping | `feat::<module>` (e.g. `feat::core-engine`) | when the module is determinable |

Label creation must be idempotent: `gh label create "<name>" --force` (create-or-update), or for glab check `glab label list` first and treat "already exists" as success. Never fall back to publishing without a mandated label. Do NOT set a Sprint milestone: issues stay in the backlog; sprint assignment happens in Sprint Planning.

## Process

1. Read the plan in full: goal, context, file structure, every task with its test commands and verification steps. If the user passes a plan path as an argument, use it; otherwise take the newest plan under `docs/superpowers/plans/`.

2. Derive the story content from the plan:
   - **Problem Statement / Solution** — from the plan's goal and context sections, restated from the user's perspective.
   - **User Stories** — walk the plan's task list; each task's deliverable behaviour becomes one or more user stories. Actors come from the plan's context. Merge duplicates; keep the list exhaustive over the plan's scope, but add nothing outside it.
   - **Testing Decisions** — from the plan's own test commands, TDD steps, and verification steps: what gets tested, at which seams the plan tests, and the plan's prior art. The plan is the authority; do not propose new test strategy.
   - **Implementation Decisions** — only decisions the plan already made (modules, interfaces, schema/API changes named in its file structure). Ambiguities and gaps go to Further Notes, flagged for the user.

3. Confirm the seams with the user: the seams at which the plan tests each task (existing seams the plan reuses beat new ones). In the same checkpoint, propose the story's Scrum labels — `SP-*` point estimate (size it from the plan's task count and weight), `P*` priority (if the conversation made it clear), `feat::<module>` grouping (from the plan's file structure) — and take what they confirm.

4. Write the story body to a scratch file (`.scratch/spec.md`), then publish it as one issue with the feature name as the title, labeled `type::story` + `S1-Todo` plus the confirmed labels. Delete the scratch file after publishing:
   - **GitLab:** `glab issue create --title "<feature>" --description "$(cat .scratch/spec.md)" -l "type::story" -l "S1-Todo"`.
   - **GitHub:** `gh issue create --title "<feature>" --body-file .scratch/spec.md -l "type::story" -l "S1-Todo"`.
   - Report the issue URL to the user. That issue is what `scrum-tickets` takes as its parent.

<spec-template>

## Problem Statement

The problem the user is facing, from the user's perspective. Derive from the plan's goal.

## Solution

The solution, from the user's perspective. Derive from the plan's goal and context.

## User Stories

A LONG, numbered list of user stories derived from the plan's tasks. Each in the format:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

Every task in the plan maps to at least one user story; nothing outside the plan appears here.

## Implementation Decisions

Only decisions the plan itself made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications recorded in the plan
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if the plan contains a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from the plan. Trim to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

Derived from the plan's test commands and verification steps:

- Which modules will be tested, per the plan's tasks
- The seams at which the plan tests (from its test commands)
- Prior art for the tests (similar tests the plan references or relies on)

## Out of Scope

Everything outside the plan: the plan's non-goals, plus anything deliberately deferred.

## Further Notes

Ambiguities, gaps, and assumptions found in the plan while deriving this story — flagged for the user, not silently resolved.

</spec-template>
