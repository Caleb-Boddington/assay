# Security

This covers two things: what the skill does to your machine, and how to report a problem with it.

The first matters more. This skill reads your private files by design, so you are entitled to know exactly what it touches.

## Supported versions

Only `main`. This is a single-branch personal project and there is no backporting.

## Reporting

Open an issue. Say whether you have a working exploit or a design objection, because they need different responses and the difference is not always obvious from a description.

## What it reads

Four locations, and the skill is instructed to read nothing else without being pointed at it:

- `~/.claude/CLAUDE.md`
- `CLAUDE.md` in the working directory and every parent directory
- `~/.claude/projects/*/memory/*.md`
- `README.md` in the working directory

It also lists the contents of `~/.claude/skills/` and `~/.claude/plugins/` to avoid recommending things you already have. It reads the names, not the files.

At the end of a run it may offer to read a notes folder you name. That is an offer. It does not act on it unless you answer.

## What leaves your machine

Search queries, built from topic keywords. "cyber security", "email triage", "CV tailoring".

Not sent: file contents, file paths, quoted lines from your notes, your name, or anything else lifted verbatim from what it read. The skill is instructed to derive keywords rather than pass text through.

Nothing is written anywhere. No telemetry, no analytics, no logging to disk.

## Threat model

**Four agents read untrusted web content.** They fetch and read `SKILL.md` files written by strangers on GitHub. A `SKILL.md` is a prompt. A malicious one could contain text addressed to the agent reading it.

All four briefs in `references/agent-prompts.md` instruct the agent to treat everything it reads as data rather than instructions, and to report any embedded instruction verbatim while scoring that candidate's safety row at 1.

**The profile is built from your private notes.** If your `CLAUDE.md` or memory files contain credentials, those enter the model's context during a run. They should not be in those files in the first place, but people put them there.

**Recommendations point at third-party code.** The skill tells you a repository is worth installing. You then clone it. Everything after the clone is between you and that repository's author.

## Not defended

Being plain about the gaps, because a security file that lists only mitigations is marketing.

**The instructions have no enforcement.** Every rule here is a line in a markdown file asking a model to behave. There is no sandbox, no allowlist, and nothing that stops the model reading a fifth location if it decides to. A skill cannot enforce its own boundaries.

**No abort.** Once the four agents are dispatched there is no way to recall them mid-run.

**Prompt-injection resistance is untested.** The instruction to treat retrieved text as data is written into all four briefs and has not been tested against a deliberately hostile `SKILL.md`. Treat it as an intention, not a guarantee.

**The safety score is a reading, not an analysis.** Row 5 depends on an agent noticing a dangerous script by reading it. Obfuscated or subtle behaviour will be missed. A high safety score means nothing alarming was spotted, and that is all it means.

**Session transcripts hold everything.** Whatever the profile step read is now in your session history, unencrypted, wherever your client keeps it.

## What to do about it

Before running this, check that your `CLAUDE.md` and memory files contain no credentials, tokens or keys. That is good practice regardless, and this skill makes it matter more.

Read any recommendation's `SKILL.md` yourself before installing it. The score is a filter, not a substitute.
