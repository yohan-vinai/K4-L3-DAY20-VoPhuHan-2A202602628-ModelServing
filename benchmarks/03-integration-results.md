# 03 - Integrate: RAG pipeline run

Host `Darwin-arm64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.0 | 4108.4 | 4108.5 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 3760.0 | 3760.2 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 2558.0 | 2558.1 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **3475.5** · total **3475.6**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput is more useful than raw throughput** because it focuses on **SLOs (Service Level Objectives)** rather than ignoring them.

Here is the breakdown of why this distinction matters based on the text:

*   **Goodput** counts requests per second that **met the TTFT (Throughput Target) and TPOT (Throughput Target)** targets.
*   **Raw throughput** ignores SLOs ent

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** caused by storing key-value pairs (KV cache) in non-contiguous pages.

Specifically, it addresses the issue where GPU memory is often fragmented into small, non-contiguous blocks. By storing the KV cache in such non-contiguous pages, PagedAttention allows the engine to skip prefilling (which is computationally expensive)

**When does splitting prefill and decode help?**

> Based on the context provided, splitting prefill and decode helps when **prefill is compute-bound and decode is memory-bandwidth-bound**.

The context explicitly states that "Disaggregated serving splits prefill and decode onto separate pools because prefill is compute-bound and decode is memory-bandwidth-bound." This indicates that the system uses separate pools for these two operations to optimi


## Which N16-N19 pieces are real

N16 Cloud/IaC, N17 data pipeline, N18 lakehouse, and N19 vector/features are all stubs in this run. Retrieval uses the lab's toy documents and keyword overlap; no N19 vector index or embedding service is connected. The llama-server call is real. LLM generation accounts for 3475.5 of 3475.6 ms, so it is the clear bottleneck; retrieval optimization cannot halve this measured pipeline, while reducing prompt/context or output tokens targets the dominant stage.
