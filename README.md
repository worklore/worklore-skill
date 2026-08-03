# worklore — stories your agent can do

[**worklore.dev**](https://worklore.dev) is a library of short, honest developer
stories about real work done with AI agents — where every story is also
**executable by your agent**. Read how someone got universal deep links working,
hand the link to your own agent ("apply this to my project"), and when it lands,
your agent reports back: **works for me ✓** — with your GitHub avatar on the
story as verified proof.

Failures are first-class: "my agent couldn't do this after five attempts" is a
publishable story that stays open until someone's story answers it.

This repo contains the **worklore skill** — it teaches your agent to write,
publish, edit, and reproduce stories.

## Install (Claude Code)

```bash
git clone https://github.com/Yahhi/worklore-skill
cp -r worklore-skill/skills/worklore ~/.claude/skills/
```

Restart Claude Code (or open a new session). That's it — no account needed yet;
registration happens in the flow, later, in ~30 seconds.

## Install (Codex)

Codex reads `AGENTS.md`. Append the skill to your global one (or a project's):

```bash
git clone https://github.com/Yahhi/worklore-skill
cat worklore-skill/skills/worklore/SKILL.md >> ~/.codex/AGENTS.md
```

Everything in the skill is plain instructions + `curl` — no Claude-specific
tooling. "worklore, write this up" works the same.

## Install (Cursor, aider, anything else)

The skill is a standard `SKILL.md`: install it however your tool consumes
skills, or paste its contents into your agent's instructions/rules file. The
only requirements are shell access and the ability to fetch URLs.

## How to start sharing stories with your agent

1. **Work normally.** Build things with your agent like you always do.
2. **When something hard finally works, say: "worklore, write this up."**
   Your agent drafts the story from what actually happened in the session —
   the narrative (your voice, honest, friction included) plus the
   machine-readable "Reproduce this" contract.
3. **Review the draft.** This is your part — the agent did the building and
   the drafting; you add the meaning: what hurt, why it mattered, what you'd
   tell the next person. Nothing publishes without your explicit approval,
   and the skill's sanitization rules keep employer internals and secrets out.
4. **First publish → registration.** Your agent shows you a GitHub device code
   ("visit github.com/login/device, enter XXXX-XXXX"). No password, no email —
   worklore only ever sees your public handle and avatar.
5. **You get a URL.** Share it anywhere. When other people's agents reproduce
   your story, their verified "works for me ✓" appears on it — and on your
   author profile: *"my stories worked for N people."*

**Got stuck instead?** Publish the failure: "worklore, write this up as an open
question." What you tried, where it broke, what you need. It stays open until
someone's story answers it — and the site suggests possible answers at publish
time.

**Want to rephrase later?** "worklore, rephrase my story" — or use the ✎ editor
on your own story pages at worklore.dev (sign in with GitHub).

## Reproducing someone else's story

You don't need this skill to reproduce — every story page has a copyable
prompt. But with the skill installed, your agent also knows the house rules:
read the story *with* you before executing (you can read everything your agent
reads — no hidden payloads, ever), run the story's Verify section before
declaring success, and report the honest outcome, including "failed."

## The transparency promise

- Every story has a raw markdown URL; the prompt you paste contains no
  instructions you can't read first.
- Reproduction counts come from GitHub-verified reports — worked, partial,
  and failed are all shown, never hidden.
- Failures stay open until answered. Nothing is quietly deleted.

[Terms](https://worklore.dev/terms.html) ·
[Privacy](https://worklore.dev/privacy.html) · Made by
[@Yahhi](https://github.com/Yahhi) and a very patient agent — the site's first
stories are about building the site itself.
