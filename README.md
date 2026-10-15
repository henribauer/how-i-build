# How I build

I build software with two AI agents: one writes the code, the other reviews it, and I approve every
step that matters. This repo explains that workflow for coding and for design, and why each rule
exists. Most of the rules are here because something broke before they were in place.

I'm Henri, a business student in Berlin. I'm not a trained software engineer, so the workflow does
the job a senior colleague would: it asks the awkward questions up front, checks the work, and writes
down what we learned so the next session doesn't start from zero.

- [The core idea](#the-core-idea)
- [The coding loop](#the-coding-loop)
- [The design track](#the-design-track)
- [Checks and review without me](#checks-and-review-without-me)
- [Memory: the knowledge engine](#memory-the-knowledge-engine)
- [Storage rules that came from breakage](#storage-rules-that-came-from-breakage)
- [What doesn't work yet](#what-doesnt-work-yet)
- [Local models](#local-models)
- [Examples](#examples)

## The core idea

```mermaid
flowchart TD
    H([Henri]) -->|describes outcome| D[Diverge: my ideas first, then the agent widens them<br/>new features only]
    D --> RS[Research: principles, existing building blocks, tools<br/>new features only]
    RS --> G[Grill: questions on intent, risk, trade-offs]
    G --> S[Spec + tickets]
    S -->|Henri OKs| B[Builder agent<br/>Claude Code + cheaper subagents]
    B -->|every turn end| CG{{Check gate<br/>tests in a sandbox}}
    CG -->|red| B
    CG -->|green| R[Reviewer agent<br/>Codex, isolated]
    R -->|findings| T{Triage}
    T -->|fix| B
    T -->|clean| C[Commit — Henri confirms]
    C --> N[Handoff note + report]
    N -->|next ticket| B
    classDef next stroke-dasharray: 5 5
    class CG,R next
```

Dashed: the check gate and the isolated review are built and tested, but not wired in yet (see
[Checks and review without me](#checks-and-review-without-me)). Until they are, I start the review myself.

Seven rules carry the whole thing:

| Rule | Why |
|---|---|
| **The builder never reviews its own work.** | An agent checking its own diff shares the blind spots that produced it. A second model with fresh context catches different mistakes. |
| **Options before choices.** | Almost every step in this workflow narrows: questions, a spec, a review. Left alone, that means building the first idea that works. So a new feature starts with my own ideas written down, and only then does the agent widen them. In that order because AI suggestions shown first anchor you, and [a 2024 study](https://pmc.ncbi.nlm.nih.gov/articles/PMC11244532) found that AI help makes ideas more similar from person to person. |
| **Look before you build.** | Most of what a new app needs exists already: patterns that are known to work, libraries, tools, and parts I built in earlier projects. A short research pass before the questions means the grill decides between real options instead of me reinventing them. |
| **Questions before code.** | A misunderstood goal is cheapest to fix before anything is built. The agent asks only about intent, risk and trade-offs, and looks up facts itself. |
| **One ticket at a time, one commit per ticket.** | Small diffs are reviewable. A commit only lands after a clean review, and I approve each commit and each push. |
| **Handoffs live in files, not chat.** | Usage limits run out mid-task. Whoever picks up next, the other agent or a new session, continues from a note in the ticket instead of a chat history it can't see. |
| **The right model for each job.** | The main agent plans, talks to me, checks results and decides. Self-contained slices go to smaller, faster models that cost a fraction as much. Hard or risky slices stay with the main agent. |
| **Search first, report after.** | Before solving something, the agent checks whether we solved it already. After solving something hard, it writes down what happened. Solved problems stay solved. |

## The coding loop

Every ticket runs through ten steps. A new feature or project gets steps 0 and 0b first; fixes don't.
Claude Code builds by default and Codex reviews. When Claude's usage is exhausted the roles swap.

| # | Step | What happens | Why |
|---|---|---|---|
| 0 | Diverge | I answer the six W-questions (why, who, what, where, when, how) in one line each and write down up to three ideas in about five minutes. Then the agent adds a variant of each, a few deliberately odd ones (reverse the problem, drop an assumption) and one from the user's side. Everything goes into a mind map in the spec, and I mark what goes forward. | I tend to jump to the next idea before the first is written down. This makes me write it down, and it means the grill chooses between options instead of polishing the first one. |
| 0b | Research | Three agents look things up in parallel: principles (how this kind of app is built well), existing building blocks (my own past projects first, then open-source libraries and APIs), and tools to build with. Every claim cites its source. The results land in one table in the spec (what it gives us, licence, cost, maintenance, use / borrow / skip), and I mark the picks. | It comes after my own ideas so they stay unanchored, and before the grill so the questions can build on what already exists. |
| 1 | Scope and grill | Goal restated in at most 3 sentences, then one question at a time until the outcome, failure boundaries and acceptance examples are clear. | I'm better at answering a sharp question than writing a full spec from scratch. |
| 2 | Search | Grep past implementation logs for a prior instance. | Re-solving a known problem wastes a session and often re-introduces the old bug. |
| 3 | Spec | `spec.md` plus one file per ticket. Planned read-only, then a Six Hats check (facts, my gut feeling, risks, benefits, the best alternative we didn't take), then I approve. | The plan is the cheapest place to disagree. The hats make sure the review isn't only about risk. |
| 4 | Git check | No repo, no build. | Review needs a diff. |
| 5 | Build one slice | Small steps, project checks after each one, no unrelated refactors. Self-contained slices go to a cheaper subagent (see below); the main agent re-runs the checks itself. | Unrelated changes hide inside a diff and slip past review. |
| 6 | Review handoff | Builder lists what changed (max. 5 bullets) and the checks it ran, then hands over. Anything touching auth, secrets, money, deletion or external input gets an *adversarial* review. | The risky categories are the ones where "looks fine" isn't enough. |
| 7 | Triage | Each finding is fixed, rejected with a one-line reason, or brought to me. Never applied blindly. Big fixes trigger a second review. | Reviewers are wrong too. Blindly applying findings swaps one agent's mistakes for another's. |
| 8 | Commit | Conventional commit message, committed when I confirm. | I stay the one who decides what ships. |
| 9 | Handoff note | Date, agent, what's done, next step, open findings, uncommitted files, appended to the ticket. | The next agent starts from this, not from memory. |
| 10 | Report | Anything that took more than one try gets a facts-only report: commands, versions, exact error text, fix. | This feeds the knowledge engine below. |

### Who builds a slice

The main agent keeps the work that needs the conversation or judgment: questions, spec, triage,
anything hard. Slices that can stand on their own go to subagents on smaller models:

| Work | Who |
|---|---|
| Mechanical slice: follows an existing pattern, renames, fixtures, docs, config | Haiku |
| Standard slice: has acceptance examples, named files and a check command | Sonnet |
| Hard slice: unclear design, cross-module, auth, secrets, money, deletion | Main agent |
| Broad search across many files | Haiku |

```mermaid
sequenceDiagram
    autonumber
    actor H as Henri
    participant M as Main agent
    participant T as Ticket file
    participant G as Guard
    participant S as Subagent<br/>Haiku/Sonnet
    participant K as Memory<br/>rules, logs, wiki
    H->>M: approves spec and tickets
    M->>T: writes the brief
    Note over T: decisions, files,<br/>check command, out of scope
    M->>G: hands off one slice with a model
    alt no model
        G-->>M: blocked
    else Opus or Fable
        G->>H: approve or deny?
    else Haiku or Sonnet
        G->>S: starts it
    end
    S->>T: reads the brief
    S->>K: reads rules and logs
    Note over S: builds and checks the slice<br/>never sees the chat
    S->>T: appends its report
    S-->>M: done
    M->>M: re-runs checks, diff vs brief
    Note over M: hands over to Codex review
```

The subagent never sees the chat. Everything it needs travels through files: the brief in the ticket
on the way in, the report (files changed, check result, anything it did differently) on the way back,
and the project rules and past code logs that every agent reads. While it runs, a status line in
Claude Code shows the task and the exact model it got, e.g. `▸ slice 02 → Sonnet 5.5`.

Three rules make that safe:

- **The brief lives in the ticket.** A subagent sees its instructions and the project rules, but not
  my conversation with the main agent. So before handing off, the main agent writes the decisions
  that matter for this slice into the ticket. If that doesn't fit, it builds the slice itself.
- **"Done" is not evidence.** The subagent reports what it changed and what the checks said. The
  main agent re-runs the checks and compares the changed files with the brief before review.
- **Visible, not silent.** Every handoff names the exact model. A small plugin blocks any subagent
  without a model and asks me before one runs on an expensive model.

That plugin went through four adversarial reviews before it held. Each found another way past it:
agents started from a scripted workflow, a session changing directory, an agent started inside
another one, relative paths, a second symlink to the same folder. A rule that's enforced needs
the same review as code, because it is code.

**Stuck rule:** after two failed attempts at the same problem, the agent stops, writes down what it
tried, and hands over to the other agent. A third attempt from the same agent usually repeats the
first two.

## The design track

Visual design is my weak spot, so design gets more structure than code. How much depends on the
ticket:

| Ticket | Extra design steps |
|---|---|
| No UI | None |
| UI change in an existing project | Real-world references before designing, a design review against Apple's Human Interface Guidelines after |
| New project with UI | The full track below, starting at D0 |

| # | Step | Rule |
|---|---|---|
| D0 | Diverge | Step 0 for design: up to three directions, each a feel in 2–3 words plus one app I'd point to. The agent adds a few more, at least one deliberately the opposite. I mark which go forward. |
| D1 | Design grill | Every style word I use ("modern", "playful") gets pinned to one line of *means / does not mean*, plus one reference. |
| D2 | References | 3–5 real screens from shipped apps (via [Mobbin](https://mobbin.com)) for the same pattern and platform, one line each on what to borrow. |
| D3 | Design spec | Two tables: tokens (colour, type, spacing, radius) and motion (what moves, trigger, duration, easing, reduced-motion fallback). I approve them. |
| D4 | Prototype | One throwaway HTML mockup of the key screen with real motion, two variants at most, taken from the directions marked in D0. I pick one. |
| D5 | Build | Tokens only, no hardcoded values. Animate `transform` and `opacity` only. Nothing loops while idle. Respect `prefers-reduced-motion`. |
| D6 | Review | The reviewing agent, never the builder, checks fresh screenshots against the HIG, plus a recording, the reduced-motion state and measured contrast. |
| D7 | Record | The chosen direction goes into an architecture decision record so no later session re-decides it. |

**Why it looks like this.** In one app I gave three style briefs in a row: "liquid chrome", then
"colour", then "playful". The first two were built before anyone pinned down what the words meant,
and each cost a full redesign pass. The third was defined first. "Playful" ended up meaning five
static devices (tinted cards, a larger radius, pill tags that carry real metadata, one loud element
per screen) and no motion at all, because none of the reference apps got their playfulness from
movement. That's where D1 comes from.

D0 exists because everything after it narrows: the references are other people's screens and the
prototype allows two variants. Without it, my only design options were the first idea and borrowed
ones. D4 exists because reference screenshots show layout but not animation, so only a working prototype
can settle durations and easing. The idle rule in D5 comes from measurement: a live animated
background cost 3–4% of a CPU core continuously, which nobody wants behind a study session on
battery. The fix was to render it once at startup (about 16 ms) and never again.

One rule sits above all of it: design must never get in the way of function.

## Checks and review without me

Starting every review myself added up to about five touchpoints per ticket. The goal is two: I approve
the spec, and I approve the commit. Two pieces get there, and both run without me:

```mermaid
sequenceDiagram
    autonumber
    actor H as Henri
    participant B as Builder
    participant G as Check gate
    participant R as Review script
    participant C as Codex<br/>isolated
    B->>B: edits files
    B->>G: turn ends
    G->>G: tests of each edited project,<br/>in a sandbox
    alt red
        G-->>B: blocked: fix it
    else green
        G-->>B: done
    end
    B->>R: ticket + repo
    R->>C: fixed prompt, ticket, diff
    C-->>R: findings
    R-->>B: review file next to the ticket
    B->>B: triage every finding
    B->>H: one handoff
    H->>B: OK, commit
```

**The check gate.** When an agent's turn ends, the tests of every project it touched run on their own.
If one is red, the agent has to fix it before it can hand anything to me. After two failed attempts it
stops and tells me instead (the stuck rule). A new project can't slip past it: the first time an agent
edits one that has no test command listed, or a reason why it has none, the gate sends it back to add one.

The tests run in a macOS sandbox, because the agent can edit them and the gate runs them with my rights.
Without the sandbox, editing a test would be a way to run anything at all.

| A check can | A check can't |
|---|---|
| write inside its own project | write anywhere else, including git's own files |
| read the workspace and its toolchain | read anything else in my home folder (SSH keys, logins, app data) |
| talk to localhost (test servers) | reach the network |
| use macOS frameworks (atlas draws PDFs) | send Apple Events, change dependencies, build output or agent settings, or touch the scripts my hooks run |
| start background processes | leave any of them running once the check ends |

Every rule is there because a test proved it was needed: each one is removed in turn, and the test suite
has to fail. I also ran every project's tests with and without the sandbox, and each gave the same result
(four projects are red either way, for reasons that have nothing to do with the sandbox).
Some gaps are still open and have their own tickets. System services aren't filtered yet, so a check could
still ask macOS itself to start an app or a background job outside the sandbox, or reach the keychain. And
the last security review found a few more ways around the file and process rules (tool config files such as
`.npmrc`, a process that slips out of the end-of-check kill). The gate stays unwired until those are closed.

**The isolated review.** A script sends one ticket's uncommitted diff to Codex and writes the review into
a file next to the ticket. The builder picks only the ticket and the repo; the prompt is fixed. Codex runs
in a fresh, read-only process that sees none of my own setup and none of the repo's: no instruction files,
no hooks, no plugins, no memories, no skills. That matters because a file in the repo, or a comment in the
diff, can try to talk a reviewer into approving.

Turning those things off isn't proof that they're off. So a probe plants a marker everywhere Codex could
pick up instructions or run something, and runs three times. First without isolation, where every marker
has to fire, which proves the markers work. Then twice through the script itself, where none may fire.
The review script refuses to run on a Codex version the probe hasn't passed.

The script and the gate went through six adversarial reviews and a security review before they held.
Most findings were ways around them that I hadn't thought of: a git setting that runs a program while
reading the diff, a hard link, a space in a path, a dependency that one of my own hooks executes later.
To find the last kind, several agents traced in parallel what each of my hooks actually runs.

## Memory: the knowledge engine

AI agents forget everything between sessions. So the workspace keeps a memory of its own, as plain
Markdown:

| Layer | Holds | Rule |
|---|---|---|
| `raw/` | Unmodified sources: docs, articles, error dumps | Never edited. Corrections go into the note that cites it. |
| `wiki/` | Synthesis: how a library behaves, an architecture pattern | Owned by the agents. |
| `code_logs/` | One record per task: what was built, what broke, the fix | Written after every non-trivial task. |

Reports land in an inbox. A hook at the end of each agent turn starts a second agent that files them
into `wiki/` and `code_logs/`, regenerates the index, and commits. An index and a log are generated
by a script and never hand-edited, so they can't drift from the files.

Knowledge about code and knowledge about me are kept apart on purpose. The test before filing:
*would this still be true in someone else's codebase?* If yes, it goes in the wiki. If not, it goes
into my personal notes vault.

## Storage rules that came from breakage

My workspace lives in iCloud Drive so it syncs between machines. Two rules exist because that went
wrong:

- **Git's object store lives outside iCloud.** A repository inside iCloud got corrupted on this
  machine (`bad tree object HEAD`, broken reflog, a stale `index.lock`), because iCloud evicts and
  re-uploads files without knowing about git. Now every repo is created with
  `git init --separate-git-dir=~/claude-local/<project>/git`: only a small pointer file sits in
  iCloud, the history sits on local disk. `node_modules` and build output live there too, symlinked
  back.
- **One writer per log file.** Several agents appending to one log inside iCloud produced
  `LOG 2.md` conflict copies and lost entries. Now each writer appends to its own daily shard
  (`logs/<writer>/<YYYYMMDD>.md`), and the combined log is generated.

`.gitignore` files also block iCloud's conflict copies (`dist 2`, `node_modules 3`) by name, since
the usual patterns don't catch them and they'd otherwise be committed as source.

## What doesn't work yet

I tried to automate the loop: a controller that runs builder, verifier and reviewer as a state
machine, with me approving in a native dialog. It's built but not usable end to end. Automatic
review is switched off until I can prove the reviewer is properly isolated from the builder, and
four recovery bugs are still open. Until then the manual loop above is the workflow. I'd rather run
a slower process I trust than a fast one I can't check.

So I took a smaller route next to it: a check gate and an isolated review (see
[Checks and review without me](#checks-and-review-without-me)). Both are built and tested, but not wired
into my setup yet. Until they are, the review step above stays manual.

## Local models

Next to the hosted agents, I experiment with local models on my own hardware. My Windows PC has an
RTX 4090 and runs GLM-4.7-Flash. The MacBook (M5 Pro, 24 GB) handles `gpt-oss:20b` through Ollama,
and that's its ceiling: I tried Qwen3 14B and removed it the same day because the machine
overheated. A 20B mixture-of-experts model that only activates a few billion parameters per token
runs fine where a dense 14B model doesn't. So the parameter count on the label tells you less than
the architecture does.

Local models aren't part of the build-and-review loop above. They're where I test what runs
offline, privately and without usage limits.

## Examples

Each one uses the same format as the real files. The ADR and the code log are shortened from real
ones. The spec and the ticket are made up to show a full cycle in one place.

- [`examples/spec.md`](examples/spec.md): a feature spec after the diverge step and the grill
- [`examples/issues/01-due-badge.md`](examples/issues/01-due-badge.md): one ticket, with review triage and a handoff note
- [`examples/adr-0001-playful-without-motion.md`](examples/adr-0001-playful-without-motion.md): a design decision record
- [`examples/code-log.md`](examples/code-log.md): a facts-only implementation report

## Tools

[Claude Code](https://claude.com/claude-code) (builder),
[Codex](https://github.com/openai/codex) (reviewer), [Obsidian](https://obsidian.md) for notes,
[Mobbin](https://mobbin.com) for design references. The workflow itself is Markdown files, a few
small scripts and a Claude Code plugin that enforces the rules an agent could break by accident.
