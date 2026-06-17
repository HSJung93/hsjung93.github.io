---
title: "Hunting the zombie GIL: how Cython profiling hooks in the release wheel slowed causalml's parallel Causal Forest"
date: 2026-06-17 08:00:00 +0900
categories: [개발, 성능]
tags: [Cython, GIL, Performance, Open Source, Causal Inference, causalml]
---

I was trying out a few statistics packages to do uplift modeling for measuring marketing performance. One of them was Uber's [causalml](https://github.com/uber/causalml) (0.16.0), and while fitting a model with its `CausalRandomForestRegressor` I noticed something off. The fit was slow, so I raised `n_jobs` to parallelize it, and it got *slower* instead. Setting `n_jobs=4` to train with four workers was about 6x slower than the default (`n_jobs=1`). The cause turned out to be a `linetrace` directive hardcoded into the tree-building Cython sources, which compiled profiling hooks all the way into the released wheel. This post walks through how I traced that cause back from the symptom, and how I verified and fixed it. For what it's worth, the issue I filed and the fix are now part of causalml's source.

- Pull request: [uber/causalml#914](https://github.com/uber/causalml/pull/914)
- Issue: [uber/causalml#913](https://github.com/uber/causalml/issues/913)

## The symptom

While comparing several meta-learners (T/S-Learner) with the Causal Forest, only the Causal Forest was unusually slow. Tuning `n_jobs` to parallelize it gave a result that ran against intuition: the more cores I added, the longer the fit took.

| setting                  | fit time |
| ------------------------ | -------: |
| default (`n_jobs` unset) |    29.0s |
| `n_jobs=1`               |    29.5s |
| `n_jobs=4`               |   192.6s |

_PyPI wheel 0.16.0, 50 trees, 12k rows._

There was no error and no warning, just a longer fit. The bug probably survived because you can't spot anything wrong unless you compare across different parallel core counts.

## Root cause

When adding cores makes things slower, it's a sign that the workers are serializing somewhere. I had to find the point where the parallel workers converge on a single lock.

causalml's tree building is written in Cython for speed. Five of its `.pyx` sources (and the matching `.pxd` headers) had this directive hardcoded:

```cython
# cython: linetrace=True
```

`linetrace` is a tool for measuring test coverage. The problem is that this setting doesn't stay in the development build; it gets compiled into the released wheel as well. When the directive is on, the C code generated for CPython is built with `CYTHON_PROFILE 1`, which inserts profiling hooks (`__Pyx_TraceCall` / `__Pyx_TraceSetupAndCall`, etc.) into every function. That includes the hot-loop `nogil` tree-building criterion functions.

Because of these hooks, worker threads that should run with the GIL released re-acquire the GIL inside the hooks and serialize. The more threads there are, the worse the contention, which is why `n_jobs=4` was slower than `n_jobs=1` (the default).

You can confirm directly that the hooks made it into the released binary by inspecting its symbols:

```console
$ nm .../causalml/inference/tree/causal/_criterion.cpython-312-darwin.so \
    | grep -E "TraceSetupAndCall|call_return_trace"
... __Pyx_TraceSetupAndCall ...
... __Pyx_call_return_trace_func ...
```

## Verification and reproduction

To verify, I compared two builds that differed only in the `linetrace=True` directive, on the same machine, with the same Cython and the same `setup.py`. The results:

| build                          | profiling hooks in `.so` | n_jobs=1 | n_jobs=4 |
| ------------------------------ | :----------------------: | -------: | -------: |
| `linetrace=True` present (orig.) |     yes (2 symbols)      |    29.4s |   190.9s |
| `linetrace=True` removed       |           none           |     4.1s |     1.7s |

Removing the `linetrace=True` directive cleared the hooks from the binary. The single-threaded fit got about 7x faster, and `n_jobs` went back to acting as a speedup (`n_jobs=4` moved from the slowest row to the fastest). Since only the compilation was changed, the statistical model's predictions did not change: the downstream T-Learner's Qini score matched to six decimal places, and `tests/test_causal_trees.py` still passed.

I also packaged the check as a Docker setup so it can be reproduced easily in an isolated environment. The Dockerfile and the reproduction command are in the issue linked at the top.

## The fix

Instead of hardcoding the directive in the sources, I moved it behind an environment variable so it's opt-in. In `setup.py`, `linetrace` is enabled only when `CYTHON_TRACE` is set:

```python
import os
linetrace = os.environ.get("CYTHON_TRACE", "0") not in ("0", "", "false", "False")
define_macros = [("CYTHON_TRACE", "1"), ("CYTHON_TRACE_NOGIL", "1")] if linetrace else []
# Extension(..., define_macros=define_macros)
# cythonize(..., compiler_directives={"linetrace": linetrace})
```

I then removed `# cython: linetrace=True` from the five `.pyx` files and their `.pxd` headers. Coverage builds can still turn instrumentation on with `CYTHON_TRACE=1 python setup.py build_ext`, while every other release build is no longer instrumented. The profiling tool isn't gone; it's just separated out to build time so it doesn't ride along into every function call.

## Wrapping up

What made this bug tricky is that it disguises itself, with no error or warning, as "the model is just heavy." The clue was the counterintuitive result: raising `n_jobs` made the fit take longer, not shorter. I went in to use a statistics library and ended up down in the generated C code and the build configuration. It was a fun excuse to actually dig into the Python GIL and wheel compilation/builds, things I'd only heard about until now.

Whether a model gives the right answer matters, and so does whether it gives that answer in time. A slow tool doesn't get used enough, and a tool that isn't used enough doesn't get verified enough either.

Back in grad school, running models for statistics work, it never crossed my mind to read the code to make it faster. The debugging instinct came from five years as a backend engineer. I've since switched teams and I'm now on a causal inference project to measure marketing performance. It's been a while since I last ran statistical models, and that's part of what made this a welcome bug: somewhere in between, I could feel that I'd grown a bit broader and a bit deeper.
