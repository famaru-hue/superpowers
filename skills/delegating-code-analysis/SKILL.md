---
name: delegating-code-analysis
description: Use when you need to read, trace, or understand a nontrivial slice of existing code — architecture, call chains, cross-file dependencies, "how does X currently work" — before writing a plan, debugging, or judging a review comment
---

# Delegating Code Analysis

## Overview

Reading and understanding existing code costs the same context whether a
cheap model or an expensive one does it. When you're running as a capable,
expensive model, spending your own context on Grep/Read cycles across a
dozen files is the most wasteful way to burn your budget — the file
contents sit in your context for the rest of the session, re-read on every
later turn, when nothing about that task needed *your* reasoning power to
happen. It needed reading.

**Core principle:** the reading stage and the deciding stage don't need the
same model. Dispatch reading to `superpowers-analyst` (Sonnet, medium
effort) and keep your own context for the judgment calls only you can make.

## When to Use

**Delegate when the investigation spans multiple files, traces a call chain,
or requires piecing together how a subsystem works** — the kind of question
that would otherwise cost you several Read/Grep round trips.

**Don't delegate a single quick lookup** — one Grep for a symbol, reading
one function you already know the location of. Dispatch overhead (writing
the brief, waiting, reading the report) costs more than just doing it
yourself when the answer is one tool call away.

## How to Dispatch

Give the analyst two things: the absolute repository root as `SCAN_ROOT`,
and **one specific question** — not "understand this codebase." Vague
dispatches produce shallow, padded reports.

```
Dispatch superpowers-analyst:
  SCAN_ROOT: /abs/path/to/repo
  Question: "Trace how a saved-goal withdrawal updates `Disponible`,
  from the UI action through MotorFinanciero to the database write.
  List every function on that path and what each one assumes about
  its caller."
```

## What Comes Back

A structured brief, not pasted source: the direct answer, the relevant
`file:line` locations, the functions/modules involved, the execution flow,
relevant architecture and dependencies, current behavior, problems or
constraints found, and any gaps it couldn't resolve. See
`agents/superpowers-analyst.md` for the exact contract — don't redefine it
here.

## Use the Report — Don't Re-Read What It Already Covered

The report is your context now. Reason and act from it directly. Reading
the files it already covered again defeats the entire point of
dispatching.

The one legitimate reason to read a file yourself afterward: the report
names a concrete gap — a function it flagged but didn't quote, a claim that
doesn't line up with something else you know, a question it says it
couldn't answer. Read *that* narrow spot, not the whole file "just to be
sure." If the report reads as internally inconsistent or contradicts
evidence you already trust, that's also grounds for a targeted re-check —
general unease is not.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Dispatching for a one-line lookup | Just Grep/Read it yourself — delegation overhead loses |
| Vague question ("understand the auth system") | Ask the one thing you actually need next |
| Re-reading files the report already covered, out of habit | Trust the report; read only a named gap |
| Pasting the report's file list into your own Read calls "to double check" | That's re-doing the work you just paid to avoid |
