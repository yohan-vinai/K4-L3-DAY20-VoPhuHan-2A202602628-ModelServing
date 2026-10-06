# Reflection — Day 20 Lab (Personal Report)

**Name:** Võ Phú Hãn
**Student ID:** 2A202602628
**Cohort:** A20-K4
**Submission date:** 2026-10-06

## 1. Hardware & runtime

- **OS:** macOS (Darwin 25.6.0, arm64)
- **CPU:** Apple M1; **cores:** 8 physical / 8 logical; **extensions:** NEON
- **RAM:** 16.0 GB; **accelerator:** Apple Metal
- **llama.cpp:** `llama-b10488-bin-macos-arm64.tar.gz` (prebuilt)
- **Model:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantizations:** Q4_K_M + UD-Q2_K_XL
- **Run on:** personal laptop

Setup ran on the M1. I chose the smaller model to reduce download and run time. Setup fetched the macOS arm64 runtime and both quantizations; no CUDA, compiler, or cloud fallback was needed.

## 2. Measurement

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 2094 | 122 / 321 | 24.0 / 30.1 | 1558 / 2044 / 2044 | 41.6 |
| UD-Q2_K_XL | 0.39 | 3117 | 123 / 150 | 20.6 / 23.5 | 1444 / 1593 / 1593 | 48.7 |

Q2 used 0.11 GB less and decoded 1.17× faster; TTFT P50 was almost unchanged. On the same prefill/decode question, Q4 gave a partly relevant but incorrect explanation, while Q2 went off-topic and invented a training/accuracy trade-off. For this factual prompt, the speed/size gain did not justify the quality loss; neither answer was reliable without checking the source.

## 3. Serving under load

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Effective concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.83 | 11000 | 16000 | 17000 | 8.6 | 0.0% |
| 50 | 0.64 | 27000 | 57000 | 57000 | 18.3 | 0.0% |

- Offered users increased 5×; delivered throughput was **0.78×** (a 22% decrease).
- P95 latency increased **3.56×**.
- Effective concurrency at 50 users: **18.3** versus `--parallel=4` slots.
- Peak `n_busy_slots_per_decode`: **3.87/4** slots; **46** requests were deferred.

Using a 20-second P95 target for this reading, the 10-user run met it (16 s), while the 50-user run did not (57 s). The high-load run shows a growing queue: Little's Law effective concurrency includes queued requests, while the 3.87 busy-slot gauge measures decode-slot occupancy. I would first cap long prompts/outputs or admission rate to reduce slot holding time; adding slots alone does not add M1 memory bandwidth.

## 4. Integration

| Day | Piece | Real or stub? |
|---|---|---|
| N16 Cloud/IaC | Not connected to the pipeline | Stub |
| N17 Data pipeline | Not connected to the pipeline | Stub |
| N18 Lakehouse | Not connected to the pipeline | Stub |
| N19 Vector + features | Toy documents + keyword overlap; no vector index or embedding service | Stub |
| N20 Serving | llama-server | Real |

Mean latency over three queries: embed **0.0 ms**, retrieve **0.1 ms**, LLM **3475.5 ms**, total **3475.6 ms**. LLM generation is nearly 100% of the measured total. To halve latency in this setup, I would reduce prompt/context or output tokens first; retrieval optimization cannot yield a 2× speedup when it takes 0.1 ms.

## 5. The single change that mattered most

**Change:** reduce llama-bench thread count from 8 to 1.

```text
before: 47.2 tok/s (`-t 8`, tg128)
after:  55.2 tok/s (`-t 1`, tg128)
speedup: 1.17×
```

The measured peak was at one thread, not near the eight physical cores. With `ngl=99`, llama.cpp offloads the model to Apple Metal; adding CPU threads does not add GPU compute or memory bandwidth, and may add host scheduling/coordination overhead for this small model. This is a plausible explanation consistent with the measurement, not a profiler finding, and applies to this M1/Metal setup.

## 6. Bonus

No bonus track completed.

## 7. What surprised me most

Reducing threads from eight to one improved measured decode throughput by 17%. Q2 was faster, but its answer to the comparison question was less accurate than Q4's.

## 8. Self-check before push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] Benchmark reports generated and observations filled
- [ ] Five screenshots in `submission/screenshots/`
- [ ] `make verify` → exit 0
- [x] Repository has the required name: `K4-L3-DAY20-VoPhuHan-2A202602628-ModelServing`
- [x] GitHub repository is public
- [ ] Final commit pushed
- [ ] URL pasted into VinUni LMS before the deadline
- [x] Model weights, runtime, and `.env` are not included

## 9. AI usage disclosure

OpenAI Codex was used to read the guide, run setup/benchmarks/load tests/pipeline on this laptop, compare the two model responses, and help draft the explanations from measured results. The measurements came from this M1; no metrics or screenshots were fabricated.
