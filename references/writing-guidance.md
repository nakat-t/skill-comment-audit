# Prevention at writing time: guidance for AGENTS.md / CLAUDE.md

The audit is an after-the-fact remedy. To reduce unnecessary comments from the start, standing instructions to the code-writing agent must be used alongside it. Neither alone is enough: standing instructions lose force over a long session, and the audit has no effect until it is run.

When the user asks for instructions of this kind, propose the following as a base.

## Conditions for the instructions to work

- **Give a standard, not just a prohibition.** "Don't write comments" alone either loses the warnings that are needed or gets ignored. Show what a comment with the right to exist looks like.
- **Attach the reason.** Explaining the harm — that a comment gets treated as a requirement in future sessions — extends judgment to situations the rules do not cover.
- **Give the urge somewhere to go.** The impulse to record history and report progress does not go away. Point to the correct outlets: the commit message and the final report.
- **Short and free of contradictions.** Check that no opposite rule such as "comment complex logic" remains elsewhere.

## Example instructions

```markdown
## Comments

A comment acts as an instruction with no expiry date, addressed to every person and agent who opens this file in the future.
Writing an in-the-moment decision or the content of a conversation into one makes later sessions treat it as a permanent requirement and distort the design around it.

- Default to no comment. Do not write what reading the code reveals.
- Write only the "why" the code cannot show and constraints not visible from that spot: as present-tense facts, in the fewest words.
- Do not write a sentence that would not make sense to a reader who has seen neither this conversation nor the diff.
  Change descriptions, history, and references to the requester ("as requested", "fixed", "previously", etc.) go in the commit message and the work report.
- Do not delete or reword an existing correct comment unless you change the code it describes. Do not add comments to match the density of the surroundings.
- At the end of the task, review the comments you added against the standard above.
```
