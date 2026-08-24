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

**Say this before the first read, not after it.** This skill fires on phrases like "what should I install", so most people meet it without having chosen it, and a disclosure printed after the reads have happened is a receipt rather than a warning. Open with one line naming **all seven locations**, the four that build the profile and the three that list what is already installed, and saying they are about to be read. Then read them. It costs a sentence.

**Count them out loud and get the number right.** That line is the only count anyone ever sees, so an undercount there is the whole disclosure being wrong, whatever the rest of the documentation says.

Four locations build the profile:

1. `~/.claude/CLAUDE.md`
2. `CLAUDE.md` in the working directory and every parent directory
3. `~/.claude/projects/<key>/memory/`, every `.md` beneath it at any depth
4. `README.md` in the working directory

Step 2 below reads three more things, and one of them is a file rather than a folder listing. That is seven, not four, and all seven are disclosed in `SECURITY.md`.

These are paths Claude Code already treats as user context. **The profile goes to the model API**, because step 3 passes it to four subagents verbatim, so treat everything read here as leaving the machine. Web searches carry topic keywords only, never a phrase lifted from a private note.

**Name every file you read at the top of the output.** The person did not get asked first, so they get told immediately after as well as before. This is not optional and it is not a footnote.

**A run costs real tokens.** The one measured run, on 19 August 2026, came to roughly 678,000 subagent tokens across five agents and 182 tool calls, with about sixteen minutes for the slowest agent. That was five agents, before the cap came down to four. One run, one profile, on one machine. It is a measurement, not a price.

## How a run goes

### 1. Read the profile sources

**Location 3 needs care.** `~/.claude/projects/*/memory/*.md` looks like one folder and is usually many. Claude Code creates a project folder per working directory, named after that directory with every non-alphanumeric character replaced by a hyphen, and it never deletes the old one when a folder moves or a temporary directory is used once. A real machine can easily carry a dozen of them: mostly empty scratch directories, one holding stale notes from a path the person abandoned, and one live.

So do not glob blindly:

- Derive the expected folder from the **git repository root** if the working directory is inside one, and from the working directory itself if it is not. Deriving it from the working directory alone silently picks the wrong folder, or none, for anyone working in a subdirectory of a repository. A Windows drive letter survives the hyphen rule intact, so `G:\My Drive\ClaudeHub` becomes `G--My-Drive-ClaudeHub`.
- If that folder does not exist, use the memory folder with the most recently modified file.
- Skip empty folders entirely.
- **Read every `.md` beneath `memory/`, at any depth, not only the ones sitting directly in it.** A one-level pattern finds almost nothing on a vault with folders in it, and people organise their notes. The danger is not the miss, it is that the miss is invisible: whatever the top level happens to hold is usually enough to pass the partial-profile check below, so the run reports a healthy profile and scores fit against a fraction of the person. A silent wrong answer is worse than a loud failure.
- **Say which folder you used.** A profile built from stale notes is worse than no profile, because it looks right. Report the age of its newest file **only if you can actually obtain it**. The tools this skill pre-grants do not return modification times, so unless you can reach a shell, say the age is unavailable rather than estimating one. An invented timestamp is worse than an absent one.

### 1a. What never reaches a subagent

Reading deeply means reading things the fit score has no use for. **A memory vault is where people keep the details of their life, and this skill hands what it reads to four model calls.** So there is a boundary, and it is drawn by category rather than by filename, because a filename list only protects the person who wrote the list.

**Skip on the name, before opening anything.** If a filename indicates a secret, do not open the file at all. Match **whole words**, not substrings: split the filename on spaces, hyphens, underscores and dots, then look for `pin`, `pins`, `password`, `passwords`, `passwd`, `secret`, `secrets`, `credential`, `credentials`, `token`, `tokens`, `apikey`, `seed`, `recovery`, `licence key`, `license key`, or a file named `.env`. Nothing read is nothing leaked, and this is the only part of the boundary that costs nothing.

**Whole words, because substring matching is wrong and quietly so.** Matched as a substring, `pin` hits "grouping", "mapping" and "shipping", so an ordinary note gets skipped and nobody is told. That is the same defect class as a glob that silently drops most of them.

**Name every file you skipped this way, in the output.** A skip is invisible otherwise, and the person is the only one who can tell a protected credential from a false positive on their own filing.

**Then classify what you did open, and withhold these categories from everything you pass on.** Not from your own reading, from the profile you hand to the four agents:

- Credentials, keys, tokens, passwords and PINs in any form
- Account, policy, customer, card and reference numbers
- Government identifiers: passport, national insurance, driving licence, tax
- **Named third parties and anything said about them.** Other people's names, what they advised, what was agreed with them, what they are going through. Somebody who never installed this skill cannot consent to being in its profile
- Health, legal and financial matters, whether the person's own or anyone else's
- Full postal addresses, phone numbers, private email addresses

**The argument for withholding these is that none of them can score anything.** Row 3 asks what this person does repeatedly. A policy number is not a recurring task, a friend's name is not a tool they use, and a tenancy dispute is not a direction they are moving in. The rubric never needed any of it, so leaving it out costs the recommendation nothing and is not a trade.

Where a file mixes useful evidence with a withheld category, take the evidence and leave the rest: "recurring task, chasing a housing matter across months" carries the pattern without the address, the landlord or the sum. **Say what you withheld by category and file, never by content.** One line: "two files carried third-party detail, summarised rather than quoted."

**Be honest about what this is.** It is an instruction to a model, not a control the harness enforces, and it can be got wrong in both directions. It sits here because the alternative was reading a whole vault and passing it on unexamined, which is worse. Anyone whose notes hold something they would not paste into a chat window should check them before running this, and `SECURITY.md` says so at more length.

Then read all four locations. Extract:

- What they do for work or study, and what they are moving towards
- Recurring tasks that show up more than once
- Tools, platforms and operating system
- Anything they have said they dislike or have already rejected
- **How they want to be worked with.** Temperament, working style, standing instructions about tone, pace and format. Whether they said something is their bottleneck.

That last one is the easiest to skip and the most valuable. The single best find in a run is often a behavioural skill justified by one line about how somebody works, a line that sits in their configuration and never makes it into a profile summary built from task lists. Task lists and tool lists find task and tool skills. Temperament is what every behavioural skill actually targets, and nothing else surfaces it.

**Read the vault in batches, and extract as you go.** There is deliberately **no cap on how many files are read**; a cap silently drops evidence, which is the defect reading deeply exists to cure. But the whole tree does not have to be resident at once, and on a real vault it will not fit in a single read.

So take the tree in batches, pull the quotable lines out of each batch, and carry forward the extract rather than the file. The profile that reaches the agents is a few kilobytes of evidence drawn from a few hundred kilobytes of notes. **This is not the summarising the next paragraph forbids.** The quotes survive verbatim with their file attribution, which is the whole of what row 3 needs. What gets dropped is the prose around them, which was never evidence.

**The parent is the binding constraint, not the subagents.** Four agents each carry a small profile comfortably. The context assembling it is where a large vault actually breaks.

**Pass the profile with its quotes attached, not as a summary.** Give each agent the verbatim lines that support each conclusion, and say which file each came from. A summary strips exactly the evidence row 3 requires.

**If none of the four exist**, say so plainly and ask them to describe what they do in one sentence, then carry on from that. Do not guess from an empty profile. Do not fall back to recommending popular skills. A popularity list is not what this is for, and anyone can find one.

**If only some exist, check whose they are before using them.** The dangerous case is partial, not empty. Run this in a work repository with no personal `CLAUDE.md` on the machine and locations 2 and 4 still hit: a project `CLAUDE.md` and a project `README.md`, both describing an employer's codebase and neither describing the person. The profile then reads as somebody who does whatever that repository does, the run is confidently wrong, and it has quietly profiled a third party's project rather than the user.

So before scoring anything, check that at least one of locations 1 and 3, the two that are actually about the person, produced something. If neither did, say exactly that: the only files found describe the project, not you. Then ask for the one-sentence description and treat the repository files as context about the work, never as evidence for row 3.

If the user passed an argument, narrow to that domain and say you have.

### 2. List what is already installed

Three places, not one. Missing any of them means recommending something the person already has, which is the fastest way to lose their trust in the whole list.

1. `~/.claude/skills/`, following links. A skill folder is often a symlink or junction pointing elsewhere, and the name in this folder is the name that counts.
2. `~/.claude/plugins/`, including `installed_plugins.json`. A plugin brings skills with it and they will not appear in `~/.claude/skills/`.
3. **Skills already available in the current session.** Some are provided by the host rather than installed, so they appear in neither folder above and will vanish in an ordinary terminal session.

That third category is a trap: a skill can be live in the session and absent from both folders, and gets recommended back to somebody already using it. Treat a session-provided skill as **not installed**, because it is not, but say so in the recommendation: "already available in this session, but not installed in Claude Code."

**Include plugin-namespaced skills.** A plugin's skills appear as `plugin-name:skill-name`. On the same day, two agents independently wasted most of a run on candidates the person already had, because the list they were handed contained standalone skills only and omitted every plugin-supplied one. An entire category of framework skills was already installed and neither agent could see it.

**Pass the complete list to all four agents, verbatim, in the `<INSTALLED>` slot in each brief.** Do not summarise it and do not trim it. An agent cannot check against what it was not given. If the briefs carry no such slot, this instruction has nowhere to land and the dedupe rule is dead in every run.

### 3. Search, four agents, one per source

Announce the fan-out before dispatching it. Subagents cost the user real tokens and they should see it coming.

Dispatch four agents using the briefs in `references/agent-prompts.md`:

| Agent | Source |
|---|---|
| 1 | `github.com/anthropics/skills`, the official set |
| 2 | Claude Code plugin marketplaces |
| 3 | Community "awesome-claude-skills" style lists |
| 4 | Individual GitHub repos containing a `SKILL.md` |

**Name a read-only subagent type in the dispatch, and say which one you used.** Dispatching without naming a type gets a general-purpose agent, which inherits every tool available to subagents: Bash, Write and Edit alongside WebFetch, plus every connected MCP server. These agents need to search and read, nothing else, and they are carrying verbatim quotes from the person's private notes while reading files written by strangers. Prefer whatever read-only agent type the host provides, if it provides one. If it offers none, say so in the output rather than passing over it in silence, because the difference matters to anyone deciding whether to run this.

Each agent searches its source, screens on descriptions down to a shortlist, reads every shortlisted candidate in full, scores it, and returns finished blocks only. Raw `SKILL.md` files never come back to this context. That is the entire reason for fanning out: twenty candidates at five to ten kilobytes each is a great deal of markdown to carry for skills that mostly get rejected.

**The four sources overlap, so tell each agent whose territory is whose.** `anthropics/skills` is agent 1's source, and it also appears on nearly every community list agent 3 reads and is reachable as a loose repository by agent 4. Without an instruction, several agents score the same candidate independently, and the spread between them is a whole verdict.

So each brief names the other three agents' sources and says: if a candidate's home belongs to another agent, note it and move on rather than scoring it again. **The parent cannot repair this afterwards.** Raw candidate files never come back here, by design, so on discovering the same skill twice with two different verdicts there is no evidence left to adjudicate with. Printing both looks broken and silently dropping one is worse.

**Four is a cap, not a starting point.** Never one agent per candidate. That scales with whatever the search happens to turn up and spawns thirty agents on somebody who asked a simple question.

If the `Agent` tool is unavailable, work the four sources one after another in this context and say that is what you are doing.

### 4. Score

Against `references/rubric.md`. Five rows, 1 to 5 each, out of 25. **Auto-reject if row 1, 2 or 3 scores under 3, or if row 5 scores 1**, whatever the total says. Row 5 gates too, because otherwise the safety row cannot reject anything.

Then read the verdict off the total, per the bands in the rubric: INSTALL at 20 or above, TRIAL from 16 to 19, REJECT at 15 or below or on any auto-reject. A total of 20 or more that falls under 20 once row 3 is discounted to 3 is capped at TRIAL.

**Every TRIAL carries an exit line.** What would settle it, and when to check. Without one it is an INSTALL with a hedge in front of it.

**The default answer is "do not install".** Context is a real cost and most skills are not worth it.

### 5. Report

One block per recommendation, in the shape below. No cap on how many, because the default-REJECT stance and the three auto-reject rows keep the list short without an arbitrary limit.

### 6. Offer to go deeper

At the very end, not before: offer to read a notes folder if they point at one.

**Bound the offer before you make it, and bound it in the same sentence.** This is the only place the skill actually asks permission, and a folder somebody points at can be a whole vault.

So the offer names its own terms. Say that you will read to a depth of three, skip anything whose filename suggests a credential, stop at roughly forty files, and that whatever you read goes to the model API the same way the first profile did. Then honour those limits and report what you skipped. An offer whose cost the person cannot see is not much better than not asking.

## Output

Open with the files read and the profile in three or four lines, so a wrong read is visible immediately. **Name any file skipped on its filename**, so a false positive on somebody's own filing is visible rather than silent.

**Then a coverage footer, before the blocks, and it is not optional:**

```
Coverage.      NN candidates enumerated across the four sources. NN screened out on their
               description without being opened. NN opened and read in full. That is N.N%.
               gh authenticated: yes / no. If no, say the pool reached was smaller for it.
```

**A fraction of one per cent is the normal answer here, and that is exactly why it gets printed.** Hiding it is how a sample gets mistaken for a sweep. Report it whatever it is.

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

This is not covered by any of the five rows. Without it, a candidate whose own file says "Works on macOS and Linux, Windows support is planned" gets scored on fit and merit for a Windows user, and passes or fails by accident. A great skill that cannot run on their machine is worth nothing to them.

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
