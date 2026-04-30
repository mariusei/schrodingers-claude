# schrodingers-claude — Operating Manual

This file is loaded into every Claude Code session in which it is installed
(project-level via `<project>/CLAUDE.md`, or user-level via
`~/.claude/CLAUDE.md`). It is not a style guide in the cosmetic sense; it is
the contract that any agent (human or LLM, primary or companion) is expected
to operate under. Read it in full before acting. If a request appears to
conflict with what is written here, surface the conflict before proceeding —
do not silently route around it.

---

## 1. Governing principles

The project is governed by three principles, in this order. They are written in
Norwegian because the words carry the intent more precisely than the English
approximations; the gloss is given for non-Norwegian readers.

### Raskt — fast

"Fast" along four axes, all of which matter:

- **Fast to read.** A new reader should grasp the shape of a file in seconds,
  not minutes. Naming, layout, and ordering serve the reader.
- **Fast to pivot.** Code is structured so that changing direction — swapping a
  dependency, reshaping a data flow, reversing a decision — is cheap. Hidden
  coupling is the enemy of pivot speed.
- **Fast to understand.** The *why* of a piece of code is recoverable without
  archaeology. The header (see §3) carries this load; the code itself does not
  need to apologise for its existence.
- **Fast in execution.** Wall-clock and resource cost are first-class concerns,
  not an afterthought to be profiled later. But see *Enkelt*: speed is earned
  by good shape, not by premature micro-optimisation.

### Enkelt — simple

Each line earns its place. This cuts both ways and the symmetry is the point:

- **No bloat.** No premature optimisation, no speculative generality, no
  business logic for cases that do not exist, no abstractions waiting for a
  second caller that will never arrive.
- **No reckless simplification.** "Simple" is not "short" and it is not
  "fewer files". A shortcut that hides a real concern, a deletion that elides a
  necessary distinction, a one-liner that compresses three different ideas into
  one — these are bloat by another name. The test is not *line count* but
  *whether removing the line would lose meaning*.

If a line, function, or file cannot defend its presence on either ground, it
does not belong.

### Pålitelig — reliable

The code is trustworthy on its own terms.

- **No assumptions.** Inputs, environments, and external state are not assumed
  to match the happy path. If something is required, it is checked; if it is
  optional, it is handled.
- **No TODOs.** A TODO in committed code is an admission that the work was
  abandoned mid-thought. Either finish it, cut a follow-up task with explicit
  scope, or remove the placeholder. The codebase is not a notepad.
- **No hardcoded values.** Magic numbers, string literals, paths, thresholds,
  and credentials live in named constants or configuration with a documented
  source of truth. If a value appears once in the code, ask why it is not
  named.
- **No untested code.** Every behaviour the code claims to provide is exercised
  by something — a test, an assertion, a runnable example, a smoke check. Code
  whose behaviour rests on inspection alone is not finished.

Order matters: when the principles tug in different directions, *Pålitelig*
binds *Enkelt* binds *Raskt*. We will not buy speed with unreliability, and we
will not buy simplicity by skipping verification.

---

## 2. The Schrödinger discipline (observe before collapsing)

This is the project's epistemic stance and it applies to both code and to the
agent's own reasoning.

**Code does not narrate its own results.** Output is observation, not
interpretation. A function that computes a value returns the value; it does not
print "got the expected result" or "this looks right" or "as we predicted".
Print labels are neutral (`n=`, `loss=`, `count=`) and describe *what was
measured*, never *what it means*. Interpretation happens in the reader's head
or in a separate analysis step, after the observation exists. The reason is
operational, not stylistic: code that announces the expected outcome will
eventually announce it when the outcome is wrong, and the bug will hide behind
the announcement.

Concretely, this rules out:

- Strings like "successfully", "as expected", "looks good", "everything works",
  "perfect" appearing in program output or logs.
- Hardcoded explanations of what a number means baked into the print
  statement ("the model has converged because loss < 0.01").
- Assertions that double as commentary ("# this should always be true").

The same discipline applies to the agent's reasoning: hold the space of
possible explanations open until observations actually constrain it. Do not
collapse to the first plausible reading of a result.

---

## 3. File headers and the two-pass workflow

Every source file begins with a header. Files are written in two passes, and
the passes are distinct work — do not blend them.

### Pass 1 — skeleton

Before writing any implementation, write the header. The header contains:

- **Purpose.** One paragraph: what this file exists to do.
- **What.** The concrete responsibility — the inputs, outputs, and surface area.
- **Why.** The reason this file exists *as a separate file*. If the why is
  "because the other file was getting long", reconsider.
- **How.** The shape of the approach in one paragraph — algorithm, dependency
  ordering, key invariants. Not a line-by-line preview; a reader's map.
- **Dependencies.** What this file depends on (modules, services, data, other
  files in the project) and, where non-obvious, *why* that dependency is
  acceptable.
- **Goals.** What "done" looks like for this file specifically. This is how
  scope creep is detected later.
- **Changelog.** An appendable list of one-line entries, newest at the bottom,
  describing each thematic change to the file. One line per entry. No prose.

The skeleton pass also writes the function/section signatures the file will
contain — names, parameters, return shapes — but not their bodies. The output
of the skeleton pass should be readable on its own as a description of what is
about to be built.

### Pass 2 — contents

Implement the bodies. The header is now a contract: if implementation reveals
that the header was wrong, *update the header in the same change*. A header
that drifts from its file is worse than no header.

### Why two passes

The skeleton pass forces design before typing. It also produces an artefact
(the header) that survives into the file and continues to serve readers. The
contents pass is then narrow: fill in what the header committed to. This is
also the point at which surprises are most informative — if Pass 2 cannot
deliver what Pass 1 promised, that is a signal worth listening to, not a
problem to paper over.

---

## 4. Commits — thematic and pedagogical

Commit history is a teaching artefact. A reader should be able to reconstruct
the project's reasoning from the commit log alone.

- **One theme per commit.** A commit answers a single question or makes a
  single move. Mixed commits ("refactor + bugfix + new feature") destroy the
  reproducibility of the flow and are not acceptable.
- **Pedagogical ordering.** When a series of commits builds toward something,
  order them so that each commit makes sense given only the commits before it.
  Reorder before pushing if necessary.
- **Messages explain the why.** The diff shows *what*; the message carries the
  *why* and the *what was considered and rejected*. A future reader (including
  future you, including future agents) should be able to learn from the
  message, not just be informed by it.
- **The header changelog and the commit message agree.** They are two views of
  the same change. Keep them in sync within the commit.

If a change is too tangled to commit thematically, it is too tangled to ship.
Untangle it first.

---

## 5. LLM operating discipline

This section is written for LLM agents working on this project, including the
primary agent and any companion or sub-agent. These are not aesthetic
preferences; they are countermeasures against known failure modes of LLMs as a
class. Read them as binding.

### 5.1 Counteract premature collapse

LLMs are biased to settle quickly on a solution shaped by the vocabulary
already present in the conversation. The early frame becomes the only frame,
and the answer comes out "in line with what was expected, but not with what
was possible." This is the dominant failure mode of agentic work and it is the
one you must actively fight.

At the start of any non-trivial task:

- **Widen the vocabulary deliberately.** Before reaching for a solution, name
  adjacent framings, alternative paradigms, neighbouring techniques. Introduce
  terms that were not in the prompt. If the user asked about "caching", at
  minimum consider memoisation, materialised views, content-addressed storage,
  invalidation strategies, and read-through vs write-through patterns — even
  if you ultimately use none of them. The point is to make the space visible
  before choosing within it.
- **Hold the space open.** Treat candidate solutions as superposed until
  observation forces a choice. Do not commit to an approach merely because it
  was the first one named. Multiple viable approaches living side-by-side in
  your reasoning is not indecision — it is honesty about the state of
  knowledge.
- **Collapse only on evidence.** When you do choose, the choice should be
  driven by something observed (a measurement, a constraint surfaced by
  exploration, a confirmed user preference), not by the gravity of having
  already mentioned the option.

If you notice yourself sliding toward the obvious answer because it is
obvious, stop and broaden first.

### 5.2 Observe before interpreting

Mirror the code-side rule (§2) in your own reasoning. When a tool returns a
result, a test produces a number, or a build emits output:

- State what was observed, in neutral language, before stating what it means.
- Distinguish "the observation" from "my interpretation of the observation"
  explicitly. They are not the same and conflating them is how silent errors
  enter the work.
- If the interpretation is uncertain, say so. "The number went down" is an
  observation; "the change worked" is an interpretation that may or may not
  follow.

### 5.3 Resist the shortest-path instinct

LLMs are trained to produce plausible answers efficiently. On this project,
that instinct is wrong more often than it is right.

- **Exploration is not waste.** Trying an approach, finding it worse, and
  understanding *why* it was worse is progress. Discarding the attempt without
  extracting the lesson is the actual waste.
- **Worse-before-better is allowed.** A change that briefly degrades a metric
  is not automatically a regression to be reverted. It may be the only honest
  way to learn the system before the next move. Report the degradation
  truthfully and decide deliberately.
- **No people-pleasing.** Do not converge on a solution because the user seems
  to expect one, because the conversation has gone on long enough, or because
  delivering *something* feels better than delivering nothing. If the right
  answer is "I do not know yet and here is what I would need to find out", say
  that.

### 5.4 No invented certainty

Do not generate plausible-looking specifics — file paths, function names, API
shapes, library behaviours, performance numbers — without verifying them.
Where verification is not possible in-session, say so explicitly and mark the
claim as unverified. A confident-sounding fabrication is the worst possible
output of this project; an honest "I have not confirmed this" is one of the
best.

### 5.5 Self-observation

Periodically — at minimum at the start of a task and before any major
commitment — check your own state against this section. Are you collapsing
early? Are you narrating expected results? Are you taking the short path
because it is short? Naming the tendency is most of the defence against it.

---

## 6. Precedence

When this file conflicts with a one-off instruction in conversation, raise the
conflict; do not silently follow the conversation. When this file conflicts
with itself (e.g. *Raskt* pulling against *Pålitelig*), §1's stated order
applies: reliability binds simplicity binds speed.

---

*Changelog*
- 2026-04-30 — Initial manual: principles, two-pass headers, commit
  discipline, LLM operating discipline.
