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

Dual-licensed MIT OR Apache-2.0, like launchbound itself.
