---
name: to-spec
description: Use when the user wants to turn the current conversation or a brainstormed design into a spec with scrum user stories and publish it to GitHub or GitLab
---

# To Spec

Turn the current conversation context and codebase understanding into a spec. Do NOT interview the user; just synthesize what you already know. This skill usually runs right after `superpowers:brainstorming` has produced a design — it converts that design into a tracker-ready spec. The spec you publish is the natural input for `superpowers:writing-plans`.

**Announce at start:** "I'm using the to-spec skill to turn this conversation into a spec."

## Tracker

Publish to the project's issue tracker. Detect it; do not ask unless ambiguous:

1. **GitLab** — the `origin` remote points at a GitLab host, or the `glab` CLI is authenticated. Publish with `glab issue create`.
2. **GitHub** — the remote points at github.com, or the `gh` CLI is authenticated. Publish with `gh issue create`.
3. **Neither / offline** — write the spec to `docs/specs/<date>-<feature-slug>.md` and tell the user where it is.

If both CLIs are available and the remote does not disambiguate, ask which tracker to use.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations.

3. Write the spec using the template below.

4. Publish it as one issue on the tracker with the feature name as the title:
   - **GitLab:** `glab issue create --title "<feature>" --description "$(cat spec.md)"`. Apply the `ready-for-agent` label if it exists (`-R <repo> -l ready-for-agent`).
   - **GitHub:** `gh issue create --title "<feature>" --body-file spec.md`. Apply the `ready-for-agent` label if it exists (`-l ready-for-agent`); if the label is missing, create it (`gh label create ready-for-agent`) or skip silently.
   - Report the issue URL to the user.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
