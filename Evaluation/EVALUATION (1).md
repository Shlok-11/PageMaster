# PageMaster — Evaluation

## How the evaluation was performed

Python scripts (created with AI assistance) generated process files and command inputs, launched the compiled C++ memory simulator with `subprocess`, inspected its output files for expected behavior, and timed execution using `time.perf_counter_ns()`. The engine was compiled with `g++ -O2 -std=c++17`.

Two performance workloads were measured separately:

- **Instruction execution:** 25 trials, each executing 4,000 simple `load` instructions in a process file (100,000 instructions total).
- **LRU stress:** 25 trials, each issuing 240 `run(pid)` commands against three 2-KB processes with only 4 KB of main memory and 8 KB of simulated virtual memory (6,000 process-run commands total). This repeatedly forces whole-process swapping.

Throughput per trial was calculated as **operations completed / elapsed seconds**. The figures below are *averages of the 25 per-trial throughputs*, not necessarily total operations divided by total elapsed time.

## Recorded results

| Test | Result |
|---|---:|
| Functional scenarios (allocation, swapping, page tables, errors, data persistence, cleanup) | 12/12 passed |
| Instruction benchmark | 25/25 trials completed |
| Mean instruction throughput | 821,830 instructions/sec |
| LRU stress benchmark | 25/25 trials without reported swapping errors |
| Mean LRU workload throughput | 19,953 **process-run commands/sec** |
| LRU throughput range | 6,844–23,281 process-run commands/sec |
| Successful swap-ins observed across LRU trials | 5,950 |
| Mean elapsed time per LRU trial | 13.12 ms |

**Metric clarification:** A `run(pid)` command may cause a swap-out, swap-in, and execution of several internal instructions. Thus, *19,953 commands/sec is not a count of individual swap operations or process-file instructions per second*. The previously cited **7,800 ops/sec** figure was **not independently reproduced** as an equivalent benchmark.

## Scope and limitations

These are results from a **locally reconstructed implementation based on the inspected PageMaster GitHub source**, **not an execution of the exact, unmodified GitHub checkout**. They should not be presented as independently verified performance measurements of that original repository. Passing functional scenarios does not prove full memory safety or exhaustive LRU eviction-order correctness; negative-address and other boundary cases require additional testing. The current code uses **process-level LRU swapping**, not per-page LRU replacement.

For reproducibility, preserve the benchmark scripts, raw trial CSV files, output logs, compiler settings, and the exact source revision used in any future rerun.
