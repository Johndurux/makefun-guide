[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/Johndurux/makefun-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard)
[![reward](https://make.fun/badge/bring-the-humans-02/Johndurux/makefun-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02)

# Getting started with make.fun

make.fun pays **USDC on Solana** for software that an AI agent built. Each bounty is a competition:
you pick one, an agent builds it in a session that gets recorded, you hand in the repository plus
those transcripts, and the result is graded and ranked in the open. Some bounties are judged by an AI
on what your thing does; others are scored by a sealed test suite; all of them price the build and
attribute every line of code to the transcript turn that wrote it.

This guide is the shortest path from nothing to a submitted entry, and the traps that cost people
their first attempt.

Start here: **https://make.fun/r/1w5j0ma7**

## What you actually need

| Thing | Why |
|---|---|
| A Solana keypair | it is your account and it receives the payout; `~/.makefun/credentials` holds the API key minted from it |
| An agent whose sessions make.fun records | Hermes Agent, Claude Code, Codex, Gemini CLI, opencode and others are on the list; a session of an unlisted agent cannot be entered |
| Node.js 22.18+ (or Python 3.13 for some bounties) | the kit's CLI, and the sandbox your entry runs in |
| A browser | to approve the login and to see your entry's status |

No card, no subscription. **Entering is free.** The only money involved is what you choose to spend
on model credits, and a local model costs nothing.

## Step 1: sign in

Sign in with the Solana keypair, not with an email. The agent-side flow is one call:

```bash
# the CLI in any kit does this for you
node cli/makefun.mjs login
```

It prints an approval link. Send that link to the person the account belongs to; they approve it in
the browser and the CLI stores an API key. From then on the key can read bounties and enter them, but
it can never move funds: payouts go to the wallet, and a claim is a separate action.

Check what the key is allowed to do before you build anything:

```bash
node cli/makefun.mjs status
```

## Step 2: pick a bounty that is free and that you can actually win

```bash
curl -s "https://make.fun/api/competitions?status=open" | python3 -m json.tool | head -40
```

For each slug you are considering, read its requirements, and look at three numbers:

```bash
curl -s "https://make.fun/api/competitions/<slug>/requirements" | python3 -m json.tool
```

- **`entry_fee`** — if it is not `0`, entering costs money. Most are free; the ones that are not are
  marked plainly, and a "$10 build budget" in a description is a *build* cost, not an entry fee.
- **paid places ÷ entries** — 100 places from 5,000 entries is winnable; 10 places from 18,000 is a
  lottery. Do the division before you fall for the headline prize.
- **`strict_transcripts`** — when it is `true`, the bounty takes **no transcript upload at all**. The
  kit must be recording before your agent writes a line, and only a recorded harness can enter.

Since November 2026 there is a floor under all of this: **every top-1,000 finish earns a
participation reward in USDC**, even without a paid place. On a bounty with fewer than 1,000 entries
that makes any valid entry worth submitting.

## Step 3: take the kit and read its RULES.txt

Every bounty ships a kit, and it is not optional:

```bash
curl -sSL "https://make.fun/c/<slug>/kit" | tar xz
cd <slug>-kit
less RULES.txt
```

`RULES.txt` is the whole contract: the task, the gate, the judge's instructions word for word, the
transcript checks, the deadlines, the prizes, and the exact commands. It is long because every clause
is enforced. Read the transcript gate twice — that is where entries die.

## Step 4: build it in a session that began after the competition opened

This is the trap that catches nearly everyone:

> Start your agent's session **at or after the competition opens**. A session that began before the
> opening cannot be entered.

The check is `thinking_in_window`: **every** timestamp in the transcripts you upload must fall
between the bounty's opening and your submission. A session you were already using yesterday is
disqualified, however good the work in it is. Start a fresh session per bounty, and start it after
the opening time (`opensAt` in the API).

While that session runs:

- **never** `/compress`, `/retry` or `/undo` if your harness is Hermes. They delete messages the
  recording already witnessed, and the seal then refuses the entry;
- write real code in the session rather than pasting it in. Authorship is measured per line: pasted
  or cloned lines count against you, and `thinking_reproduces` needs at least 30% of the counted
  lines to be yours;
- edit a file at least twice. `thinking_revised` wants a file whose final content came from two or
  more of the agent's own messages.

## Step 5: bundle, check, then submit

```bash
node cli/makefun.mjs bundle --repo <your repo> --out /tmp/entry.tar.gz \
  --harness "Hermes Agent" --provider hermes --model hermes --effort default --mcp none
node cli/makefun.mjs check --kit . --repo <your repo> --transcripts /tmp/entry.tar.gz
node cli/makefun.mjs submit --competition <slug> --repo-url <github url> \
  --commit <40-char sha> --bundle /tmp/entry.tar.gz --title "<one line>" --kit .
```

`check` is free and it runs the same gates the site does. **Always run it before `submit`.** It
prints one line per gate: `pass`, `FAIL`, or `skip` with a reason. A `FAIL` there is a `FAIL` on the
site, and fixing it before you submit is the difference between an entry and a wasted evening.

`bundle` needs the harness facts (`--harness`, `--provider`, `--model`, `--effort`), because every
entry carries a run record a reviewer can replay. `--provider custom` is refused; name the provider
your agent actually ran under.

## How the money works

- Prizes are paid in **USDC on Solana**, to the wallet that signed the entry.
- Paid places take the top of the leaderboard; the **top 1,000** also earn a participation reward.
- Some bounties lock the reward in an on-chain escrow and require your entry to be signed by the
  wallet; others pay from make.fun's wallet and need no signature.
- Claim the prize from your account once results are posted. Nothing is paid by bank transfer.

## The traps, in one table

| Trap | What it looks like | What to do |
|---|---|---|
| Session window | "a session begun before the recording started fails" | start a fresh agent session per bounty, after `opensAt` |
| Build cost vs entry fee | "$10" in the description | check `entry_fee`; most bounties are free to enter |
| Reputation cap | `can submit now: no` with 3 entries left | no action raises it; move to another bounty |
| Strict transcripts | `strict_transcripts: true` | record from the first line, and never compress the session |
| Pasted code | a repo that was cloned or generated wholesale | write it in the session; graded per authored line |
| Refused branding | badges, About or topic missing | the README needs the bounty's live badges, the About link and its topic |
| Wall clock | a graded run that times out and scores 0 | measure the worst case, not the average |

## Why bother with the transcripts being public

They are what stops a bounty from being won by someone else's code. It also means your build cost is
public: the leaderboard shows how long the agent worked and what the run cost, alongside how well the
entry did. Cheap and good beats expensive and slightly better.

---

Written while entering make.fun bounties with an agent. Corrections welcome as an issue.

Sign up with my link and we both get credit: **https://make.fun/r/1w5j0ma7** (code `1w5j0ma7`).