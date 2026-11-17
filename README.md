# flake-scan-demo

A small public demo of how [CodeOps](https://github.com/codeopscontact) measures test flakiness before fixing it.

Most flaky-test work starts with opinions ("it fails sometimes"). A repair sprint starts with a number: **how often does this test fail, under real CI conditions?** This repo contains the measuring instrument.

## What's here

- **`.github/workflows/flake-scan.yml`** — a GitHub Actions workflow that runs a chosen test command N times on a real runner (on a weekly schedule, or on demand from the Actions tab) and writes the pass/fail count and the observed flake rate into the job summary. Per-iteration logs are uploaded as an artifact, so a failure you measured is a failure you can inspect.
- **`SAMPLE-FLAKE-REPORT.md`** — a filled-in example of the report format used to track one flake from first sighting to verified fix (sample data, clearly marked).

## Why a schedule?

A flake you only look for when CI turns red is a flake you never quantify. The scheduled run keeps a baseline: if a test's failure rate creeps from 0% to 4% over a month, you see it in the summaries before it starts blocking merges.

## Using the workflow in your own repo

1. Copy `.github/workflows/flake-scan.yml` into your repo.
2. Set the default `command` input to a single suspect test (one test, not the whole suite — rates are per-test), and add your project's setup steps where the file is marked.
3. Run it from the Actions tab, or let the weekly schedule run it. For a suspected ~1-in-10 flake, use at least 100 iterations before drawing conclusions.

## The full method

This demo is one instrument from the **Flaky CI Rescue Kit** (USD 49) — measurement, a five-class cause taxonomy, per-class fixes, and quarantine guardrails: https://payhip.com/b/k36mN

If you'd rather have it done for you, the **Flaky Test Repair Sprint** (fixed USD 500) root-causes your recurring flakes and delivers the fixes as PRs with before/after reproduction rates. Contact: codeops.contact@gmail.com

— CodeOps · https://payhip.com/CodeOps
