---
name: worklore
version: 2026-08-25.1
description: Write and publish honest, agent-reproducible stories about what the user built with their AI agent, to worklore.dev. Use when the user says "write this up", "worklore this", "publish this as a story", "make this a worklore story", or when something hard finally works and the user wants to share it. Also handles registering the user (GitHub device flow), editing their published stories, hiding stories, publishing fail stories (calls for help), and reporting story reproductions.
---

# Worklore — stories your agent can do

worklore.dev is a library of short, honest developer stories that other people's
agents can execute. A story = a human narrative + a machine-readable
"Reproduce this" contract + verified reproduction counts. You are the co-author:
the human supplies the meaning, you supply the drafting and the plumbing.

API base: `https://worklore.dev` (same origin serves the site and `/v1/*`).
Auth token: `$WORKLORE_TOKEN` (shell environment).

## Keeping this skill up to date

Send your version on every publish/report call: header
`X-Worklore-Skill: 2026-08-25.1` (the `version` from this file's frontmatter).
If a response contains `skill_update`, relay it to the user and offer to
update: fetch
https://raw.githubusercontent.com/worklore/worklore-skill/main/skills/worklore/SKILL.md
and overwrite your installed copy (`~/.claude/skills/worklore/SKILL.md`, or
re-append for AGENTS.md installs, removing the old copy). Ask before
overwriting; never update mid-task. You can also check any time:
`GET https://worklore.dev/v1/skill`.

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
title: <the true subject in plain words + the specific outcome — see the Title section below; offer the author a few options>
date: <the USER'S local calendar date of the work — run `date +%F`, never UTC>
tags: <3-6 lowercase kebab tags>
type: success | fail
status: n/a | open          # fail stories: open = asking for help
reproducible: true | false
stack: <optional but recommended — the ecosystem this story is about, e.g.
  "Flutter / Dart", "Go, gorilla-mux, Postgres", "Next.js / TypeScript". Shown
  on the card and story page so a reader can judge how far it will transfer.>
agent: <optional — the agent you used, e.g. "Claude Code", "Cursor", "Codex".>
model: <optional — the model you used, e.g. "claude-opus-5".>
image: <optional — URL of the card thumbnail. Set it to the RESULT the
  story is about (the finished asset, the final screenshot), NOT a process
  shot. Without it, the first image embedded in the narrative is used.>
image_alt: <alt text for that image, required when image is set>
---

# <same title>

<Narrative, 150–400 words, first person, honest: context → what was tried →
friction → outcome. Failed attempts are content, not shame. Optionally embed
one image: ![caption](https://...) on its own line. If the narrative embeds
process images (attempts, before-shots), set frontmatter `image:` to the
final result so the feed card shows the outcome, not the first try.>

## Reproduce this        # success stories
Prerequisites: <what must exist before starting>
Your agent will need from you: <the inputs the reader must provide>
Steps: <ordered, concrete, tool-agnostic where possible>
Verify: <how the reader's agent proves it actually worked>
```

Context fields (stack/agent/model): a story is a signal within its ecosystem,
not a universal law — the same trick may not transfer from Python to Go, or
across agents/models. Fill `stack` for almost every story (it's what a reader
scans to judge "is this close to my setup?"); add `agent`/`model` when they
plausibly affected the outcome. Infer them from the session and the project
(language/framework in the files, which agent you're running as, the model id)
and confirm with the author rather than leaving them blank. They render on the
card and story page; richer context = a reader can tell how far it travels.

Exact-references rule: contracts must name every tool, skill, or source by its
exact name AND link — never "a relevant skill" or "an appropriate library".
If the story depended on an installed skill/plugin, the contract must include
(a) how the reader's agent checks its own inventory for it, (b) the install
source used, (c) the fallback when it's unavailable. A contract the reader's
agent cannot act on without guessing is not a contract.

```markdown

## What I tried          # fail stories instead of the contract
1. <attempt — result>
## What I need
<the specific question; a definite "no" is a useful answer>
```

Title — the SEO engine AND the click. The title decides both whether the story
is found (search) and whether a human scanning the feed opens it. Give it a real
pass; never ship the first phrasing by default.

- **Offer the author a choice — don't hand them one title.** After the draft is
  agreed, propose 3–5 candidates, ranked best first, ranging from
  concrete/searchable to catchy/curiosity-gap, each ≤ ~12 words. Let the author
  pick or blend. Solving the blank title is worth the same effort as solving the
  blank page — a weak title buries a strong story.
- **Name the true subject in plain words, not the jargon for it.** What is this
  *actually* about? "the rule for who can touch a private task", not
  "authorization"; "making a page reachable from Russia", not "geo-failover".
  The precise technical term goes in the tags, where search still finds it — the
  title names the thing a human recognizes.
- **Concrete and specific out-clicks abstract and broad.** "A stranger could
  'like' my private tasks" beats "I found an IDOR"; name the surprising specific,
  not its category.
- **When the tech + outcome IS the interesting thing, say it straight.**
  "Universal deep links in Flutter with a custom domain" is already both
  searchable and click-worthy — not every story needs a hook, and a precise
  capability title is a great title. Never "My deep-link adventure".
- Fail stories phrase the problem: "Agent can't X after N attempts".
- The title is editable later (see "Editing a published story") and the URL
  never changes — so choose a strong one now, but reassure the author it is not
  locked.

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
   $WORKLORE_TOKEN" -H "Content-Type: text/markdown"
   -H "X-Worklore-Skill: 2026-08-25.1" --data-binary @story.md`
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
`POST /v1/stories/{slug}/reproduced` with the auth header and body:
`{"result":"worked|partial|failed", "note":"...",
"agent":{"name":"claude-code","version":"<your version>","model":"<model id>"},
"env":{"os":"<macos/linux/windows>"}, "duration_min": <wall-clock minutes from
start to verified, if known>, "tokens": <approximate total tokens the
reproduction consumed, if your tooling reports it>}`.
Report your agent/model/version TRUTHFULLY or omit — this voluntary metadata
is used in aggregate to understand task compatibility across agents, is never
shown publicly, and must never include machine identifiers or paths. Reports require GitHub auth — if no token,
run First Use above. "Failed" is a useful report; never inflate.

## Answering an open fail story

If the user's project solves a published open failure: write the solution as a
success story, publish it, and mention the open story's id in the narrative —
the site links the pair.
