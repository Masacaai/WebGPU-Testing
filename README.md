# WebGPU Memory Model Litmus Tests

Classic memory-consistency litmus tests implemented in WGSL and run across three different
GPU memory architectures, to find out whether WebGPU implementations actually enforce the
ordering the WGSL specification describes.

Sole-authored for CS 554: Concurrent Computing Systems, Fall 2025.

## Why

The WGSL specification describes a memory model. It does not describe what shipping
implementations do with it. That gap is only observable empirically: relaxed-memory outcomes
are rare, non-deterministic, and invisible to reading either the spec or the source.

## What is tested

Four classic litmus tests, each as a standalone page:

| Test | File | What it asks |
|---|---|---|
| Message passing | `test.html` | Does a flag write become visible after the data write it guards? |
| Store buffering | `test-storebuffer.html` | Can two threads each read the other's pre-write value? |
| Coherence / relaxed | `relaxed.html` | Do independent read/write pairs observe a consistent order? |
| Non-atomics | `non-atomic.html`, `non-atomic-device-memory.html` | Are non-atomic reads usable for synchronization? |

`device-memory-test.html` repeats the work against device memory rather than workgroup
memory. `test-auto.html` runs the suite without manual stepping. `non-atomic-debug.html` is
instrumentation kept for reproducing the non-atomic result.

## Hardware

Three deliberately different points in the memory hierarchy:

- **Intel UHD** — integrated, shared with system memory
- **NVIDIA RTX 4090** — discrete, separate device memory
- **Apple M2 Ultra** — unified memory

## Results

**Relaxed memory outcomes occur in production WebGPU.** Behaviour forbidden under sequential
consistency was observed on NVIDIA device memory in **0.02% of 50,000 iterations** (five runs
of 10,000). A rate that low is the point: a smaller sample reads as noise, and a developer
who tests once will conclude the ordering is safe.

**Non-atomic reads are not usable for portable synchronization.** One vendor's compiler
optimizes them aggressively enough to break the pattern entirely, regardless of what the
hardware would have permitted.

**Memory-hierarchy level, not vendor, dominates the observed behaviour.** The integrated,
discrete and unified-memory machines differed from each other more than any two cards from
the same vendor would.

## Running them

Open any page in a WebGPU-capable browser and click to run. No build step and no
dependencies; each file is self-contained. Iteration counts are editable at the top of each
page's script. Results print in the page.

Rare outcomes need volume. A single run proves nothing either way.

## Notes

The GPU labels in these pages were corrected on 2026-09-30: three files carried stale `RTX
3090` strings left over from earlier testing on different hardware, while `test.html` itself
said `4090` in two places and `3090` in a third. The tests were run on the RTX 4090, which is
what the course report records.
