# Telecom Decision Model Benchmark

A controlled experiment comparing decision-making systems on telecom incident triage, evaluating accuracy, latency, throughput, scalability, robustness, and cost.

Two benchmark notebooks are provided:

1. **`jev_laya_otel_sonnet_gemini_benchmark.ipynb`** -- Compares Jev, Laya, OTel-2.0, Claude Sonnet, and Gemini
2. **`jev_kev_laya_otel_sonnet_gemini_benchmark.ipynb`** -- Extends the benchmark to include Kev (open-source Jev-style model)

---

## Systems Compared

| System | Type | Parameters | Deployment |
|--------|------|------------|------------|
| **Jev** (`jev-1.13`) | TypeSafe AI "System One" decision model; typed decisions with probabilities | Proprietary | HTTP API (OpenRouter / TypeSafe) |
| **Kev** (`jaredpalmer/kev-4b`) | Open-source Jev-style model: Qwen3.5 base + LoRA + pointer head | ~4B | Local server (`POST /v1/systemone`) |
| **Laya** | Small typed-decision encoder | ~421M | In-process (`pip install laya`) |
| **OTel-2.0-Decision** | AT&T's 4-bit Gemma-4 26B-A4B model | ~26B (4-bit) | OpenAI-compatible API with logprobs |
| **Claude Sonnet** | Frontier generative LLM | -- | OpenRouter chat completions |
| **Gemini** | Frontier generative LLM | -- | OpenRouter chat completions |

---

## What It Measures

- **Policy Quality** -- Exact-match accuracy across all 4 decision fields (severity, responsible team, escalation, action)
- **Per-Field Accuracy** -- Individual accuracy for severity classification, team assignment, escalation decision, and remediation action
- **Latency Profile** -- P50, P95, P99 response times at concurrency levels 1, 5, and 15
- **Throughput** -- Decisions per second under load
- **Scalability** -- How latency and accuracy change under concurrent requests
- **Robustness** -- Transport error rates, valid output rates, strict JSON compliance, retry counts
- **Cost Efficiency** -- Operational cost per 1,000 requests
- **Confidence Calibration** -- Severity accuracy when model confidence is >= 0.9 vs < 0.9

---

## Dataset

The benchmark uses 53 synthetically generated telecom incidents stored in `data/evaluated_incidents_dataset.json`. Each incident includes:

- **Element type**: 5G base stations, backhaul links, core routers, etc.
- **Telemetry**: packet loss %, latency (ms), duration (min), users affected
- **Context**: region, status (DEGRADED/DOWN), cause, operator notes
- **Ground truth**: rule-based labels for severity, responsible team, escalation, and action

Additionally, 3 complex narrative edge cases test unstructured reasoning vs. structured parsing.

---

## Performance Results

### Jev vs. Kev vs. OTel (from `jev_kev_laya_otel_sonnet_gemini_benchmark.ipynb`)

#### Policy Quality (Exact Match, All 4 Fields)

| System | Severity | Responsible | Escalate | Action | Exact Match |
|--------|----------|-------------|----------|--------|-------------|
| **Jev** | **1.00** | **1.00** | **1.00** | **0.96** | **0.96** |
| Kev | 0.60 | 0.83 | 0.89 | 0.64 | 0.38 |
| OTel | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |

Jev achieves near-perfect policy compliance across all fields. Kev, while using the same typed-decision interface, shows significant accuracy degradation -- particularly on severity classification (60%) and action selection (64%). OTel failed entirely during this run due to HTTP 502 errors from the shared AT&T test endpoint (424 retries, 100% transport error rate).

#### Response Latency (P50 / P95)

| System | Concurrency 1 | Concurrency 5 | Concurrency 15 |
|--------|---------------|----------------|-----------------|
| **Jev** | 202 / 243 ms | 214 / 264 ms | 222 / 309 ms |
| Kev | 787 / 800 ms | 3,534 / 3,737 ms | -- |
| OTel | N/A (errors) | N/A (errors) | N/A (errors) |

Jev maintains sub-300ms P95 latency even at concurrency 15. Kev's latency explodes under load, jumping from ~800ms to 3.5 seconds at concurrency 5.

#### Throughput Scaling

| System | Concurrency 1 | Concurrency 5 |
|--------|---------------|----------------|
| **Jev** | 4.79 dec/s | 20.39 dec/s |
| Kev | ~1.3 dec/s | ~1.4 dec/s |

Jev scales linearly with concurrency. Kev's throughput is bottlenecked by its local inference.

#### Cost Efficiency

| System | Cost per 1,000 Requests | Exact Match |
|--------|------------------------|-------------|
| **Jev** | $0.068 | 0.96 |
| Kev | $0.431 | 0.38 |
| OTel | $0.00 (failed) | 0.00 |

Jev delivers 2.5x higher accuracy at 6x lower cost than Kev.

### Jev vs. Laya vs. OTel (from `jev_laya_otel_sonnet_gemini_benchmark.ipynb`)

#### Policy Quality at Concurrency 1

| System | Severity | Responsible | Escalate | Action | Exact Match |
|--------|----------|-------------|----------|--------|-------------|
| **Jev** | **1.00** | **1.00** | **1.00** | **0.94** | **0.94** |
| Laya | 0.21 | 0.34 | 0.64 | 0.42 | 0.02 |
| OTel | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |

#### Latency at Concurrency 1

| System | P50 | P95 | P99 |
|--------|-----|-----|-----|
| **Jev** | 202 ms | 243 ms | 274 ms |
| Laya | 106 ms | 114 ms | 116 ms |

Laya is faster (runs locally, compute-only latency) but its accuracy is too low for production use. Jev's ~200ms latency includes network round-trip time.

---

## Robustness Analysis

### Structured Output Compliance

| System | Strict JSON Rate | Valid Output Rate |
|--------|-----------------|-------------------|
| **Jev** | 100% | 100% |
| **Kev** | 100% | 100% |
| Laya | 100% | 100% |
| OTel | 0% | 0% |

All systems except OTel produce valid, parseable structured outputs.

### Latency Stability (P95/P50 Jitter Ratio)

| System | Jitter Ratio |
|--------|-------------|
| Kev | 1.01x |
| **Jev** | 1.06x |

Both Jev and Kev exhibit stable latency profiles with minimal jitter under load.

---

## Benchmark Graphs

The following visualizations are generated by the notebooks and saved to `graph and performance/`:

| Graph | Description |
|-------|-------------|
| `benchmark.png` | Overview: latency, throughput, and accuracy comparison |
| `publication_quality_benchmark.png` | Response latency profiles (P50 vs P95) and throughput scaling across concurrency levels |
| `robustness_dimensions_benchmark.png` | Cost vs. accuracy frontier, structured output compliance, and latency jitter |
| `token_cost_scaling_benchmark.png` | Input/output token consumption and operational cost scaling with concurrency |

### Key Visual Insights

**Speed vs. Accuracy Frontier**: Jev occupies the ideal top-left position (fast + accurate). Kev sits at the bottom-right (slower + less accurate). This shows that Jev's cloud-based inference is both faster and more accurate than Kev's local inference.

**Cost vs. Accuracy Frontier**: Jev achieves ~96% exact match at $0.068/1k requests. Kev reaches ~38% at $0.431/1k. The cost-accuracy tradeoff strongly favors Jev.

**Per-Field Accuracy Heatmap**: Jev scores 1.00 on severity, responsible, and escalate fields and 0.96 on action. Kev's weakest field is severity (0.60), followed by action (0.64). OTel scores 0.00 across all fields.

---

## Quick Start

### 1. Install Dependencies

```bash
pip install laya httpx python-dotenv matplotlib pandas seaborn
```

### 2. Configure API Keys

Set environment variables or create a `.env` file:

- `OPENROUTER_API_KEY` -- Required for Jev (via OpenRouter), Sonnet, and Gemini
- `TYPESAFE_API_KEY` -- Required if using Jev via TypeSafe directly
- Laya needs no key (runs locally)
- Kev needs no key (runs on local server)
- OTel needs no key on AT&T's test endpoint

### 3. Run the Notebooks

Open either notebook in `src/` and execute cells in order:

1. **Sections 1--2**: Install dependencies and configure API keys
2. **Section 3**: Define the triage policy and incident simulator
3. **Section 4**: Set up backend implementations
4. **Section 5**: Benchmark engine (runner, metrics, tables, plots)
5. **Section 6**: Smoke test (1 call per backend)
6. **Section 7**: Full benchmark (adjust `n` and `concurrency` as needed)
7. **Section 8**: Analyze results with visualizations

### 4. Output

Results are saved to `graph and performance/`:
- `summary.csv` / `summary.json` -- Aggregated metrics per backend and concurrency level
- `details.csv` -- Per-incident predictions and ground truth
- `*.png` -- Performance visualization plots

---

## Notebook Structure

| Section | Description |
|---------|-------------|
| 1 | Installation |
| 2 | API keys and configuration |
| 3 | Simulator: shared triage policy, incident generator, ground truth |
| 4 | Backend implementations (Jev, Kev, Laya, OTel, Sonnet, Gemini) |
| 5 | Benchmark engine: runner, metrics collector, tables, plot generators |
| 6 | Smoke test |
| 7 | Full benchmark (standard incidents + complex narrative edge cases) |
| 8 | Analysis and visualization |

---

## Key Notes

- **Fairness**: Remote APIs (Jev, Sonnet, Gemini) include network latency. Laya and Kev run locally, so their latency is compute-only.
- **OTel**: Requires 4 requests per incident (one per question), run in parallel. In the benchmarked run, OTel's shared endpoint returned HTTP 502 errors consistently.
- **Ground Truth**: Rule-based, measuring policy-following accuracy rather than telecom domain expertise.
- **Complex Incidents**: Section 7b includes 3 narrative edge cases to test unstructured reasoning vs. structured parsing. Jev handled all 3 correctly; Kev made incorrect predictions on severity and action fields.

## License

This is a benchmark notebook for research and evaluation purposes.
