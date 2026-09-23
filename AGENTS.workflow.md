# Global Luna and Sol coding workflow

Use `gpt-6-luna` with `max` reasoning as the default Codex model.

## Programming tasks

- Small, clear, localized programming tasks stay entirely with Luna max. Luna
  inspects the relevant code, makes the change, and runs focused validation.
  Do not call Sol just because a small task involves more than one file.
- Large or uncertain programming tasks go to a fresh `sol_coder` agent using
  `gpt-6-sol` with `max` reasoning. Luna first gathers only the evidence needed
  to define the task, then sends a focused Task Packet of roughly 1,000-3,000
  tokens. Sol implements and tests within that scope. Luna reviews the diff and
  verifies the result before reporting completion.
- Treat a task as small when its behavior is clear, the change is localized,
  and it has no meaningful architecture or cross-component decision. Treat it
  as large when it requires architecture or public-interface choices,
  coordinated changes across components, a new multi-part feature, a difficult
  bug that remains unclear after one focused investigation, or material
  concurrency, security, compatibility, performance, or numerical judgment.
  Do not use a fixed line-count threshold. Promote a task to Sol if evidence
  gathered during a small-task investigation reveals one of these conditions.

## Non-programming requests

For questions, explanations, and research requests that do not require code
changes, Luna gathers any necessary evidence, sends a concise Task Packet to a
fresh read-only `sol_reasoner` using `gpt-6-sol` with `max`, waits for its
analysis, then checks and delivers the answer.

## Sol handoff rules

Use at most one Sol agent at a time. For each request that requires Sol, spawn a
fresh agent with no inherited conversation history (`fork_turns="none"`) and
wait for its completed response. Never reuse a Sol thread from another request.
Sol receives only the Task Packet and its explicitly named evidence; never send
the whole conversation, repository, broad file dumps, long logs, credentials,
or unrelated context.

If a required Sol agent cannot start, times out, reaches a usage or concurrency
limit, or returns no usable result, stop that route and ask the user whether to
retry or explicitly waive Sol for that request. Small programming tasks do not
require Sol and may continue with Luna.

Use this Task Packet structure:

```text
TASK PACKET

GOAL
One precise outcome.

RELEVANT CONTEXT
Only facts that change the implementation or decision.

KEY CODE
Exact file paths and symbols, with only small essential excerpts.

CONSTRAINTS / INVARIANTS
Compatibility, performance, safety, scope, and validation constraints.

ATTEMPTS / EVIDENCE
What Luna inspected or tried, key output, and conclusions already ruled out.

QUESTIONS / DELEGATED WORK
What Sol must decide or implement.

EXPECTED RESPONSE
For `sol_coder`: implementation summary, changed files, tests, and risks.
For `sol_reasoner`: answer or decision, rationale, and material caveats.
```

Complex coding packets should be roughly 1,000-3,000 tokens. Simple research
packets should be as short as possible and never padded. If a packet would
exceed 3,000 tokens, narrow the task or summarize more. If Sol requests more
information, Luna gathers only the requested evidence and sends a focused
follow-up packet.

The `sol_coder` may edit code and run relevant tests only within the requested
scope. It must not make unrelated changes. Luna owns final review and
verification. If validation reveals a new material decision, send a new
focused packet rather than forwarding all prior context.
