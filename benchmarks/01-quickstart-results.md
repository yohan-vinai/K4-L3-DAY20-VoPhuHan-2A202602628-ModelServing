# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 2118 | 96 / 268 | 16.9 / 22.2 | 1123 / 1521 / 1521 | 59.2 |
| UD-Q2_K_XL | 0.39 | 2090 | 98 / 141 | 17.7 / 22.3 | 1204 / 1545 / 1545 | 56.6 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.05x slower** than `Q4_K_M` here, despite being 0.11 GB smaller. On this M1 run, Metal offload was active; these measurements do not isolate whether quantization-kernel overhead or run-to-run variation explains the difference, so I do not attribute it to a profiled bottleneck.

## Your observation

Q2 saved 0.11 GB, but decoded at 56.6 tok/s versus Q4 at 59.2 tok/s (about 4.4% slower); TTFT P50 was nearly the same (98 ms vs 96 ms). The smaller file was not a speed win in this run. In a separate same-question comparison, Q4 gave a partly relevant but incorrect explanation, while Q2 went off-topic and invented a training/accuracy trade-off. Neither answer was reliable without checking the source, so the size saving did not justify choosing Q2 for this use.
