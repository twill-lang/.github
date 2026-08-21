<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/twill-lang/twill/main/assets/twill-wordmark-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/twill-lang/twill/main/assets/twill-wordmark.svg">
    <img alt="twill" src="https://raw.githubusercontent.com/twill-lang/twill/main/assets/twill-wordmark.svg" width="340">
  </picture>
</p>

<p align="center">
  <b>A small language where tensors are the primitive, <code>grad</code> is built in,<br>
  and a shape mistake is an error you see before the program runs.</b>
</p>

<p align="center">
  <a href="https://github.com/twill-lang/twill/releases"><img alt="release" src="https://img.shields.io/github/v/release/twill-lang/twill?sort=semver&style=flat-square&labelColor=12332C&color=4FB79B"></a>
  <img alt="status: early prototype" src="https://img.shields.io/badge/status-early%20prototype-D2F0E4?style=flat-square&labelColor=12332C">
  <img alt="dependencies: none" src="https://img.shields.io/badge/dependencies-none-A8DCCB?style=flat-square&labelColor=12332C">
  <a href="https://github.com/twill-lang/twill/blob/main/LICENSE"><img alt="MIT" src="https://img.shields.io/badge/license-MIT-7FE3C4?style=flat-square&labelColor=12332C"></a>
</p>

---

Most machine-learning code is a language plus a numeric framework bolted on top.
twill goes the other way. Tensors are the built-in data type, differentiation is
a language operation rather than a library call, and a static checker reads your
shapes before anything executes.

This prices a European call by Monte Carlo and gets its delta and vega by
differentiating the pricer, with no bumping and no second library:

```rust
seed(42)
let Z = randn(200000)                              # fixed shocks: the price is smooth in its inputs

fn call_price(S0, K, r, sigma, T) {
  let drift = (r - 0.5 * sigma * sigma) * T
  let ST = S0 * exp(drift + sigma * sqrt(T) * Z)   # simulated terminal prices
  exp(-r * T) * mean(relu(ST - K))                 # discounted expected payoff
}

let price = call_price(100.0, 100.0, 0.05, 0.2, 1.0)
let delta = grad(fn(s) = call_price(s, 100.0, 0.05, 0.2, 1.0))(100.0)
let vega  = grad(fn(v) = call_price(100.0, 100.0, 0.05, v, 1.0))(0.2)
```

```
$ twill examples/montecarlo_option.tw
European call, S0=100 K=100 r=5% vol=20% T=1y, MC paths: 200000
  price = 10.442696  (Black-Scholes 10.4506)
  delta = 0.636269  (Black-Scholes 0.6368)
  vega  = 37.488476   (Black-Scholes 37.524)
```

No tape object, no `requires_grad`, no `.backward()`.

## Where it actually stands

twill is an early prototype. The current release is v1.7.1. The reference
implementation is a single Go binary with no dependencies, 26,638 lines
excluding tests. MIT licensed.

What is not done yet:

- It is interpreted. Tensor ops loop in Go, and there is no vectorized or GPU
  backend. There is a tracing compiler behind `TWILL_TRACE=1`; it is correct and
  on most programs slower end to end, so it ships off. twill's
  [docs/CODEGEN.md](https://github.com/twill-lang/twill/blob/main/docs/CODEGEN.md)
  has the measurements and the reason.
- Autodiff is reverse-mode and first-order. `grad(grad(f))` is refused rather
  than silently answered with zero; `hessian` gives second derivatives via
  forward-mode jets.
- The shape checker is best-effort, not a full type system. It flags a mismatch
  only when it is certain, and stays quiet otherwise.
- As of v1.4.0 the twill compiler written in twill runs on the Go bootstrap and
  reproduces the reference across every stage. It runs on the bootstrap rather
  than as its own Go-free binary; bootstrapping to a standalone twill-built
  compiler is the next step. `twill check` matched the Go command byte-for-byte
  on every corpus file and `twill fmt` on every one it formats, bar a by-design
  blank-line divergence. Those runs were counted at v1.5.0, at 443 and 89 files;
  the corpus has grown since and the counts are a snapshot, not a running total.

## The repositories

**The language**

| Repo | What it is |
| --- | --- |
| [twill](https://github.com/twill-lang/twill) | The language and the reference implementation: run, check, fmt, test, REPL, the standard library, and the self-hosted implementation under `src/`. |

**The ecosystem.** Nine libraries written in twill, against `mode systems`, the
systems subset described in twill's `docs/self-hosting.md`. Each was written
ahead of the language that runs it, and for a long time this paragraph said none
of them ran. **They run now.** `twill test tests` passes in all nine against
v1.7.1: 61 suites, 0 failures, counted per repo below. Each still carries a
`docs/needs.md` recording what it asked the language for and which of those
arrived.

Two caveats worth having before you clone. heddle's suite takes about 16 minutes
because two of its files are real NUTS runs, and warp's example needs an MNIST
download it will tell you how to fetch.

| Repo | What it is | Suites |
| --- | --- | --- |
| [spool](https://github.com/twill-lang/spool) | The package manager. | 6 |
| [loom](https://github.com/twill-lang/loom) | The training framework: epochs, callbacks, checkpointing, metrics, over a step function you pass in. | 8 |
| [warp](https://github.com/twill-lang/warp) | Data pipelines and dataset loaders. | 5 |
| [skein](https://github.com/twill-lang/skein) | Text and sequence handling: tokenisers with an offset map that points at the source, not the normalised string. | 11 |
| [heddle](https://github.com/twill-lang/heddle) | Probabilistic programming and Bayesian inference. NUTS, HMC, ADVI, and the diagnostics that catch a sampler lying. | 8 |
| [selvedge](https://github.com/twill-lang/selvedge) | Model serialisation and the model registry. | 6 |
| [shuttle](https://github.com/twill-lang/shuttle) | Inference and serving. No network server: twill has no sockets, and none is planned. | 6 |
| [bobbin](https://github.com/twill-lang/bobbin) | Benchmarking and profiling. Median and interquartile range, never mean and sigma. | 5 |
| [weft](https://github.com/twill-lang/weft) | Plots, in the terminal and out of it. Terminal charts and SVG. | 6 |

## Start here

- [twill-lang.github.io](https://twill-lang.github.io)
- [Documentation](https://twill-lang.github.io/docs/)
- [Install](https://github.com/twill-lang/twill#install), or download a binary from [releases](https://github.com/twill-lang/twill/releases)
- [docs/BENCHMARKS.md](https://github.com/twill-lang/twill/blob/main/docs/BENCHMARKS.md) for how fast it is and where the time goes
- [docs/CORRECTNESS.md](https://github.com/twill-lang/twill/blob/main/docs/CORRECTNESS.md) for the evidence behind `grad` and the checker

Bug reports, small fixes and design discussion are all welcome.
