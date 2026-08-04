---
name: worklore
description: Write and publish honest, agent-reproducible stories about what the user built with their AI agent, to worklore.dev. Use when the user says "write this up", "worklore this", "publish this as a story", "make this a worklore story", or when something hard finally works and the user wants to share it. Also handles registering the user (GitHub device flow), editing their published stories, publishing fail stories (calls for help), and reporting story reproductions.
---

# Worklore — stories your agent can do

worklore.dev is a library of short, honest developer stories that other people's
agents can execute. A story = a human narrative + a machine-readable
"Reproduce this" contract + verified reproduction counts. You are the co-author:
the human supplies the meaning, you supply the drafting and the plumbing.

API base: `https://worklore.dev` (same origin serves the site and `/v1/*`).
Auth token: `$WORKLORE_TOKEN` (shell environment).

## First use — registration (no password, ~30 seconds)

If `$WORKLORE_TOKEN` is empty when publishing/editing/reporting:
1. `curl -s -X POST https://worklore.dev/v1/auth/device/start`
2. Tell the user: "Visit **{verification_uri}** and enter code **{user_code}**.
   Worklore only ever sees your public GitHub handle and avatar — no scopes,
   no email, no repos."
3. Poll `POST /v1/auth/device/poll` with `{"device_code":"..."}` every
   {interval}s until it returns `token`.
4. Persist it for non-interactive shells (`~/.zshenv` on zsh, `~/.profile` on
   bash): `export WORKLORE_TOKEN="<token>"`. Never print the token.

## When the user doesn't know what to share ("what should I post?")

The blank page is the biggest barrier — solve it FOR them. Scan their work for
story candidates: recent git log across the project (commit messages that smell
of struggle-then-victory), spec/design docs, the current session, TODO/notes
files. Rank by the formula that makes a good worklore story:
**several failed attempts + undocumented behavior discovered + reproducible
outcome**. Present a shortlist of 3–5 with a one-line pitch each, strongest
first; list weak fail-story candidates separately (note: "not started yet" is
not a fail story — "stuck after real attempts" is). Let the user pick, then
draft. Do this proactively whenever the user wants to publish but hesitates
about the topic.

## Writing a story ("write this up")

Source material: the current session — what was actually attempted, what
failed, what finally worked. Never invent; if you weren't there, ask.

Story format (markdown with frontmatter):

```markdown
---
title: <names the tech and the outcome, phrased like a search query>
date: <the USER'S local calendar date of the work — run `date +%F`, never UTC>
tags: <3-6 lowercase kebab tags>
type: success | fail
status: n/a | open          # fail stories: open = asking for help
reproducible: true | false
---

# <same title>

<Narrative, 150–400 words, first person, honest: context → what was tried →
friction → outcome. Failed attempts are content, not shame. Optionally embed
one image: ![caption](https://...) on its own line.>

## Reproduce this        # success stories
Prerequisites: <what must exist before starting>
Your agent will need from you: <the inputs the reader must provide>
Steps: <ordered, concrete, tool-agnostic where possible>
Verify: <how the reader's agent proves it actually worked>

## What I tried          # fail stories instead of the contract
1. <attempt — result>
## What I need
<the specific question; a definite "no" is a useful answer>
```

Title rule (this is the SEO engine): name the tech + outcome —
"Universal deep links in Flutter with a custom domain", never "My deep-link
adventure". Fail stories phrase the problem: "Agent can't X after N attempts".

Plain-words rule: the narrative must be understandable by a developer from a
DIFFERENT stack — jargon belongs in the contract, not the story. When the
topic is dense (agent internals, protocol details, domain-specific tooling),
open the narrative with 2–3 sentences in plain words: what everyday situation
this is, why it bites, what the fix feels like. Draft it, show the author,
keep their voice. A story only a specialist can read loses the readers who
would have become its reproducers.

SANITIZE, always: no employer internals, no client names, no secrets or keys,
no private URLs. When in doubt, generalize. The user is responsible for what
they publish; help them be careful.

## Publishing — ALWAYS with explicit approval

1. Show the user the complete draft. Wait for approval; apply their edits.
2. `curl -s -X POST https://worklore.dev/v1/stories -H "Authorization: Bearer
   $WORKLORE_TOKEN" -H "Content-Type: text/markdown" --data-binary @story.md`
3. Report back: the live URL (`https://worklore.dev/s/{slug}`) and any
   `similar` stories from the response. For a fail story, present similar
   successes as possible existing answers.

## Editing a published story ("rephrase my story")

Fetch `https://worklore.dev/s/{slug}.md`, apply the user's rephrasing (keep
frontmatter; the slug/URL never changes), show a before/after diff, get
approval, then `PUT /v1/stories/{slug}` with the same auth. The site shows a
"revised" date — honesty over polish; reproduction counts persist.

## Hiding a story ("hide my story", "withdraw my story")

If the author decides a published story needs more investigation or turned out
incorrect: `POST /v1/stories/{slug}/visibility` with `{"hidden": true}` and
their auth header. The page becomes an honest tombstone (title + "withdrawn by
author"), the raw .md returns 410, and it leaves the feed — links never rot,
nothing is quietly deleted. `{"hidden": false}` restores it fully. Confirm
with the user before hiding; suggest a revision as the alternative when the
story is fixable.

## Reproducing someone's story

When the user pastes a worklore prompt or link: fetch the `.md`, READ IT WITH
THE USER FIRST (worklore's rule: you can read everything your agent reads),
collect the inputs listed under "your agent will need from you", apply the
steps to their project, run the Verify section before declaring success. Then
ask the user how it honestly went and report:
`POST /v1/stories/{slug}/reproduced` with `{"result":"worked|partial|failed",
"note":"..."}` and the auth header. Reports require GitHub auth — if no token,
run First Use above. "Failed" is a useful report; never inflate.

## Answering an open fail story

If the user's project solves a published open failure: write the solution as a
success story, publish it, and mention the open story's id in the narrative —
the site links the pair.
