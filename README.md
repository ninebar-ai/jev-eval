# Telecom Decision Model Benchmark

A controlled experiment comparing five decision-making systems on telecom incident triage: **Jev**, **Laya**, **OTel-2.0**, **Claude Sonnet**, and **Gemini**.

## What It Measures

- **Quality**: Accuracy on severity classification, team assignment, escalation decisions, and remediation actions
- **Latency**: Response time (p50, p95, p99) across different concurrency levels
- **Throughput**: Decisions per second
- **Scalability**: How performance changes with concurrent requests
- **Reliability**: Transport error rates, valid output rates, retry behavior
- **Cost**: Operational cost per 1,000 requests

## Systems Compared

| System | Type | Access |
|--------|------|--------|
| **Jev** (`jev-1.13`) | TypeSafe AI "System One" decision model | HTTP API (OpenRouter or TypeSafe) |
| **Laya** | ~421M typed-decision encoder | Runs locally (`pip install laya`) |
| **OTel-2.0-Decision** | AT&T's 4-bit Gemma-4 26B-A4B model | OpenAI-compatible API with logprobs |
| **Claude Sonnet** | Frontier generative LLM | OpenRouter chat completions |
| **Gemini** | Frontier generative LLM | OpenRouter chat completions |

## Quick Start

### 1. Install Dependencies

```bash
pip install laya httpx python-dotenv matplotlib pandas seaborn
```

### 2. Configure API Keys

Set environment variables or create a `.env` file:

- `OPENROUTER_API_KEY` - Required for Jev (if using OpenRouter), Sonnet, and Gemini
- `TYPESAFE_API_KEY` - Required if using Jev via TypeSafe directly
- Laya needs no key (runs locally)
- OTel needs no key on AT&T's test endpoint

### 3. Run the Notebook

Open `src/jev_laya_otel_sonnet_gemini_benchmark.ipynb` and execute cells in order:

1. **Section 1-2**: Install and configure
2. **Section 3**: Define policy and incident simulator
3. **Section 4**: Set up backend implementations
4. **Section 6**: Smoke test (1 call per backend)
5. **Section 7**: Full benchmark (adjust `n` and `concurrency` as needed)
6. **Section 8**: Analyze results with visualizations

### 4. Results

Results are saved to `results/` with:
- `summary.csv` - Aggregated metrics per backend/concurrency
- `details.csv` - Per-incident predictions
- `benchmark.png` - Performance plots
- Publication-quality analysis plots

## Notebook Structure

- **Section 1**: Installation
- **Section 2**: API keys & configuration
- **Section 3**: Simulator (shared policy, incident generator, ground truth)
- **Section 4**: Backend implementations (Jev, Laya, OTel, Sonnet, Gemini)
- **Section 5**: Benchmark engine (runner, metrics, tables, plots)
- **Section 6**: Smoke test
- **Section 7**: Full benchmark (standard + complex edge cases)
- **Section 8**: Analysis and visualization

## Key Notes

- **Fairness**: Remote APIs (Jev, Sonnet, Gemini) include network latency. Laya runs locally, so its latency is compute-only.
- **OTel**: Needs 4 requests per incident (one per question), run in parallel.
- **Ground Truth**: Rule-based, measures policy-following accuracy, not telecom expertise.
- **Complex Incidents**: Section 7b includes 3 narrative edge cases to test unstructured reasoning vs. structured parsing.

## License

This is a benchmark notebook for research and evaluation purposes.
