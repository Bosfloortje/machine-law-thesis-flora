# Machine Law Thesis — LLM Explanation Research

This repository contains the dataset and code accompanying the thesis:

> **From Rule Execution to Citizen Explanation: Evaluating LLM-Generated Explanations in Rule-Based Government Systems**  
> Floortje Bos

---

## What This Repository Contains

This research investigates whether large language models (LLMs) can generate understandable and legally correct explanations of automated government decisions. Three explanation approaches are compared across five LLMs and three Dutch laws, evaluated by both automated metrics and human annotators (citizens and legal experts).

| Approach | Description |
|---|---|
| `open` | Raw engine output — no explicit decisive condition provided to the LLM |
| `flat` | Decisive condition + key facts provided as plain text bullets (ablation baseline) |
| `graph` | The full decision graph is extracted from the rule engine and provided to the LLM as a structured representation: outcome, citizen-specific facts, each condition with its evaluation result, and the applicable legal articles. The LLM uses this complete graph to generate the explanation (full approach) |

**Models:** GPT-4o, Claude Haiku, Mistral 7B, LLaMA 3.1 8B, DeepSeek R1 8B  
**Laws:** Health Care Benefit law (Zorgtoeslagwet), Social Assistance law (Participatiewet), Alcohol law (Alcoholwet)

---

## Demo: poc-machine-law

The rule execution engine, law definitions, citizen profiles, and web interface in this repository are based on [poc-machine-law](https://github.com/MinBZK/poc-machine-law), a proof-of-concept developed by the Dutch Ministry of the Interior and Kingdom Relations (Ministerie van Binnenlandse Zaken en Koninkrijksrelaties). This demo was built to explore what automated rule execution and citizen-facing explanation could look like in future government systems — it is not a production system.

This thesis builds on that demo by adding an LLM explanation layer and evaluating whether the generated explanations are understandable and legally correct.

---

## How to Use This Repository

### 1. Setup
```bash
uv sync
```

### 2. Explore the generated explanations
All thesis output is in `llm_explanations/output/final_output_complete/` — 5 models × 3 approaches × 3 laws, 200 profiles each. Each JSONL file contains one explanation record per citizen profile including the evaluation trace (decisive condition, key facts, outcome).

### 3. Explore the human annotation data
Survey responses from citizens and legal experts are in `llm_explanations/annotations/input/`. Parsed results (scores, inter-rater agreement, correlations with automated metrics) are in `llm_explanations/annotations/results/`.

### 4. Reproduce the evaluation
```bash
# Automated metrics (faithfulness + readability + contestability)
uv run python llm_explanations/scripts/evaluation/evaluate.py \
    --input llm_explanations/output/final_output_complete/haiku/graph/graph_haiku_zorgtoeslag.jsonl

# Correlation between automated metrics and human scores
uv run python llm_explanations/scripts/evaluation/correlate.py
```

### 5. Generate new explanations
```bash
# Open approach — raw engine output, no explicit decisive condition
uv run python llm_explanations/scripts/extract.py \
    --approach open --law zorgtoeslag --models haiku

# Flat approach — decisive condition + key facts as plain text (ablation baseline)
uv run python llm_explanations/scripts/extract.py \
    --approach flat --law zorgtoeslag --models haiku

# Graph approach — full decision graph provided to the LLM (full approach)
uv run python llm_explanations/scripts/extract.py \
    --approach graph --law zorgtoeslag --models haiku
```

### 6. Generate synthetic citizen profiles
```bash
# 200 CBS-weighted profiles (no LLM required)
uv run python llm_explanations/scripts/profiles/generate_profiles.py --count 200
```

### 7. Run the web interface
```bash
export ANTHROPIC_API_KEY=...
export FEATURE_CHAT=1
uv run web/main.py    # available at http://localhost:8000
```

---

## Repository Structure

```
machine-law-thesis/
├── llm_explanations/               # Main research directory
│   ├── scripts/                    # Extraction, evaluation, chat, profile generation
│   │   ├── evaluation/             # Automated metrics (Faithfulness, Flesch, Contestability)
│   │   └── profiles/               # Rule-based CBS-weighted profile generator
│   ├── annotations/                # Human annotation data and parsing scripts
│   │   ├── first_initial_look/     # Exploratory pre-study (informed methodology design)
│   │   ├── input/                  # Final survey responses (3 laws × 2 rater groups)
│   │   └── results/                # Parsed CSVs: scores, inter-rater, correlations
│   └── output/
│       ├── final_output_complete/  # All thesis results — 5 models × 3 approaches × 3 laws
│       ├── evaluation_output/      # Automated evaluation results
│       └── chat/                   # Multi-turn chat output
├── explain/                        # LLM engine: providers, graph context, guard
├── laws/                           # Machine-readable law definitions (YAML)
├── data/                           # Synthetic citizen profiles (profiles.yaml)
└── web/                            # Web interface for interactive chat
```

---

## Environment Variables

| Variable | Required for | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | Claude / Haiku | Anthropic API key |
| `OPENAI_API_KEY` | GPT-4o | OpenAI API key |
| `FEATURE_CHAT` | Web chat | Set to `1` to enable chat endpoint |

Local open-source models (Mistral, LLaMA, DeepSeek) require [Ollama](https://ollama.ai):
```bash
ollama serve
ollama pull mistral:7b
ollama pull llama3.1:8b
ollama pull deepseek-r1:8b
```

---

## Laws

| Law | Dutch name | Slug |
|---|---|---|
| Health Care Benefit law | Zorgtoeslagwet | `zorgtoeslag` |
| Social Assistance law | Participatiewet | `participatiewet/bijstand` |
| Alcohol law | Alcoholwet | `alcoholwet/vergunning` |
