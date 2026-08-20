---
name: assay
description: Reads your own Claude configuration and notes, works out what you actually do, then searches GitHub and the plugin marketplaces for skills and plugins that fit you and vets each one before recommending it. Nothing is scored without its file being opened and read, and each survivor is scored out of 25 and given a verdict of INSTALL, TRIAL or REJECT with its drawbacks named. Use when asked what skills are worth having, which skills would suit me, what should I install, or to find skills for what I do. Reads local files without asking first, names them before it starts, and passes the profile it builds to four subagents.
argument-hint: "[a domain to narrow to, optional]"
allowed-tools: Agent, Read, Glob, Grep, WebSearch, WebFetch, AskUserQuestion
---

# Assay

Most skill directories let you search by task. You already know your task. What you do not know is which of four hundred skills is worth the context it costs, which one quietly runs a script you did not read, and which one solves a problem you do not have.

This reads your own setup first, then goes looking. The search terms come from you.

## What this reads, before anything else

**Say this before the first read, not after it.** This skill fires on phrases like "what should I install", so most people meet it without having chosen it, and a disclosure printed after the reads have happened is a receipt rather than a warning. Open with one line naming the four profile locations and saying they are about to be read, then read them. It costs a sentence.

Four locations build the profile:

1. `~/.claude/CLAUDE.md`
2. `CLAUDE.md` in the working directory and every parent directory
3. `~/.claude/projects/*/memory/*.md`
4. `README.md` in the working directory

Step 2 below reads three more things, and one of them is a file rather than a folder listing. All seven are disclosed in `SECURITY.md`, and "four locations, and no others" was wrong until 20 August 2026.

These are paths Claude Code already treats as user context. **The profile goes to the model API**, because step 3 passes it to four subagents verbatim, so treat everything read here as leaving the machine. Web searches carry topic keywords only, never a phrase lifted from a private note.

**Name every file you read at the top of the output.** The person did not get asked first, so they get told immediately after as well as before. This is not optional and it is not a footnote.

**A run costs real tokens.** The one measured run, on 19 August 2026, came to roughly 678,000 subagent tokens across five agents and 182 tool calls, with about sixteen minutes for the slowest agent. That was five agents, before the cap came down to four. One run, one profile, on one machine. It is a measurement, not a price.

## How a run goes

### 1. Read the profile sources

**Location 3 needs care.** `~/.claude/projects/*/memory/*.md` looks like one folder and is usually many. Claude Code creates a project folder per working directory, named after that directory with every non-alphanumeric character replaced by a hyphen, and it never deletes the old one when a folder moves or a temporary directory is used once. Tested on a real machine on 19 August 2026: twelve matching folders, ten of them empty scratch directories, one holding notes four days stale from a path the person had abandoned, and one live.

So do not glob blindly:

- Derive the expected folder from the **git repository root** if the working directory is inside one, and from the working directory itself if it is not. This was wrong until 20 August 2026: the instruction said working directory alone, which silently picks the wrong folder, or none, for anyone working in a subdirectory of a repository. A Windows drive letter survives the hyphen rule intact, so `G:\My Drive\ClaudeHub` becomes `G--My-Drive-ClaudeHub`.
- If that folder does not exist, use the memory folder with the most recently modified file.
- Skip empty folders entirely.
- **Say which folder you used and how old its newest file is.** A profile built from stale notes is worse than no profile, because it looks right.

Then read all four locations. Extract:

- What they do for work or study, and what they are moving towards
- Recurring tasks that show up more than once
- Tools, platforms and operating system
- Anything they have said they dislike or have already rejected
- **How they want to be worked with.** Temperament, working style, standing instructions about tone, pace and format. Whether they said something is their bottleneck.

That last one is the easiest to skip and the most valuable. On 19 August 2026 an agent's single best find was a focus and next-action skill, justified by a line about focus being the person's bottleneck. That line was in their configuration and absent from the profile summary the agent had been given, so it nearly went unrecommended. Task lists and tool lists find task and tool skills. Temperament is what every behavioural skill actually targets, and nothing else surfaces it.

**Pass the profile with its quotes attached, not as a summary.** Give each agent the verbatim lines that support each conclusion, and say which file each came from. A summary strips exactly the evidence row 3 requires.

**If none of the four exist**, say so plainly and ask them to describe what they do in one sentence, then carry on from that. Do not guess from an empty profile. Do not fall back to recommending popular skills. A popularity list is not what this is for, and anyone can find one.

**If only some exist, check whose they are before using them.** The dangerous case is partial, not empty, and it was unhandled until 20 August 2026. Run this in a work repository with no personal `CLAUDE.md` on the machine and locations 2 and 4 still hit: a project `CLAUDE.md` and a project `README.md`, both describing an employer's codebase and neither describing the person. The profile then reads as somebody who does whatever that repository does, the run is confidently wrong, and it has quietly profiled a third party's project rather than the user.

So before scoring anything, check that at least one of locations 1 and 3, the two that are actually about the person, produced something. If neither did, say exactly that: the only files found describe the project, not you. Then ask for the one-sentence description and treat the repository files as context about the work, never as evidence for row 3.

If the user passed an argument, narrow to that domain and say you have.

### 2. List what is already installed

Three places, not one. Missing any of them means recommending something the person already has, which is the fastest way to lose their trust in the whole list.

1. `~/.claude/skills/`, following links. A skill folder is often a symlink or junction pointing elsewhere, and the name in this folder is the name that counts.
2. `~/.claude/plugins/`, including `installed_plugins.json`. A plugin brings skills with it and they will not appear in `~/.claude/skills/`.
3. **Skills already available in the current session.** Some are provided by the host rather than installed, so they appear in neither folder above and will vanish in an ordinary terminal session.

That third category is a trap, found on 19 August 2026 when an agent nearly recommended a skill that was already live in the session but absent from both folders. Treat a session-provided skill as **not installed**, because it is not, but say so in the recommendation: "already available in this session, but not installed in Claude Code."

**Include plugin-namespaced skills.** A plugin's skills appear as `plugin-name:skill-name`. On the same day, two agents independently wasted most of a run on candidates the person already had, because the list they were handed contained standalone skills only and omitted every plugin-supplied one. An entire category of framework skills was already installed and neither agent could see it.

**Pass the complete list to all four agents, verbatim, in the `<INSTALLED>` slot in each brief.** Do not summarise it and do not trim it. An agent cannot check against what it was not given, and until 20 August 2026 the briefs had no such slot, so this instruction had nowhere to land and the dedupe rule was dead in every run.

### 3. Search, four agents, one per source

Announce the fan-out before dispatching it. Subagents cost the user real tokens and they should see it coming.

Dispatch four agents using the briefs in `references/agent-prompts.md`:

| Agent | Source |
|---|---|
| 1 | `github.com/anthropics/skills`, the official set |
| 2 | Claude Code plugin marketplaces |
| 3 | Community "awesome-claude-skills" style lists |
| 4 | Individual GitHub repos containing a `SKILL.md` |

Each agent searches its source, screens on descriptions down to a shortlist, reads every shortlisted candidate in full, scores it, and returns finished blocks only. Raw `SKILL.md` files never come back to this context. That is the entire reason for fanning out: twenty candidates at five to ten kilobytes each is a great deal of markdown to carry for skills that mostly get rejected.

**Four is a cap, not a starting point.** Never one agent per candidate. That scales with whatever the search happens to turn up and spawns thirty agents on somebody who asked a simple question.

If the `Agent` tool is unavailable, work the four sources one after another in this context and say that is what you are doing.

### 4. Score

Against `references/rubric.md`. Five rows, 1 to 5 each, out of 25. **Auto-reject if row 1, 2 or 3 scores under 3, or if row 5 scores 1**, whatever the total says. Row 5 was added to that list on 20 August 2026, because before then the safety row could not reject anything.

Then read the verdict off the total, per the bands in the rubric: INSTALL at 20 or above, TRIAL from 16 to 19, REJECT at 15 or below or on any auto-reject. A total of 20 or more that falls under 20 once row 3 is discounted to 3 is capped at TRIAL.

**Every TRIAL carries an exit line.** What would settle it, and when to check. Without one it is an INSTALL with a hedge in front of it.

**The default answer is "do not install".** Context is a real cost and most skills are not worth it.

### 5. Report

One block per recommendation, in the shape below. No cap on how many, because the default-REJECT stance and the three auto-reject rows keep the list short without an arbitrary limit.

### 6. Offer to go deeper

At the very end, not before: offer to read a notes folder if they point at one.

## Output

Open with the files read and the profile in three or four lines, so a wrong read is visible immediately.

Then one block each:

```
### owner/repo - skill-name - 21/25
What it does.  One line.
Horizon.       "Now" if it serves work they do today, "Ahead" if it serves the
               direction they named. Both are real recommendations.
Runs on.       Which operating systems. Say plainly if it will not run on theirs.
Why you.       Tied to something specific found in their files. Quote it.
               On a REJECT this line becomes "Why not", and names the disqualifier.
Trust.         Author, last commit, licence, official or marketplace listed.
               "No provenance available" is a valid answer, and a damning one.
Watch out.     Drawbacks, scripts it runs, keys it needs.
Verdict.       INSTALL, TRIAL or REJECT, then the clone URL.
Trial exit.    On a TRIAL only. "Use it on the next three CV rewrites. Keep it if the
               output needed no restructuring, drop it if it never fired."
```

"Why you" is the line that justifies this skill existing. If it could be written about anyone, the recommendation is not personalised and the score on row 3 is wrong.

**On a rejection, do not write a "Why you" line.** Arguing the fit for something you are about to reject buries the disqualifier in the one place a reader does not look for it. Write "Why not" and lead with the reason.

### Platform is an auto-reject

**If a skill does not run on the person's operating system, reject it whatever it scores.**

This is not covered by any of the five rows and it caught the rubric out in testing on 19 August 2026: a candidate whose own file said "Works on macOS and Linux. Windows support is planned" was being scored on fit and merit for a Windows user, and only failed by accident. A great skill that cannot run on their machine is worth nothing to them.

Note the operating system in the profile at step 1, and check every candidate against it.

**Which candidates get a full block, and which get one line:**

- Anything that hit an auto-reject gets **one line**. Its scores are not interesting; it failed a gate.
- Anything that cleared every gate gets a **full scored block**, whatever the verdict. A skill that scored 18 and still earns a REJECT is the most useful entry in the list, because the reasoning is not obvious and the person would otherwise install it.

A rejection with a reason teaches the person something about their own setup. A silent rejection teaches them nothing.

## What this will not do

**It does not install anything.** It hands over a clone URL and a verdict, and stops. Installing on someone's behalf, from a list generated by reading their private files, without asking, is too much for one command.

**It does not vet connectors.** A connector, also called an MCP server, is an external service Claude talks to. It holds credentials and moves data through a third party. Vetting one means reading a privacy policy and an OAuth scope list, not a markdown file. A score out of 25 that treats "this text file is safe" and "this company can read your Gmail" as the same measurement is a bad score.

One unscored mention at the end, clearly labelled as not vetted, is allowed. Nothing more.

**It does not score anything it has not opened.** Candidates are screened on their descriptions down to a shortlist, then every shortlisted one has its actual `SKILL.md` read in full before scoring. The output says which were screened out unopened and which were read, because a list that hides that boundary is claiming coverage it does not have. Blog posts, videos and X threads are leads to a repo, never sources in themselves.

## Limits, stated honestly

A profile built from four files is a thin read of a person. It will miss anything they have not written down.

The search only sees public GitHub and the marketplaces. Anything private, internal, or newly published is invisible to it.

Row 3, fit, is the most subjective row in the rubric and the one most likely to be generous. Treat a high total that rests on row 3 with suspicion.
