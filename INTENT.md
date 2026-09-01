# mio — Intent

This file states what the project is, why it exists, and what it deliberately is not. Everywhere else in the spec layer the code has the last word: when `README.md` or `SPEC/` disagrees with it, the code wins and the document is updated. Here that reverses — when this file disagrees with the code, the code is what is suspect.

## What

mio (澪) keeps a system's core representation: a graph tracing requirements to evidence — statements of how each requirement should be verified — to the implementations of that evidence, with the production code behind each requirement derived mechanically from what those implementations exercise. It is exposed as a CLI and an MCP server, with bundled skills — so that any consumer, an agent, a human reader, or another system, can draw a bounded view of the system through the same channel.

## Why

Neither AI agents nor humans can hold a whole system in context. mio marks the navigable channel: from any requirement, reach exactly the code that realizes it (progressive disclosure) — and what has no channel (unverified requirements, undocumented behavior, unreachable code) surfaces mechanically.

## Non-goals

- Not a test framework or runner. mio reads what evidence implementations exercise; it does not own their execution.
- Not a code-generation pipeline. mio represents and reveals the system; it does not build it.
