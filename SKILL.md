---
name: comment-audit
description: Audit whether each comment in source code deserves to exist, delete the unnecessary or harmful ones, and trim the ones worth keeping to the fewest words. Always use this when asked to clean up, prune, review, or audit comments; when the user says "too many comments", "remove the AI-looking comments", or "the comments still describe the task history or what the user asked for"; and when polishing an implementation before committing or judging comment quality in a PR review. Also use it as the standard for deciding, while writing code, whether a comment belongs on a given line. Covers removing AI-generated, redundant, change-narrating, and conversation-leaking comments.
---

# Comment Audit

You are the auditor of comments. Look at every comment in the target code and ask one question: does this have the right to exist? Delete the ones that do not, rewrite the ones that do but are badly written, and protect the ones that do.

This is not a procedure. It is a way of thinking to judge from. Derive the language- and project-specific details from it yourself.

## Starting point: a comment is an instruction with no expiry date

Comments are not executed, but they are read. And today the primary reader of code is an AI agent. Agents take comments as authoritative — on par with the code, sometimes above it. When a comment and the code disagree, an agent may believe the comment. A comment therefore acts as a prompt with no expiry date, applied to every future session that opens the file.

So the harm goes well beyond "verbose and hard to read."

- A single line saying "done this way at the user's request" promotes one task's in-the-moment decision into a permanent product requirement. The next agent treats it as an immovable constraint and distorts the design to work around it.
- Whose request, when, and how far it reached cannot be verified from the comment. An unverifiable authority takes up residence in the codebase.
- A comment that merely restates the code becomes a lie the moment the code changes. A lying comment is worse than no comment.

Meanwhile, the reader can read code perfectly well. What it does and how it works can be understood from the code. Saying it again in natural language adds no information and only adds the risk of disagreement.

From this follows the default stance. **The default is no comment. A comment is permitted to exist only when it carries information that cannot be recovered from the code by any means.**

## Why unnecessary comments appear

Knowing where they come from makes them faster to spot.

- **Mistaking the audience.** An agent writes code while talking to the person it is conversing with. Reports and justifications addressed to that person ("as requested", "fixed", "changed from X") end up in a place that should address the file's future readers. The reader of a comment is not the other party in the conversation.
- **Leaving session memory behind.** Agents carry no memory across sessions, so they write what they learned during investigation, what they tried that failed, and the history of a bug into the nearest persistent place: the source file. The longer the session, the stronger this tendency. The content is commit-message or working-note material, not an explanation of the code.
- **Tutorial voice.** Training data contains a great deal of code that explains itself line by line for beginners. That voice is carried into production code.
- **Forgotten scaffolding.** Writing "next, do X" before writing the code helps generation, but once the code is written the scaffolding is no longer needed.
- **Matching the neighbors, and accretion.** Where comments are dense, more get added at the same density. Nobody removes any, so blocks only grow, commit after commit. A long comment block is usually the sediment of several sessions.

## Three questions that grant the right to exist

Ask them of every comment. Only a comment that answers "yes" to all three stays.

1. **Does it carry information that cannot be obtained from the code?** Something that careful reading of the code, names, types, and tests would not reveal. If it restates what reading reveals, it has no right to exist.
2. **Does it read as a currently true fact to a reader who knows nothing?** Someone who has seen neither this conversation, nor this diff, nor any previous version opens the file a year from now. Does it make sense, and is it correct? If it needs words like "previously", "changed", "fixed", "added", or "addressed", it is history, not a description of the present.
3. **Would its absence at this line cause harm?** If removed, does it become more likely that someone will break this code, or ruin it while meaning to "improve" it? If not, it can go.

## What to delete

- **Restatements of the code.** Play-by-play of the logic, repetition of names, types, or signatures, markers for the end of a block, obvious headings.
- **Narration of change.** What was fixed, what was changed from what, what was removed, where something moved. Obituaries for code that used to be here. The diff and the commit message already record all of it.
- **Conversation leakage.** "At the user's request", "as pointed out", "per review feedback", "as asked". References to the requester, the AI, or a reviewer; trial and error or investigation during a session; dated work logs.
- **Unverifiable appeals to authority.** "Per the spec", "required by requirements", with nothing behind them. References to a section number or a work ticket alone, with the reason itself absent. The target will move and be lost.
- **Stale content.** Descriptions of code that no longer exists or behavior that has changed. The code is always the truth; never trust the comment and doubt the code.
- **Commented-out code.** Version control already keeps it.
- **Decoration.** Emphasis markers, emoji, signatures, opinions, defensive disclaimers.

## What to keep

The "why" that cannot be written in code, and the "contract" that cannot be seen from where the reader stands.

- **Why the obvious way is not used.** Why a simpler-looking version is wrong. Comments that protect against well-meaning simplification are the most valuable of all.
- **Invariants and constraints not visible from here.** Ones that originate in another file, an external system, execution order, concurrency, or assumptions about data. This includes cross-file correspondences such as "keep in sync with X".
- **Workarounds for external quirks.** Bugs or traps in a library or API, the workaround, and the condition under which it becomes unnecessary.
- **Contracts the signature cannot express.** At a public boundary: units, value ranges, side effects, behavior on failure, the caller's obligations.
- **Meaning of literals.** Units, ordering, or provenance that a number or a table cannot state for itself.
- **Domain facts.** Business or regulatory rules that cannot be derived from looking at the code, and their reasons.
- **Markers of unfinished work.** TODO / FIXME pointing at work that is actually unfinished. Deleting one hides incompleteness.

Once you decide to keep a comment, a separate question remains. **"Worth keeping" and "worded minimally" are independent judgments.** A comment with a real reason in it is often three times too long. Only the non-obvious fact the reader needs at that line, in the fewest words. Be suspicious of long blocks; when several points are mixed together, split them and place each beside the line it governs. The closer a comment sits to its code, the more likely it is updated with it. But do not reword a comment that is already concise and correct to suit your taste. An unnecessary diff is noise too.

## Comments that mix requests and history

This is where the most care is needed. A comment such as "at the user's request, no retry here" tangles together an **attribution** that must go (who said it, when) and a **constraint** that may need to stay (must not retry).

- Always remove the attribution and the history. "Someone said so" is not a reason.
- Put the constraint through the three questions. If the code expresses the behavior plainly, no comment is needed. If the code alone looks like a mistake or a shortcut that someone is likely to "fix", rewrite the constraint as a present-tense fact, with its reason.
- When no reason can be found in the comment or its surroundings, **do not invent one**. A plausible fabricated reason is worse than the thing you are removing. Accept that it may have been nothing more than an in-the-moment decision, delete the comment, and raise it in the report as an item needing human confirmation. If it truly is a permanent requirement, its place is not a comment but a test, a spec, or a design document.

```ts
// Before: Changed to not retry here at the user's request (previously retried 3 times)
// After (reason confirmed): No retry: the payment API is not idempotent and would double-charge.
// After (reason not confirmed): no comment. Report: "Reason for not retrying is unknown. If it is a requirement, pin it with a test."
```

## Information has a proper home

Deleting is usually not discarding information but returning it to where it belongs. Holding this distinction lets you delete without hesitation.

| Information | Home |
|---|---|
| What it does | The code itself, names, structure |
| What changed and why | Commit message, PR description |
| Requirements or behavior that must hold | Tests, types, assertions |
| The overall story of why the design is this way | Design docs, ADRs, README |
| Findings during work, course of an investigation | Work log, issue tracker. Not the source |
| The "why" and "contract" that only mean something at this line | A comment |

The scope of this audit is comments. It does not write documents or add tests. When information that should move is found, propose it in the report.

## What not to touch

- **Machine-read comments.** Directives to linters or compilers, type-check suppressions, code-generation markers, shebangs, encoding declarations. These are code in the shape of a comment. The reason attached to a suppression, however, is subject to the audit.
- **License and copyright notices.**
- **Code.** This audit does not change code. No renames, no extractions. When a comment compensates for unclear code, it is doing real work: keep it and report the spot as a refactoring candidate.
- **The project's explicit conventions.** If there is an agreement such as "public APIs must have doc comments", follow it. But apply the same questions to the content: reduce a doc comment that only repeats the signature to the minimum the convention requires, and protect one that states a contract.

The same leakage ("verifies the fix for the reported bug") occurs in test names and test descriptions. When they are within the scope of the request, apply the same thinking.

## When in doubt

The cost of error is asymmetric.

- In doubt about a comment that explains *how*: delete. What is lost is still in the code.
- In doubt about a comment that explains *why*: keep. Deleting one real warning costs far more than leaving one mediocre comment.
- A comment whose correctness cannot be confirmed: check it against the code. If it still cannot be decided, keep it and raise it in the report.

## A warning to the auditor

You have the same habits as the writer of unnecessary comments.

- Do not put a new comment where a deleted one was. "Removed unnecessary comments" is exactly the kind of comment this audit removes.
- The diff tells what was deleted. The report and the commit message tell why. Leave nothing in the file.
- Do not decide how much to keep by the surrounding comment density. The only standard is the three questions.
- Do not mistake volume for results. Sometimes almost everything goes; sometimes nothing does.

## Relation to prevention

The audit is an after-the-fact remedy; its effect lasts only when combined with standing instructions to the agent that writes the code. When the user wants comments not to be written in the first place, or when the same kinds of comments recur audit after audit, read `references/writing-guidance.md` and propose guidance for AGENTS.md / CLAUDE.md.

## Report

Keep it brief. There is no need to list each deleted play-by-play comment; that is the purpose of the audit, not news. What to convey is what needs human judgment:

- Constraints whose reason could not be confirmed (a request to confirm whether they are requirements)
- Places where a comment was compensating for unclear code (refactoring candidates)
- Information that should move to documents or tests
- Borderline decisions (one line each, so they can be overruled)
