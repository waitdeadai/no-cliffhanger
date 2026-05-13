# no-cliffhanger

[![tests](https://github.com/waitdeadai/no-cliffhanger/actions/workflows/test.yml/badge.svg)](https://github.com/waitdeadai/no-cliffhanger/actions/workflows/test.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-hook-orange)](https://code.claude.com/docs/en/hooks)

> A Claude Code Stop hook that blocks dangling *"want me to continue?"* / *"let me know if you'd like me to..."* / *"happy to expand"* permission-loop endings, so the model either does the next thing or closes honestly with a partial status — never makes the operator re-authorize work that was already authorized.

`no-cliffhanger` is one bash file (~80 lines, depends only on `jq`) wired into Claude Code's `Stop` and `SubagentStop` events. It inspects the last 320 characters of every outgoing assistant message and pattern-matches the dangling permission-loop vocabulary that LLMs default to at message end. When matched, it blocks with a repair-guidance template that tells the model to either complete the next piece of work or use the partial-closeout shape.

Two allow-clauses keep the hook out of the way of legitimate uses:

- A message that ends with `Status: partial` / `Status: blocked` / `Status: verified` / `Next step:` (the verification-framework shape) passes through.
- An explicit Y/N or multiple-choice question (`(y/n)`, `pick one of: A) ... B) ...`) passes through — when a real decision is needed, ask explicitly.

## Why this exists

Permission-loops are the cousin of sycophancy: model behavior trained for politeness that ends up making the operator do work the model could just do. *"Should I proceed with the next file?"* after the operator has already said *"go through all the files"* is the model defecting on its own authorization.

The pattern is documented (it's a flavor of what the AAAI 2026 dark-pattern paper calls the *"loop of death"*: model and user volleying without progress). No published Stop-hook tool addresses it.

## Differentiation

Existing anti-hesitation tooling (Marco Lancini's [stop-phrase-guard.sh](https://blog.marcolancini.it/2026/blog-my-claude-code-setup/)) catches *early-exit* phrases — "should I proceed?" appearing instead of action. `no-cliffhanger` catches the *symmetric* failure: dangling permission-loops appearing AFTER action that already happened, asking permission to keep going.

## Install

```bash
mkdir -p .claude/hooks
curl -fsSL https://raw.githubusercontent.com/waitdeadai/no-cliffhanger/main/no-cliffhanger.sh \
  -o .claude/hooks/no-cliffhanger.sh
chmod +x .claude/hooks/no-cliffhanger.sh
```

Merge `settings.example.json` entries into `.claude/settings.json`. Requires `jq`.

## Receipts

See [RECEIPTS.md](RECEIPTS.md) for five reproducible local fixture tests.

## Physics-backed engine

This standalone hook remains the simplest install path. For users who want the
benchmark-backed, rule-pack-hashed engine version, the same closeout mechanic is
also available in [AgentCloseoutBench](https://github.com/waitdeadai/agent-closeout-bench):

```bash
git clone https://github.com/waitdeadai/agent-closeout-bench
cd agent-closeout-bench
bash adapters/claude-code/install.sh /path/to/your/project no-cliffhanger
bash scripts/hook-smoke.sh
```

The physics-backed adapter maps `no-cliffhanger` to the `cliffhanger` category
engine and can be used for daily enforcement, fixtures, benchmark evaluation,
and opt-in content-free collaboration telemetry.

## Sister tools

Part of the [LLM Dark Patterns Hooks](https://github.com/waitdeadai/llm-dark-patterns) suite.

- [no-vibes](https://github.com/waitdeadai/no-vibes), [time-anchor](https://github.com/waitdeadai/time-anchor), [no-curfew](https://github.com/waitdeadai/no-curfew), [no-sycophancy](https://github.com/waitdeadai/no-sycophancy)
- [honest-eta](https://github.com/waitdeadai/honest-eta) — vibe time estimates and linear-scaling parallelism claims.
- [no-fake-recall](https://github.com/waitdeadai/no-fake-recall) — false-memory recall claims without quoted prior content.
- [no-fake-stats](https://github.com/waitdeadai/no-fake-stats) — fabricated percentages and amounts without source.
- [no-fake-cite](https://github.com/waitdeadai/no-fake-cite) — academic citation patterns without verifiable URL.
- [minmaxing](https://github.com/waitdeadai/minmaxing) — parent harness

## License

Apache-2.0.
