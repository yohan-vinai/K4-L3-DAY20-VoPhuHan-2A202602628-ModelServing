# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Darwin-arm64` · llama.cpp `b10488`
CPU: **8 physical · 8 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 55.2 | 100% |
| 4 | 46.7 | 85% |
| 8 | 47.2 | 86% |
| 16 | 44.9 | 81% |

**Best**: `-t 1` at 55.2 tok/s
**Slowest tested**: `-t 16` at 44.9 tok/s (1.23x spread)
**Against the physical-core default** (`-t 8`, 47.2 tok/s): 1.17x

Use this in your run:

```bash
LAB_N_THREADS=1 make bench
```

## Your explanation

The measured peak is at 1 thread (55.2 tok/s); throughput drops to 47.2 at 8 threads and 44.9 at 16. That is the opposite of a knee near the 8 physical cores. With `ngl=99`, llama.cpp offloads the model to Apple Metal, so adding CPU worker threads does not add GPU compute or memory bandwidth; it may instead add host scheduling/coordination overhead for this small model. This is a plausible explanation, not a profile of the runtime, and the result is specific to this M1/Metal setup.
