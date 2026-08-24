---
name: to-tickets
description: Use when the user wants to break a superpowers plan, spec, or the current conversation into tracer-bullet tickets with blocking edges and publish them to GitHub or GitLab
---

# To Tickets

Break a plan, spec, or conversation into a set of **tickets**: tracer-bullet vertical slices, each declaring the tickets that **block** it.

**Announce at start:** "I'm using the to-tickets skill to break this into tickets."

A **superpowers plan** (`docs/superpowers/plans/YYYY-MM-DD-<feature>.md`, produced by `superpowers:writing-plans`) is the richest source: publish the plan itself as the parent issue on the tracker, then every task group becomes a ticket whose acceptance criteria come from the plan's own verification steps.

## Tracker

Publish to the project's issue tracker. Detect it; do not ask unless ambiguous:

1. **GitLab** — the `origin` remote points at a GitLab host, or the `glab` CLI is authenticated.
2. **GitHub** — the remote points at github.com, or the `gh` CLI is authenticated.
3. **Neither / offline** — local files (see step 5).

If both CLIs are available and the remote does not disambiguate, ask which tracker to use.

## Scrum labels

Apply the project's Scrum label vocabulary (the team's board convention, e.g. the GitLab CE label set):

| Kind | Labels | Rule |
|------|--------|------|
| Status (board column) | `S1-待办` `S2-进行中` `S3-评审` `S4-完成` (or the team's English equivalents) | exactly one, always the current state; tickets are born `S1-待办` |
| Type | `type::story` `type::task` `type::bug` `type::spike` `type::test-case` | the parent story/spec issue is `type::story`; each ticket is `type::task` |
| Priority | `P0-紧急` `P1-高` `P2-中` `P3-低` | inherit from the parent story unless the user says otherwise |
| Story points | `SP-1` `SP-2` `SP-3` `SP-5` `SP-8` `SP-13` | the parent story carries the estimate; do not SP-label individual tickets unless the user asks |
| Module grouping | `feat::<module>` (e.g. `feat::核心引擎`) | inherit from the parent story |

If a label is missing on the tracker, create it on the fly (`glab label create` / `gh label create`) or skip silently. Do NOT set a Sprint milestone: tickets stay in the backlog; sprint assignment happens in Sprint Planning.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (a plan path, a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each ticket its **blocking edges**: the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change (rename a column, retype a shared symbol) whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand-contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches cannot land green, that is not a ticket problem - surface it to the user as a design problem.

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: short descriptive name
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the end-to-end behaviour this ticket makes work

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct: does each ticket only depend on tickets that genuinely gate it?
- Should any tickets be merged or split further?

Iterate until the user approves the breakdown.

### 5. Publish the tickets to the tracker

Publish the approved tickets in dependency order (blockers first) so each ticket's blocking edges can reference real identifiers.

**Parent first.** If the source was a superpowers plan or an existing issue/spec, publish it as the parent issue (full plan body on GitHub/GitLab; title + link locally) and reference it in every ticket. Label the parent `type::story` + `S1-待办` (plus its `SP-*`/`P*`/`feat::` labels if known). Do NOT close or modify any pre-existing parent issue.

- **GitHub** → `gh issue create --title ... --body-file ...` per ticket, labeled `type::task` + `S1-待办` (plus inherited `P*`/`feat::`); create missing labels with `gh label create "<name>"` first. Where the platform offers a native relationship, prefer it: attach each ticket to the parent as a sub-issue; otherwise list blockers in the "Blocked by" section as issue references (`#N`).
- **GitLab** → `glab issue create --title ... --description ...` per ticket, `-l "type::task" -l "S1-待办"` (plus inherited labels); create missing labels with `glab label create -n "<name>"` first. Prefer the native blocked-by relationship: `glab api "projects/:id/issues/<ticket-iid>/links" -f target_issue_iid=<blocker-iid> -f link_type=is_blocked_by` (the current issue is blocked by the target); otherwise list blockers in the "Blocked by" section.
- **Local files** → write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first), plus a `00-parent.md` holding the plan/spec. Each file's "Blocked by" lists the numbers/titles it depends on. Use the per-ticket file template below: one ticket per file, never a single combined file.

Tickets are born grabbable: `S1-待办` means an agent or a developer can pick one off the frontier. Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain that means top to bottom.

<local-ticket-template>

# <NN>: <Ticket title>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective, not a layer-by-layer implementation list.

**Blocked by:** the numbers/titles of the tickets that gate this one, or "None (can start immediately)".

**Status:** S1-待办

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

</local-ticket-template>

<issue-template>

## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective, not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- A reference to each blocking ticket, or "None (can start immediately)".

</issue-template>

In either form, avoid specific file paths or code snippets: they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.
