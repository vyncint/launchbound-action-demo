# launchbound action demo

A worked example of running
[launchbound](https://github.com/vyncint/launchbound)'s convergence safety
gate in CI with `uses: vyncint/launchbound/action@v1` — **no GPU anywhere**.

[![convergence-gate](https://img.shields.io/github/actions/workflow/status/vyncint/launchbound-action-demo/gate.yml?label=convergence-gate)](https://github.com/vyncint/launchbound-action-demo/actions/workflows/gate.yml)
[![GPU](https://img.shields.io/badge/GPU-not%20required-brightgreen)](https://github.com/vyncint/launchbound)

Two cuda-oxide kernels under [`kernels/`](kernels/), exercising the gate in
both directions on every push:

| kernel | space | expectation | job |
|---|---|---|---|
| [`reduce-stable-demo`](kernels/reduce-stable-demo/) | 4 configs | all admitted — `fail-on: refused` keeps it that way | **clean kernel stays clean** |
| [`reduce-flip-demo`](kernels/reduce-flip-demo/) | 4 configs | the two `block_x=64` configs are refused (RC001: a `warp_id()` guard around a block-wide barrier is safe only at one warp) | **known-flip kernel is caught** — the job asserts the refusals |

The flip kernel is the case the tool exists for: same source, safe at
`block_x=32`, undefined at 64 — and on real hardware the unsafe configs can
run *faster*, silently. An ordinary autotuner hands you one; the gate
refuses it with the rule ID and source line.

See [.github/workflows/gate.yml](.github/workflows/gate.yml) — the whole
integration is one `uses:` step plus a sibling checkout of the pinned
cuda-oxide commit so the kernels' `cuda-device` path dependency resolves.

## Testing the tool, not just demonstrating it

`gate.yml` uses `@v1` because that is what this README tells you to copy, and
a demo should exercise what users actually get. That leaves exactly one thing
it cannot see, and it is the thing that happened: `v1` is a floating tag
somebody has to move, and when a release forgets to, every consumer silently
keeps the old action while the demo goes on passing. `v1` sat on 1.0.2 through
two releases, so these kernels were being gated by reconverge 0.1.11 while
1.2.0's whole subject was pinning 0.3.0.

[`latest.yml`](.github/workflows/latest.yml) asks the two questions `gate.yml`
cannot:

1. **does `@v1` resolve to the same action as the newest release tag?** — a
   direct diff of `action/action.yml` at both refs;
2. **do the gate's invariants still hold against the newest published
   `launchbound-cli`?** — the same two kernels, with the crate version
   resolved from crates.io at run time rather than from the action's default.

Both crates come from crates.io; nothing here is ever built from a checkout of
the tool repository. It runs daily, on dispatch, and on `repository_dispatch`
so a launchbound release can trigger it, and opens one issue per failing
version rather than one per run.

A changed verdict here is worth reading carefully: these two kernels have a
convergence property known *by construction*, so the gate reaching a different
answer means the analyzer changed, not that the demo drifted.

Dual-licensed MIT OR Apache-2.0, like launchbound itself.
