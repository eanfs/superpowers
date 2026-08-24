---
name: to-tickets
description: Use after superpowers:writing-plans has produced an implementation plan, to publish each plan task verbatim as a ticket issue on GitHub or GitLab with blocking edges from the plan's task order
---

# To Tickets

Turn a superpowers plan into **tickets**: one issue per plan task, carrying the task's content **verbatim**. The plan (`docs/superpowers/plans/YYYY-MM-DD-<feature>.md`, produced by `superpowers:writing-plans`) is the single source of truth — never rewrite, merge, or re-decompose its tasks.

**Announce at start:** "I'm using the to-tickets skill to publish the plan tasks as tickets."

Run this after `superpowers:writing-plans` (and optionally after `superpowers:to-spec` published the plan's story — that story issue is the natural parent).

## Tracker

Publish to the project's issue tracker. Detect it; do not ask unless ambiguous:

1. **GitLab** — the `origin` remote points at a GitLab host, or the `glab` CLI is authenticated.
2. **GitHub** — the remote points at github.com, or the `gh` CLI is authenticated.
3. **Neither / offline** — local files (see step 4).

If both CLIs are available and the remote does not disambiguate, ask which tracker to use.

## Scrum labels

Apply the project's Scrum label vocabulary (the team's board convention; English set shown):

| Kind | Labels | Rule |
|------|--------|------|
| Status (board column) | `S1-Todo` `S2-InProgress` `S3-Review` `S4-Done` | exactly one, always the current state; tickets are born `S1-Todo` |
| Type | `type::story` `type::task` `type::bug` `type::spike` `type::test-case` | the parent story issue is `type::story`; each ticket is `type::task` |
| Priority | `P0-Urgent` `P1-High` `P2-Medium` `P3-Low` | inherit from the parent story unless the user says otherwise |
| Story points | `SP-1` `SP-2` `SP-3` `SP-5` `SP-8` `SP-13` | the parent story carries the estimate; do not SP-label individual tickets unless the user asks |
| Module grouping | `feat::<module>` (e.g. `feat::core-engine`) | inherit from the parent story |

Label creation must be idempotent: `gh label create "<name>" --force` (create-or-update), or for glab check `glab label list` first and treat "already exists" as success. Never fall back to publishing without a mandated label. Do NOT set a Sprint milestone: tickets stay in the backlog; sprint assignment happens in Sprint Planning.

## Process

### 1. Read the plan

Take the newest plan under `docs/superpowers/plans/`, or the plan path the user passes as an argument. Read it in full: goal, context, file structure, and every task with its code blocks, test commands, and verification steps.

### 2. Map tasks to tickets — verbatim

One plan task = one ticket. The mapping is mechanical, not creative:

- **Ticket title** = the plan task's heading, verbatim.
- **Ticket body** = the plan task's complete content, copied **word-for-word**: description, code blocks, file paths, test commands, verification steps — everything under that task's heading up to the next task. Do not summarize, translate, trim, or re-order. The implementer of the ticket must need nothing that is in the plan but not in the ticket.
- **Blocking edges** = the plan's task order: a task that depends on an earlier task's files or outputs is blocked by that task's ticket. A plan written in strict sequence yields a linear chain; respect explicit dependencies stated in the plan over position when they differ.
- **Acceptance criteria** = the plan task's own verification steps and test commands, as checkboxes — copied, not paraphrased.

If a single task is too large for one issue, do NOT split it yourself: tell the user and ask whether to revise the plan (via `superpowers:writing-plans`) instead.

### 3. Confirm with the user

Present the mapping as a numbered list: ticket number, title (verbatim plan heading), blocked-by set. Ask:

- One-to-one mapping correct? (every task has exactly one ticket)
- Blocking edges correct? (matches the plan's dependency statements)

Iterate until the user approves. Do not offer to merge or split tasks — that would diverge from the plan; route such changes back to `superpowers:writing-plans`.

### 4. Publish the tickets to the tracker

Publish the approved tickets in dependency order (blockers first) so each ticket's blocking edges can reference real identifiers.

**Parent first.** If the source plan's story was published (e.g. by `superpowers:to-spec`), that existing story issue is the parent — use it as-is. Otherwise publish the plan itself as the parent issue (full plan body on GitHub/GitLab; title + link locally) labeled `type::story` + `S1-Todo`. Reference the parent in every ticket. Do NOT close or modify any pre-existing parent issue.

- **GitHub** → `gh issue create --title ... --body-file ...` per ticket, labeled `type::task` + `S1-Todo` (plus inherited `P*`/`feat::`); create missing labels with `gh label create "<name>" --force` first. To attach a ticket to the parent as a sub-issue (API-only): `gh api repos/$OWNER/$REPO/issues/<parent-number>/sub_issues -F sub_issue_id=$(gh api repos/$OWNER/$REPO/issues/<ticket-number> --jq .id)` — note the sub-issue API needs the child's **database ID**, not its issue number. Otherwise reference the parent and blockers in the "Blocked by" section as `#N`.
- **GitLab** → `glab issue create --title ... --description ...` per ticket, `-l "type::task" -l "S1-Todo"` (plus inherited labels); check `glab label list` and create missing labels with `glab label create -n "<name>"`. Prefer the native blocked-by relationship: `glab api "projects/:id/issues/<ticket-iid>/links" -f target_issue_iid=<blocker-iid> -f link_type=is_blocked_by` (the current issue is blocked by the target); otherwise list blockers in the "Blocked by" section.
- **Local files** → write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered `01` up in plan task order, plus a `00-parent.md` holding the plan (or a link to the story issue). Each file's "Blocked by" lists the numbers/titles it depends on. One ticket per file, never a single combined file.

Tickets are born grabbable: `S1-Todo` means an agent or a developer can pick one off the frontier. Work the **frontier**: any ticket whose blockers are all done. For a strictly sequential plan that means top to bottom.

<local-ticket-template>

# <NN>: <verbatim plan task heading>

**What to build:** the plan task's complete content, copied word-for-word from the plan — description, code, test commands, verification steps. Everything the implementer needs.

**Blocked by:** the numbers/titles of the tickets that gate this one, or "None (can start immediately)".

**Status:** S1-Todo

- [ ] Verbatim acceptance criterion from the plan's verification step
- [ ] Verbatim test command passing, from the plan

</local-ticket-template>

<issue-template>

## Parent

A reference to the parent story issue on the tracker.

## What to build

The plan task's complete content, copied word-for-word from the plan — description, code, test commands, verification steps.

## Acceptance criteria

- [ ] Verbatim criterion from the plan's verification steps
- [ ] Verbatim test command passing, from the plan

## Blocked by

- A reference to each blocking ticket, or "None (can start immediately)".

</issue-template>

The verbatim rule overrides the usual "avoid file paths and code snippets" convention: the plan is the authority and tickets carry its content unchanged, so a ticket is self-contained even for an implementer who never sees the plan file.
