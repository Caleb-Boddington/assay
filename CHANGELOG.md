# Changelog

## 0.1.1 - 20 August 2026

Correctness and disclosure only. No change to what the skill does or how it scores, beyond one gate that was never able to fire.

Everything here came out of a Quorum audit of `0.1.0` at commit `eeee555`, run 19 August 2026, Full tier, 33 agents, ranked by credibility cost rather than user harm. The audit reports and never edits, so this is the first commit that acts on it. Full report at `docs/2026-08-19-quorum-audit.md`.

**The false claim, in three files**

`README.md`, `SKILL.md` and `SECURITY.md` all said nothing is uploaded. `references/agent-prompts.md` orders the profile passed to each agent verbatim, so it was false in all three, and worst in the security file. All three now say plainly that profile text goes to the model API, and that it goes nowhere else. This was the audit's headline finding: a security document, written by someone entering security, contradicted by the skill's own instructions.

**The safety row could not reject anything**

Only rows 1 to 3 gated, so a candidate scoring 5,5,5,5,1 totalled 21 and survived. The briefs order a score of 1 on spotting an embedded instruction, and that score changed no outcome. Row 5 scoring 1 is now an auto-reject, and an embedded instruction is an auto-reject on its own whatever the other rows say. The paid-key case moved from row 5 = 1 to row 5 = 2, so the new gate rejects hostility rather than price.

Found only by joining two departments in the audit. All six missed it individually.

**"Four locations, and no others" was wrong**

It is seven. Step 2 also opens the contents of `installed_plugins.json`, which `SECURITY.md` denied in the words "It reads the names, not the files", and reads the skills the current session provides, which was disclosed nowhere at all.

**The measured cost is now where a user sees it**

Roughly 678,000 subagent tokens, 182 tool calls, sixteen minutes for the slowest agent. The figure was already published in `docs/2026-08-19-first-run.md` and missing from both files anyone reads first. It is stated dated, and marked as one run on one profile across five agents rather than the four the skill now caps at. A measurement, never a price.

**TRIAL is defined**

It appeared in the output shape and nowhere in the rubric: no bands, no exit criterion. The first published run was non-monotonic because of it, a 22/25 earning TRIAL while three separate 21/25 entries earned INSTALL. Bands are now INSTALL at 20 or above, TRIAL 16 to 19, REJECT at 15 or below or on any auto-reject, with a 20-or-above capped at TRIAL when discounting row 3 to 3 takes it under 20. Every TRIAL states what would settle it and when to check.

**The name collision is named here rather than found later**

`assay.tools` scores packages on agent-friendliness and security. Same verb, same object, adjacent market. Stated in the README, with no claim of affiliation.

**Two correctness bugs**

The memory folder was derived from the working directory. Claude Code keys it on the git repository root when there is one, so the old rule silently picked the wrong folder, or none, for anyone working in a subdirectory of a repository.

Partial profile existence was unhandled. Only the all-missing case was covered, so running inside somebody else's repository with no personal `CLAUDE.md` on the machine still hit locations 2 and 4, and quietly profiled the employer's project as though it were the user. The skill now checks that at least one of the two person-shaped locations produced something before scoring fit.

**Smaller, same cause**

The briefs gained an `<INSTALLED>` slot. `SKILL.md` ordered the installed list passed to all four agents verbatim and there was nowhere for it to go, so the dedupe rule was dead in every run to date.

Each brief now says the sixty-request GitHub limit is per IP address and shared by all four agents, roughly fifteen each, rather than telling each agent it has sixty.

`version:` is gone from the frontmatter. It is not a documented field, Claude Code ignores it, and claude.ai Skills packaging rejects it.

The disclosure is printed before the first read rather than after it. The description fires on "what should I install", so most people meet this skill without choosing it, and a disclosure that arrives after the reads is a receipt rather than a warning.

**Not addressed, and worth stating**

The audit named four questions the run never answered: whether two locations rather than four would build an adequate profile, whether the disclosure survives auto-invocation in practice rather than in principle, whether the skill is portable to a second machine, and how large the rubric's scoring drift actually is. Drift of at least 2 is established. Its size is not.

The injection defence still rests on one fixture the author wrote, refused once. `SECURITY.md` now says so in those words.

## 0.1.0 - 19 August 2026

First release. Experimental.

Grown out of a private skill that searched on a topic you gave it. This one works out the topic from you, which is the whole reason it exists as a separate thing.

**What it does**

- Builds a profile from four local locations, without asking first, and names them in the output
- Lists installed skills and plugins so nothing gets recommended twice
- Fans out to four agents, one per source: official repository, plugin marketplaces, community lists, loose GitHub repositories
- Scores every candidate out of 25 against five rows, with three of them able to auto-reject
- Reports a block per recommendation with the drawbacks named, and one line for every rejection

**Decisions worth recording**

Maintenance was dropped as a scored row. The earlier private version auto-rejected anything without a commit in six months. A skill is a text file, not a library, and that rule was throwing away good work for no reason. Last commit date is now reported as a fact and weighed by the reader.

Plugins are in scope, connectors are not. A plugin is the same kind of object as a skill and the same checks apply. A connector holds credentials and moves data through a third party, and vetting one means reading a privacy policy rather than a markdown file. Scoring both on one scale would make the scale meaningless.

There is no confirmation step between reading your files and searching. That was a deliberate choice for speed over a gate, and it is the reason the disclosure sits at the top of the README rather than buried in it.

The skill installs nothing. Installing on someone's behalf, from a list generated by reading their private files, without asking, is too much for one command.

**Tested before release**

One full run, five agents, on 19 August 2026. It was run to find defects rather than to find skills, and it found eleven. The largest: the reading rule was literally impossible against a marketplace of two thousand plugins under a sixty-request hourly rate limit; frontmatter was never checked, so two of the best candidates in the whole run turned out to be undiscoverable by Claude Code after their bodies had been read in full; and the profile step never extracted temperament, which is what every behavioural skill targets.

The injection defence was tested against a hostile fixture and held. One attack, one shape, once.

**Known weak points on release**

Row 3, fit, is the most subjective row and has been tested against one profile.

Two agents exhausted GitHub's unauthenticated API budget mid-run and reached perhaps half the candidate pool they otherwise would have. A run without `gh` authenticated sees less than it should, and does not currently say by how much.
