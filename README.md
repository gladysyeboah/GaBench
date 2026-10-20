# GaBench

A multiple-choice benchmark for evaluating LLMs on **Ga language** exams (BECE - Basic Education Certificate Examination).

Ga is a Kwa language spoken by approximately 745,000 people in the Greater Accra region of Ghana.

## Dataset

| Language | ISO 639-3 | Family | Questions | Source |
|----------|-----------|--------|-----------|--------|
| Ga | gaa | Kwa | 879 | King Quaye Ga Past Questions (BECE) |

The dataset is derived from `raw/ghanaian_mmlu_ga_bece.xlsx` and converted to `data/ga_mmlu_bece.jsonl`.

Each entry contains:
- `question_id` — unique identifier
- `language` — "ga"
- `test` — "bece"
- `question` — the exam question in Ga
- `options` — list of 4 answer options (A–D)
- `answer` — the correct answer text
- `answer_choice` — the correct choice letter (A/B/C/D)
- `year` — exam year
- `subject` — subject name
- `source` — source attribution

## Quick Start

```bash
pip install -r requirements.txt

# Dry run (no model needed)
python evaluate.py --dry-run

# Evaluate an API model via NVIDIA Build
export NVIDIA_BUILD_API_KEY=your_key_here
python evaluate.py --model nvidia/deepseek-ai/deepseek-v4-flash --languages ga

# Evaluate an OpenAI model
export OPENAI_API_KEY=sk-...
python evaluate.py --model openai/gpt-4o --languages ga

# Evaluate a local HuggingFace model
python evaluate.py --model meta-llama/Llama-3-8B-Instruct --device cuda:0 --languages ga
```

## Supported Backends

### API Models (via NVIDIA Build, OpenAI, Anthropic, Gemini)

```bash
# NVIDIA Build API (OpenAI-compatible)
python evaluate.py --model nvidia/deepseek-ai/deepseek-v4-flash

# OpenAI
python evaluate.py --model openai/gpt-4o

# Anthropic Claude
python evaluate.py --model anthropic/claude-sonnet-4-20250514

# Google Gemini
python evaluate.py --model gemini/gemini-2.0-flash
```

### Local Models (HuggingFace Transformers)

```bash
pip install torch transformers accelerate
python evaluate.py --model meta-llama/Llama-3-8B-Instruct --device cuda:0
```

### Local Models (vLLM)

```bash
pip install vllm
python evaluate.py --model vllm/meta-llama/Llama-3-8B-Instruct
```

## NVIDIA Build API Setup

The GaBench benchmark includes built-in support for the **NVIDIA Build API** (an OpenAI-compatible endpoint at `https://integrate.api.nvidia.com/v1`).

To use NVIDIA Build models:

1. Get your API key from [NVIDIA Build](https://build.nvidia.com/)
2. Set it as an environment variable:
   ```bash
   export NVIDIA_BUILD_API_KEY=your_key_here
   # or with multiple keys for rate-limit rotation:
   export NVIDIA_BUILD_API_KEYS=key1,key2,key3
   ```
3. Run evaluation:
   ```bash
   python evaluate.py --model nvidia/<model-name> --languages ga
   ```

The backend automatically handles:
- API key rotation across a pool of keys
- Rate-limit (429) retry with exponential backoff
- Thinking model support (disables reasoning_effort for fair comparison)

## Options

```
--model, -m          Model spec (required unless --dry-run)
--data, -d           Dataset path (default: data/ga_mmlu_bece.jsonl)
--languages, -l      Filter languages (default: ga)
--tests, -t          Filter tests (e.g. bece)
--output, -o         Results dir (default: results/)
--max-questions      Limit question count
--resume             Resume from checkpoint
--dry-run            Test without model
--device             Device (default: auto)
--dtype              Data type (default: auto)
--max-new-tokens     Generation length (default: 256)
--temperature        Sampling temperature (default: 0.0)
--api-key            API key (or set env var)
```

## Results Format

Results are saved to `results/<model_slug>/`:

- `results.json` — aggregated scores
- `predictions.jsonl` — per-question predictions
- `checkpoint.json` — resumable checkpoint

## Directory Structure

```
GaBench/
├── backends/
│   ├── __init__.py
│   ├── api_backends.py      # OpenAI, Anthropic, Gemini, NVIDIA Build, Mistral
│   ├── base.py              # Abstract backend interface
│   ├── factory.py           # Creates backend from model spec
│   ├── transformers_backend.py  # HuggingFace local
│   └── vllm_backend.py      # vLLM local
├── data/
│   └── ga_mmlu_bece.jsonl   # Ga BECE dataset (879 questions)
├── raw/
│   └── ghanaian_mmlu_ga_bece.xlsx   # Source Excel file
├── results/                 # Benchmark outputs (gitignored)
├── hf-space/                # Static HTML leaderboard
├── evaluate.py              # Main evaluation script
├── leaderboard.py           # Compare results across models
├── prepare_dataset.py       # Convert raw data to JSONL
├── CONTRIBUTING.md
├── requirements.txt
└── README.md
```

## License

Data sourced from King Quaye Ga Past Questions (BECE).
