# Changelog

Every version, what changed, and why. Where something was wrong, the defect is written up
under the version that fixed it, including the ones that reflect badly on the project.

**What the numbers mean.** `MAJOR.MINOR.PATCH`. Major changes what the skill is for. Minor
changes what a run actually reads, produces or scores, the kind of change where the same
input gives a different answer. Patch corrects a defect or a claim without changing the
outcome of a run.

---

## v0.2.0

The version where somebody ran it end to end and wrote down what happened. Everything below
either changes what a run produces or removes a claim that was not true.

**Added**

- **A boundary on what reaches a subagent.** Files whose name indicates a secret are no
  longer opened at all, matched as whole words rather than substrings. Beyond that the skill
  classifies what it reads and withholds six categories from the profile it hands out:
  credentials, account and reference numbers, government identifiers, named third parties
  and anything said about them, health and legal and financial matters, and addresses and
  phone numbers.

  *Why it costs nothing.* None of those categories can score. Row 3 asks what somebody does
  repeatedly, and a policy number is not a recurring task, a friend's name is not a tool they
  use, and a tenancy dispute is not a direction they are moving in. The rubric never needed
  any of it, so this is not a trade against recommendation quality.

  *Why whole words.* Matched as a substring, `pin` hits "grouping", "mapping" and "shipping".
  A rule that silently drops ordinary notes is the same defect class as a pattern that
  silently drops most of them, so every skip is now named in the output and the person
  decides whether it was a credential or their filing.

  *What it is not.* An instruction to a model, not a control. It can fail in either
  direction, and it is stated that way in all three of `SKILL.md`, `README.md` and
  `SECURITY.md`.

- **A coverage footer in the output.** Candidates enumerated, screened unopened, opened and
  read, and the percentage. Also whether `gh` was authenticated, because a thin list from a
  starved search looks exactly like a thin market and only the run can tell the difference.

  The instruction to report this appeared five times in the prose and had no line in the
  output shape to land in, so it had never once appeared in a run. That is the third time
  this project has shipped an instruction with nowhere to write into, after the installed
  list and the screened-versus-opened boundary. **The pattern is the finding.** Check the
  templates against the prose rather than assuming the prose is enough.

- **Scoring ladders in all four agent briefs.** Each brief carried the five row questions,
  the gates and the bands, and none of the 1-to-5 descriptors, and cited the rubric zero
  times. Four agents were scoring out of twenty-five with no scale. Two honest scorers
  reading the same file land a whole verdict apart when that happens.

- **A read-only subagent type in the dispatch**, where the host provides one, and a line
  saying which was used or that none was available.

- **Cross-agent territory in each brief.** The four sources overlap: the official repository
  appears on nearly every community list and is reachable as a loose repository. Without an
  instruction, several agents score the same candidate independently and produce verdicts a
  band apart, and the parent cannot adjudicate because raw files never reach it by design.

- **Limits on the offer to read a notes folder.** It was the only place the skill asks
  permission and the only read with no depth, no file count, no size and no second
  disclosure. A folder somebody points at can be a whole vault. The offer now states its
  own terms before the answer.

**Fixed**

- **The memory read was one level deep and missed almost everything.** The pattern matched
  files sitting directly in the memory folder and nothing below it, so on a vault with
  folders in it the skill read a handful of notes and scored fit against a fraction of the
  person.

  *Why it was invisible.* Whatever the top level happens to hold is usually enough to satisfy
  the partial-profile check, so the run printed the same disclosure, dispatched the same four
  agents and produced an identically healthy-looking report. There was no failure to see. It
  now reads every `.md` beneath the memory folder at any depth, with no file cap, because a
  cap silently drops evidence and that is the defect being cured.

- **The vault is now read in batches.** The whole tree does not fit in a single read on a
  real machine, and the parent context assembling the profile is the binding constraint
  rather than the four subagents, which each carry a small profile comfortably. Quotes
  survive verbatim with their file attribution; what is dropped is the prose around them,
  which was never evidence.

- **The runtime disclosure undercounted.** It named four profile locations while the run read
  seven. Every other occurrence of that undercount corrected itself a line or two later; this
  one was an instruction rather than a description, and it governed the only count anyone
  ever sees before the first read.

- **`SECURITY.md` said the subagents hold WebFetch.** They are general-purpose. Dispatched
  without a named type, an agent inherits every tool available to subagents: Bash, Write and
  Edit alongside WebFetch, plus every connected MCP server. The old sentence understated the
  blast radius by roughly everything, and the threat model above it then reasoned correctly
  from a wrong premise. Stated now as capability rather than as an incident, because nothing
  has been exploited.

- **"Nothing is written to disk" was false**, and the same file contradicted it thirty-four
  lines later. The skill writes nothing itself, having neither Write nor Bash. Claude Code
  writes every session to a plain-text transcript in the same tree the memory notes are read
  from, so everything the profile step read is on disk twice.

- **The two-stage design was defended by a constraint that is wrong twice over.** The claim
  was that one marketplace carried over two thousand plugins and that GitHub's
  sixty-requests-an-hour limit made reading every candidate impossible.

  The figure appeared five times across the rubric and the briefs and was never cited at any
  occurrence; the largest marketplace anyone has actually counted holds a few hundred. And
  the limit does not govern reading: fetching a shortlisted `SKILL.md` from the raw content
  host is a CDN request and leaves the API budget untouched. The screen-then-read split
  survives because the real constraint was always tokens, and it is defended that way now.
  A design held up by a false constraint invites someone to check the constraint and discard
  the design with it.

- **An instruction no granted tool could satisfy.** The run was ordered to report the age of
  the newest file in the memory folder it picked. None of the pre-granted tools returns a
  modification time, so the instruction could only be skipped or answered with a guess. It
  now asks for the age only where it can be obtained and requires saying so when it cannot.

- **`README.md` never mentioned the threat model at all.** Every disclosure lived in
  `SECURITY.md`, which is the file people open after installing, if ever. The README is what
  a stranger reads before deciding. It now says that four agents read stranger-written
  prompts while holding quotes from private notes, that those agents are general-purpose,
  that the session transcript keeps everything, and that a safety score describes one
  markdown file rather than the repository a clone actually brings.

- **Row 5's lowest score auto-rejected every skill ever written.** It read "contains text
  addressed to the model reading it", and a `SKILL.md` is exactly that. It now names
  instructions that try to steer the reading agent, and says explicitly that a prompt
  template a skill offers its user is not an attack.

- **Row 4 scored the presence of a `references/` folder rather than whether loading is
  conditional.** If every run pulls those files anyway, the split is filing rather than
  progressive loading. Assay fails the corrected version and scores 2 on its own row 4.

- **Row 3 invited a citation the scorer cannot make.** The briefs allowed the quote to come
  from "any of their configuration files you have read", and a subagent has read none. That
  is an invitation to fabricate. The quote must come from the profile text supplied.

**Changed**

- **The files the model executes no longer carry the history of their own corrections.**
  Sentences recording what a file used to say, and when, were sitting inside `SKILL.md`, the
  rubric and all four briefs. For a reader of this changelog that is the record. For somebody
  who has just cloned the skill it is noise paid for on every run, and worse, it is
  confusing: a line about what was true before a given date reads as evidence you have an old
  copy. The reasoning has been kept everywhere it was load-bearing. What changed, and when,
  is what this file is for.

- **One machine's measurements are no longer written in as general facts.** A particular
  vault's file count, byte size and coverage percentage had been recorded as though they were
  the expected shape, which anchors a scorer to a stranger's setup. They are stated as the
  general rule they illustrate. The one measurement that keeps its date is the token cost of
  a run, because a measurement without a date is not evidence.

**Scope, settled**

Assay is a find-me-skills tool, with safety vetting as a feature of it. Narrowing it to
vetting alone, on the grounds that Claude Code now ships its own skill search, was considered
and rejected.

**Still open**

The rubric's band edges sit inside its own measured scoring drift, and the row 3 discount
clause inverts at totals of 20 and 21, so a lower fit score can produce a better verdict.
Drift has not been re-measured since the ladders were added; that they reduce it is a
prediction, not a result. Windows is untested. And nothing establishes that these
recommendations beat the first-party skill search Claude Code now ships, which is the
comparison that decides whether the discovery half is worth keeping.

---

## v0.1.1

Correctness and disclosure only. No change to what the skill does or how it scores, beyond
one gate that had never been able to fire.

**Fixed**

- **`README.md`, `SKILL.md` and `SECURITY.md` all said nothing is uploaded.** The agent
  briefs order the profile passed to each agent verbatim, so it was false in all three, and
  worst in the security file. All three now say plainly that profile text goes to the model
  API and nowhere else.

- **The safety row could not reject anything.** Only rows 1 to 3 gated, so a candidate
  scoring 5,5,5,5,1 totalled 21 and survived. The briefs order a score of 1 on spotting an
  embedded instruction, and that score changed no outcome. Row 5 scoring 1 is now an
  auto-reject, and an embedded instruction is an auto-reject on its own. The paid-key case
  moved from 1 to 2, so the new gate rejects hostility rather than price.

- **"Four locations, and no others" was wrong.** It is seven. The installed step also opens
  the contents of `installed_plugins.json`, which the security file denied in the words "it
  reads the names, not the files", and reads the skills the session provides, which was
  disclosed nowhere.

- **TRIAL had no definition.** It appeared in the output shape and nowhere in the rubric: no
  bands, no exit criterion. Scoring was non-monotonic as a result, with a 22/25 earning TRIAL
  while three separate 21/25 entries earned INSTALL. Bands are now INSTALL at 20 or above,
  TRIAL 16 to 19, REJECT at 15 or below or on any auto-reject, with a 20-or-above capped at
  TRIAL when discounting row 3 to 3 takes it under 20. Every TRIAL states what would settle
  it and when to check.

- **The memory folder was derived from the working directory.** Claude Code keys it on the
  git repository root where there is one, so the old rule silently picked the wrong folder,
  or none, for anyone working in a subdirectory of a repository.

- **Partial profile existence was unhandled.** Only the all-missing case was covered, so
  running inside somebody else's repository with no personal `CLAUDE.md` still hit the
  working-directory locations and quietly profiled that project as though it were the user.
  The skill now checks that at least one of the two person-shaped locations produced
  something before scoring fit.

- **The briefs had no `<INSTALLED>` slot.** `SKILL.md` ordered the installed list passed to
  all four agents verbatim and there was nowhere for it to go, so the dedupe rule was dead in
  every run.

- **Each brief claimed sixty API requests an hour.** That budget is per IP address and shared
  by all four agents, so roughly fifteen each.

**Changed**

- The measured cost of a run moved into `README.md` and `SKILL.md`, where somebody sees it
  before starting rather than after.
- `version:` removed from the frontmatter. It is not a documented field, Claude Code ignores
  it, and claude.ai Skills packaging rejects it.
- The disclosure prints before the first read rather than after it. The description fires on
  "what should I install", so most people meet this skill without choosing it, and a
  disclosure that arrives after the reads is a receipt rather than a warning.
- The name collision with an existing product scoring packages on agent-friendliness and
  security is stated in the README, with no claim of affiliation.

---

## v0.1.0

First release. Experimental.

Grown out of a private skill that searched on a topic you gave it. This one works out the
topic from you, which is the whole reason it exists as a separate thing.

**Added**

- A profile built from local locations, read without asking first and named in the output
- An installed list, so nothing gets recommended twice
- A fan-out to four agents, one per source: official repository, plugin marketplaces,
  community lists, loose GitHub repositories
- Five scored rows out of 25, with auto-rejects
- One block per recommendation with the drawbacks named, and one line for every rejection

**Decisions worth recording**

- **Maintenance is not a scored row.** The earlier private version auto-rejected anything
  without a commit in six months. A skill is a text file, not a library, and that rule threw
  away good work for no reason. Last commit date is reported as a fact and weighed by the
  reader.
- **Plugins are in scope, connectors are not.** A plugin is the same kind of object as a
  skill and the same checks apply. A connector holds credentials and moves data through a
  third party, and vetting one means reading a privacy policy rather than a markdown file.
  Scoring both on one scale would make the scale meaningless.
- **No confirmation step between reading your files and searching.** A deliberate choice of
  speed over a gate, and the reason the disclosure sits at the top of the README rather than
  buried in it.
- **The skill installs nothing.** Installing on somebody's behalf, from a list generated by
  reading their private files, without asking, is too much for one command.

**Known weak points on release**

Row 3, fit, is the most subjective row and had been tested against one profile. Two agents
exhausted GitHub's unauthenticated API budget mid-run and reached perhaps half the candidate
pool they otherwise would have, and the run did not say by how much.
