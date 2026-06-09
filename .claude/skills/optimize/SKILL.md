---
name: optimize
description: Analyze a codebase for performance and produce an optimization report — where the program is slow, how to make it faster, and the expected speed impact, effort, and risk of each change. Use when the user asks to "optimize the program", "make it faster", "improve performance", "find bottlenecks", "why is this slow", or "reduce latency/memory". Reads and understands the code first, reasons from profiling and algorithmic complexity (not guesses), and reports recommendations ranked by impact — without rewriting hot code until asked.
---

# Performance Optimization Report

Guiding idea: **the fastest program is the one that does the least work on the most
cache-friendly data — everything else is detail.** Your job is to find where the
program does too much work or touches memory badly, and write a clear report on how
to fix it and what speedup to expect. **Report first; don't rewrite hot code until
the user approves** — performance changes can alter behavior and hurt readability.

## Step 0 — Measure, don't guess (most important rule)

Optimization without measurement is superstition. The bottleneck is rarely where
intuition says. Per **Amdahl's Law**, optimizing code that's 1% of runtime yields at
most 1% — effort must go to the actual hot path.

- Look for existing benchmarks/profiles. If none, **recommend how to profile this
  stack** (e.g. `cProfile`/`py-spy` for Python, `perf`/`flamegraph` for native,
  Chrome DevTools/`--prof` for JS, `pprof` for Go, JMH for Java) and what workload to
  measure under.
- Be explicit about confidence: mark each finding as **measured** (backed by a
  profile/benchmark) vs **suspected** (inferred from reading the code). Never present
  a guess as a proven hotspot.

## Step 1 — Understand the codebase

- Map the architecture, entry points, and the hot paths users actually wait on
  (request handlers, render loops, batch jobs, tight inner loops).
- Identify the language/runtime — it changes what matters (GC pressure in
  managed langs, cache layout in native, event-loop blocking in JS, the GIL in
  Python, allocations everywhere).
- Note the data: sizes, shapes, access patterns, and where data crosses boundaries
  (DB, network, disk, serialization).

## Step 2 — Analyze against the five principles

For each, find concrete issues and cite `file:line`.

### 1. Do less work — algorithmic complexity (biggest wins)
- Better algorithms/data structures beat micro-tweaks. O(n²)→O(n log n) dwarfs any
  constant-factor gain. Look for: nested loops over the same data, linear scans that
  should be hash/index lookups, repeated sorting, quadratic string building.
- Eliminate redundant computation: hoist invariants out of loops, **memoize/cache**
  pure results, avoid recomputing what hasn't changed.
- Watch for hidden O(n) per call (e.g. `list.contains` in a loop) and **N+1 queries**.

### 2. Respect the memory hierarchy — data locality
- CPUs are starved by memory, not compute; a cache miss costs ~100× an arithmetic op.
- Prefer **contiguous data** (arrays over linked lists / pointer-chasing object
  graphs), **sequential access** (prefetcher-friendly), and packing hot fields tightly
  so the working set fits in cache. Flag structure-of-arrays vs array-of-structures
  opportunities for hot loops (data-oriented design).

### 3. Reduce overhead
- Cut allocations and **GC pressure** (reuse buffers, pool objects, avoid boxing).
- **Batch** I/O and system calls (one bulk DB query/write, not per-row); buffer.
- Don't pay for abstractions you don't need in hot loops (excess indirection, virtual
  dispatch, deep call chains, needless copies).

### 4. Exploit parallelism
- Use cores (threads/async), **overlap I/O with compute** instead of blocking,
  parallelize independent work. Note SIMD/vectorization chances in numeric loops.
- Reduce dependency chains and unpredictable branches that stall the pipeline.
- Be honest about cost: concurrency adds bugs/locks — recommend only where the win is
  real and the work is actually independent.

### 5. Right-size everything else
- Connection pooling, lazy vs eager loading, compression trade-offs, caching layers,
  appropriate data types and indexes.

## Step 3 — Write the report

Rank findings by **impact**, not by how easy they are to spot. For each:

- **Location** — `file:line` and the hot path it's on.
- **Principle** — which of the five.
- **Problem** — what work/memory cost is being paid, with complexity (e.g. "O(n²)
  over `users`, n≈10k").
- **Fix** — concrete change, with a corrected snippet where useful.
- **Expected effect** — speed/memory impact, framed honestly: a rough magnitude
  ("~n× fewer comparisons", "removes ~1 DB round-trip per row") and a note that
  **Amdahl's Law caps the gain by this path's share of total runtime** — so tie it to
  how hot the path actually is.
- **Effort & risk** — small/medium/large; does it change behavior or readability?
- **Confidence** — measured vs suspected.

Suggested layout:

```
# Optimization Report — <scope> — <date>

## How this was assessed
<profiled? or read-only inference? what workload>

## Top opportunities (ranked by impact)
### 1. [HIGH] N+1 queries in order listing — Principle 1 (do less work)
- Location: src/api/orders.js:48 (per-request hot path)
- Problem: 1 query + N per-row queries; O(n) round-trips, n = #orders.
- Fix: single JOIN / batched `WHERE id IN (...)`.
- Expected effect: ~N fewer DB round-trips; if this path is ~60% of request time,
  expect a large latency drop. Confirm with a before/after benchmark.
- Effort: small · Risk: low · Confidence: suspected (profile to confirm)

## Lower-priority / micro-optimizations
## Not worth it / premature
<things that look slow but are cold paths — Amdahl says skip>
```

End with: *"Want me to implement any of these? Tell me which and I'll apply it and
re-measure."*

## Rules

- **Measure before claiming.** Distinguish measured hotspots from suspected ones;
  recommend profiling when there's no data. Don't dress up guesses as facts.
- **Impact-ranked, Amdahl-aware.** Push algorithmic/hot-path wins first; call out
  premature/cold-path optimizations as not worth it.
- **Report, don't rewrite** hot code until approved. Note when a speedup costs
  readability or changes behavior — that's a trade-off the user should choose.
- **Correctness first.** A faster wrong answer is worthless; preserve behavior and
  recommend re-running tests/benchmarks after any change.
