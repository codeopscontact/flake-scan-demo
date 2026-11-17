# SAMPLE — Flake Report: checkout totals include tax after discount

> **This is a sample report with invented data**, published by CodeOps to show the report format used in a Flaky Test Repair Sprint. The project, runs, and numbers below are illustrative, not a real client engagement.

<!-- Title format used in a real tracker: `flake: <test name> (<class>)` -->

## Test

- **Test:** `tests/checkout/totals.test.ts :: checkout totals include tax after discount`
- **Suite / job:** CI job "unit (shard 2)", GitHub Actions, ubuntu-latest
- **First seen:** 2026-10-27

## Evidence

- **Failed run:** run #1842 (example) — commit: `9f3ac21`
- **Passed re-run of the same commit:** run #1843 (example), no code change
- **Failure output / error message:**
  ```
  expect(received).toBe(expected)
  Expected: 108.25
  Received: 100.00
    at Object.<anonymous> (tests/checkout/totals.test.ts:57:24)
  ```
  (Received value equals the pre-tax total — the tax step had not been applied when the assertion ran.)
- **Seed / test order:** order recorded by the runner; failing runs all had `totals.test.ts` scheduled after `promo.test.ts` on the same worker
- **How often:** 5 failures in the last 60 runs of this job (≈8%)

## Classification

- [x] **A — Timing / race** (sleeps, un-awaited async, real-clock assertions)
- [ ] B — Shared state
- [ ] C — External dependencies
- [ ] D — Resource limits
- [ ] E — Ordering dependence

Primary class A: the assertion runs against a total that is recalculated asynchronously; whether the recalculation has landed depends on timing.

## Reproduction

- **Loop result:** failed 4/50 running the single test file in a loop on CI; failed 0/50 locally on a fast machine
- **Conditions that make it worse:** only under CI CPU contention; worse when the shard runs with 4 parallel workers

## Impact

- **Merges blocked / re-runs caused this week:** ~6 re-runs, 2 delayed merges
- **In the current top-10 flake ranking?** yes (#3)

## Disposition

- **Owner:** CodeOps (sample)
- **Action:** fix now
- **Verification bar for closing:** 0 failures in ≥ 200 runs

## Resolution

- **Root cause:** The test applied a discount, which triggers an asynchronous price recalculation (tax included), and asserted on the total without awaiting the recalculation. On a loaded runner the assertion regularly won the race and read the stale pre-tax total.
- **Fix:** await the recalculation's completion signal in the test setup instead of asserting immediately (sample PR: #412, example link)
- **Verified:** 0 failures in 300 loop iterations on CI after the fix; flake-scan summary for the verification run: 300 run, 300 passed, observed flake rate 0.0%
