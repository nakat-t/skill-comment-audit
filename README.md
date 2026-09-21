# Comment Audit (Agent Skill)

An [Agent Skill](https://www.anthropic.com/news/skills) that audits every comment in your source
code against one question — **does this have the right to exist?** — deletes the ones that do not,
and trims the ones that do to the fewest words.

It targets the comments an AI agent leaves behind in particular: play-by-play narration of the
logic, records of what was changed and fixed, and leakage from the conversation
(*"changed at the user's request"*). It covers **comments only** — it does not rewrite code, add
tests, or write documents.

## Install

```sh
npx skills add nakat-t/skill-comment-audit --skill comment-audit
```

Add `-g` to install it globally (user-level) instead of into the current project:

```sh
npx skills add nakat-t/skill-comment-audit --skill comment-audit -g
```

## How to use

The skill is meant to trigger on its own — when you ask to clean up, prune, or review comments,
when you say "there are too many comments" or "remove the AI-looking ones", and when you polish an
implementation before committing. It also serves as the standard for deciding, while writing code,
whether a comment belongs on a given line at all.

Or invoke it explicitly:

```text
Use the comment-audit skill on src/payment/.
```

### What an audit looks like

A comment that tangles an **attribution** that must go together with a **constraint** that may need
to stay:

```ts
// Changed to not retry here at the user's request (previously retried 3 times)
```

The attribution and the history always go. The constraint is then put through the three questions —
and the outcome depends on whether a real reason can be found:

```ts
// Reason confirmed:
// No retry: the payment API is not idempotent and would double-charge.

// Reason not confirmed:
// (no comment — raised in the report instead)
```

When no reason can be found, the skill **does not invent one**. A plausible fabricated reason is
worse than the comment it replaces. The line is deleted and the question is handed back to you:
*"Reason for not retrying is unknown. If it is a requirement, pin it with a test."*

## What's inside

### Why a comment is not harmless

- **A comment is an instruction with no expiry date.** Comments are not executed, but they are
  read — and today the primary reader of code is an AI agent. Agents take comments as
  authoritative, sometimes above the code itself. A comment is a prompt applied to every future
  session that opens the file.
- **One line can turn a moment's decision into a permanent requirement.** "Done this way at the
  user's request" is unverifiable — whose request, when, how far it reached — and the next agent
  treats it as an immovable constraint and distorts the design around it.
- **A comment that restates the code becomes a lie the moment the code changes.** A lying comment
  is worse than no comment.
- **So the default is no comment.** A comment is permitted to exist only when it carries
  information that cannot be recovered from the code by any means.

### Three questions that grant the right to exist

Only a comment that answers *yes* to all three stays.

1. **Does it carry information that cannot be obtained from the code?** Not from careful reading of
   the code, the names, the types, or the tests.
2. **Does it read as a currently true fact to a reader who knows nothing?** Someone who has seen
   neither the conversation nor the diff opens the file a year from now. Words like "previously",
   "changed", "fixed", or "addressed" mark it as history, not description.
3. **Would its absence at this line cause harm?** Would removing it make it more likely that
   someone breaks this code, or ruins it while meaning to improve it?

### What goes, and what stays

| Deleted | Kept |
|---|---|
| Restatements of the code, block-end markers, obvious headings | **Why the obvious way is not used** — the most valuable comments of all |
| Narration of change: what was fixed, changed, moved, removed | Invariants and constraints not visible from this spot (another file, execution order, concurrency) |
| Conversation leakage: "at the user's request", "as pointed out", session trial and error | Workarounds for external quirks, and the condition under which they become unnecessary |
| Unverifiable appeals to authority: "per the spec", a ticket number with no reason attached | Contracts the signature cannot express: units, ranges, side effects, failure behavior |
| Stale descriptions of code that no longer exists | The meaning of literals, and domain facts that cannot be derived from the code |
| Commented-out code, decoration, emoji, disclaimers | TODO / FIXME pointing at work that is genuinely unfinished |

### Worth keeping ≠ worded well

Two independent judgments. A comment with a real reason in it is often three times too long: keep
only the non-obvious fact the reader needs at that line. Long blocks get split, each point placed
beside the line it governs. But a comment that is **already concise and correct is left alone** — an
unnecessary diff is noise too.

### Information has a proper home

Deleting is usually not discarding information but returning it to where it belongs.

| Information | Home |
|---|---|
| What it does | The code itself, names, structure |
| What changed and why | Commit message, PR description |
| Requirements or behavior that must hold | Tests, types, assertions |
| Why the design is this way | Design docs, ADRs, README |
| Findings during work, course of an investigation | Work log, issue tracker. Not the source |
| The "why" and "contract" that only mean something at this line | A comment |

### What it never touches

- **Machine-read comments** — linter and compiler directives, suppressions, codegen markers,
  shebangs, encoding declarations. These are code in the shape of a comment.
- **License and copyright notices.**
- **The code.** No renames, no extractions. Where a comment compensates for unclear code, it is
  doing real work: it stays, and the spot is reported as a refactoring candidate.
- **Your project's explicit conventions.** If public APIs must carry doc comments, they do — but
  one that merely repeats the signature is cut to the minimum the convention requires.

### When in doubt

The cost of error is asymmetric. In doubt about a comment explaining **how**, delete — what is lost
is still in the code. In doubt about one explaining **why**, keep — losing one real warning costs
far more than leaving one mediocre comment.

### Prevention at writing time

The audit is an after-the-fact remedy, and its effect lasts only alongside standing instructions to
the agent that writes the code. When you want the comments not to be written in the first place, the
skill reads `references/writing-guidance.md` and proposes a section for your `AGENTS.md` /
`CLAUDE.md`.

### The report

Kept brief, and deliberately not a list of everything deleted — that is the purpose of the audit,
not news. What it surfaces is what needs your judgment: constraints whose reason could not be
confirmed, comments that were compensating for unclear code, information that should move to a test
or a document, and borderline calls, one line each, so you can overrule them.

## License

MIT. See [LICENSE](LICENSE).
