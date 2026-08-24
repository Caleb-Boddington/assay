# Assay

Reads your own Claude setup, works out what you actually do, then finds skills that fit you and vets each one before recommending it.

![version](https://img.shields.io/badge/version-0.2.0-blueviolet)
![licence](https://img.shields.io/badge/licence-MIT-blue)
![agents](https://img.shields.io/badge/agents-4_per_run-orange)
![status](https://img.shields.io/badge/status-experimental-yellow)

## What it reads

This is first because the skill reads local files **without asking you first**. You should know that before you install it, not after.

Four locations build the profile:

1. `~/.claude/CLAUDE.md`
2. `CLAUDE.md` in the working directory and every parent directory
3. Your Claude Code memory folder, **every `.md` in it at any depth**
4. `README.md` in the working directory

Then, to avoid recommending you something you already run, it reads three more: the folder names in `~/.claude/skills/`, the folder names **and the contents of `installed_plugins.json`** in `~/.claude/plugins/`, and the list of skills the current session already provides. That is seven things, not four. That distinction matters: three of the seven are an inventory of what you run, not a profile of you.

These are paths Claude Code already treats as user context.

**Your profile text goes to the model API.** It is passed to four subagents verbatim, quotes included, because the fit score cannot be checked without the evidence behind it. So this sends your notes to the same provider you are already talking to, and to nowhere else: no telemetry, no analytics, and no server behind this skill.  If a line in your `CLAUDE.md` should not leave your machine, check it before you run this. Full detail in [SECURITY.md](SECURITY.md).

Web searches carry topic keywords only, never a phrase lifted from a private note. Every run names the files it read at the top of its output, so you can see what it based its guesses on.

If none of those files exist, it says so and asks you to describe what you do in one sentence. If only the working-directory ones exist, which happens when you run it inside somebody else's repository, it says that too rather than profiling the repository and calling it you.

## What it exposes you to

A reader deciding whether to install should not have to open `SECURITY.md` to learn any of the following, and most people never do.

**Four agents read text written by strangers, while holding quotes from your notes.** That is the job: they fetch `SKILL.md` files from public repositories and score them. A `SKILL.md` is a prompt, so a hostile one can contain instructions addressed to the agent reading it. The four briefs tell each agent to treat retrieved text as data and to reject anything carrying an embedded instruction. That defence has been tested against exactly one hostile fixture, written by the author, refused once. It is a smoke test. It licenses nothing.

**Those agents are general-purpose.** They inherit every tool available to subagents, which means Bash, Write and Edit alongside WebFetch, plus any MCP server you have connected. Nothing constrains where a fetch goes. Retrieved text and your profile text share a context, so a fetch could in principle be steered somewhere of an attacker's choosing carrying profile text with it. No exploit has been run and none is offered.

**Your session transcript keeps all of it**, in plain text, under `~/.claude/projects/`, which is the same tree the skill read your memory notes from.

**The safety score is a reading, not an analysis.** A candidate scoring well on safety means nothing alarming appeared in the one markdown file that was opened. It does not mean the repository is safe, and a clone brings the whole repository. Read anything you install yourself.

## What it will not pass on

Item 3 above reads your whole memory tree, which on a real vault is most of what you have written down. So the skill classifies what it reads and withholds the categories a recommendation cannot use: credentials, account and reference numbers, government identifiers, other people's names and anything said about them, health and legal and financial matters, and addresses and phone numbers. Files whose name indicates a secret are never opened at all, matched as whole words and each skip named in the output so you can catch a false positive.

It can afford to withhold all of that because none of it scores. The fit row asks what you do repeatedly, and a policy number is not a recurring task.

**Nothing enforces this.** It is an instruction to a model and it can fail in either direction. Keep secrets out of files you let an agent read, which is worth doing whether or not you run this.

## What a run costs

Roughly 678,000 subagent tokens, 182 tool calls, and about sixteen minutes for the slowest agent. Measured once, on 19 August 2026, on one profile, on one machine, and across five agents rather than the four the skill now caps at.

That is a measurement, not a price. Your run will differ. It is here because a fan-out of four agents is not free and you should see the order of magnitude before you start, not after.

## The problem

Skill directories let you search by task. You already know your task. What you do not know is which of several hundred published skills is worth the context it costs, which one runs a script you never read, and which one solves a problem you do not have.

Searching by task also means you only ever find what you already thought to look for. Nobody searches for the skill that would have saved them four hours a week, because they do not know it exists.

So this searches by *you* instead.

## Install

```bash
git clone https://github.com/Caleb-Boddington/assay.git ~/.claude/skills/assay
```

Windows:

```powershell
git clone https://github.com/Caleb-Boddington/assay.git $env:USERPROFILE\.claude\skills\assay
```

Then restart Claude Code. `SKILL.md` sits at the root and reads `references/rubric.md` and `references/agent-prompts.md` as it runs.

## Usage

```
What skills should I be using?
```

Or narrow it:

```
/assay security
```

## How it works

1. Reads the four locations above and builds a profile: what you do, what you use, what comes up repeatedly.
2. Lists what you already have installed, so nothing gets recommended twice.
3. Fans out to four agents, one per source: Anthropic's official repository, the plugin marketplaces, community lists, and loose GitHub repositories.
4. Each agent screens candidates on their descriptions, reads every shortlisted one's actual `SKILL.md` in full, scores it, and returns finished blocks only. Raw files never reach your main context, which is the whole reason for fanning out.
5. Reports one block per recommendation, with the drawbacks named.

Four agents is a cap, not a starting point. One agent per candidate would scale with whatever the search turned up and spawn thirty agents on somebody who asked a simple question.

## The rubric

Five rows, 1 to 5 each, out of 25. Full detail in `references/rubric.md`.

| Row | Question |
|---|---|
| 1 | Does it do something Claude will not already do unprompted? |
| 2 | Is the trigger narrow and specific? |
| 3 | Does it fit **this** person? |
| 4 | Context cost against payoff |
| 5 | Safety and dependencies |

**Auto-reject if row 1, 2, 3 scores under 3, or if row 5 scores 1**, whatever the total. The default answer is "do not install".

Row 5 gates for a reason. Without it only the first three rows reject, so a candidate scoring 5,5,5,5,1 totals 21 and survives: the rubric can spot a skill trying to steer the agent reading it, score it 1 as instructed, and recommend it anyway.

**The verdict comes off the total.** INSTALL at 20 or above, TRIAL from 16 to 19, REJECT at 15 or below or on any auto-reject. Anything at 20 or above that falls under 20 once row 3 is discounted to 3 is capped at TRIAL, because fit is the row most likely to be scored generously. Every TRIAL states what would settle it and when to check.

The bands matter more than they look. Without them TRIAL is a word in the output with no definition behind it, and scoring goes non-monotonic: a 22/25 lands on TRIAL while three separate 21/25 entries land on INSTALL.

**Auto-reject if it will not run on your operating system**, whatever it scores. This sits outside the five rows, because a great skill you cannot run is worth nothing. A lot of published skills quietly assume macOS or Linux.

Maintenance is deliberately not scored. A skill is a text file, not a library. It has no dependencies to rot and no patches to miss, so six quiet months proves nothing except that nobody needed to change it. The last commit date is reported as a fact and weighed by you.

## What it will not do

**It does not install anything.** You get a clone URL and a verdict. Installing on your behalf, from a list generated by reading your private files, without asking, is too much for one command.

**It does not vet connectors.** A connector, or MCP server, is an external service Claude talks to. It holds credentials and moves your data through a third party. Vetting one means reading a privacy policy and an OAuth scope list, not a markdown file. A score that treats "this text file is safe" and "this company can read your Gmail" as the same measurement is a bad score. Connectors get at most an unscored mention, clearly labelled.

**It does not score anything it has not opened.** Candidates are screened on their descriptions to a shortlist, then every shortlisted one is read in full. No scoring from repository names, README claims, badges or star counts, and the output says which candidates were screened out without being opened.

## Known limitations

A profile built from four files is a thin read of a person. It misses anything you have not written down, which for most people is most things.

The search only sees public GitHub and the marketplaces. Anything private, internal or newly published is invisible.

Row 3, fit, is the most subjective row and the one most likely to be scored generously, because everything looks relevant if you squint. Treat a high total resting mainly on row 3 with suspicion.

Four agents cost real tokens. A run is not free, and the skill announces the fan-out before it starts so you can stop it.

It has been run against one profile. The fit scoring is the part most likely to disappoint on a profile unlike that one.

GitHub rate-limits unauthenticated API access at sixty requests an hour. That budget is per IP address, not per agent, so four agents running at once share it and get roughly fifteen requests each, against 5,000 when authenticated. It binds the searching, not the reading: fetching a shortlisted file is a CDN request and does not draw on it. Install and authenticate `gh` before running, or expect a smaller pool. It is tempting to claim this limit makes reading every candidate impossible. That is wrong twice over, and the reasoning is in [references/rubric.md](references/rubric.md).

**The run does not tell you what fraction of the pool it reached.** The instruction to report screened-against-opened exists in the prose and has no line in the output template to land in, so it has never appeared in a run.

## The name

There is a collision, and it is better heard here than found later. [assay.tools](https://assay.tools) is an existing product describing itself as the quality layer for agentic software, and it scores packages on agent-friendliness and security. Same verb, same object, adjacent market. This project is unaffiliated with it, is not a competitor to it, and is a personal skill of a few hundred lines rather than a platform.

The name stayed because the word is the right one: an assay is a test of what something actually contains, as opposed to what its label claims. If the overlap ever causes real confusion, this one is the one that should move.

## Licence

MIT. See [LICENSE.md](LICENSE.md).
