# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Darwin-arm64` · llama.cpp `b10488`
Settings: `threads=8` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 2094 | 122 / 321 | 24.0 / 30.1 | 1558 / 2044 / 2044 | 41.6 |
| UD-Q2_K_XL | 0.39 | 3117 | 123 / 150 | 20.6 / 23.5 | 1444 / 1593 / 1593 | 48.7 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.17x faster** than `Q4_K_M` here, for 0.11 GB less on disk.

## Your observation

`UD-Q2_K_XL` reduced size by 0.11 GB and improved decode from 41.6 to 48.7 tok/s (1.17x), while TTFT P50 stayed about the same. On the same prefill/decode question, Q4 gave a partly relevant but incorrect explanation; Q2 went off-topic and invented a training trade-off. For this factual prompt, the speed/size gain did not justify the quality loss; neither answer was reliable without checking the source.
