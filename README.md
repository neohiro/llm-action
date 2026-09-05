# neohiro/llm-action

> Vendors the [FreeModelsRouter](https://github.com/neohiro/LLM) + provider/model data files into
> your repository, sets all required `*_API_KEY` environment variables, and exposes
> structured outputs so your pipeline can call `llm/score_milestone.py` with zero manual vendoring.

## Usage

```yaml
- uses: neohiro/llm-action@v1
  with:
    llm-router-enabled: true
    openai-key: ${{ secrets.OPENAI_API_KEY }}
    anthropic-key: ${{ secrets.ANTHROPIC_API_KEY }}
    groq-key: ${{ secrets.GROQ_API_KEY }}
    cerebras-key: ${{ secrets.CEREBRAS_API_KEY }}
    sambanova-key: ${{ secrets.SAMBANOVA_API_KEY }}
    github-models-key: ${{ secrets.GITHUB_MODELS_API_KEY }}
    cloudflare-key: ${{ secrets.CLOUDFLARE_API_KEY }}
    cloudflare-account-id: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    openrouter-key: ${{ secrets.OPENROUTER_API_KEY }}
    google-ai-key: ${{ secrets.GOOGLE_AI_API_KEY }}
    huggingface-key: ${{ secrets.HUGGINGFACE_API_KEY }}
    nvidia-key: ${{ secrets.NVIDIA_API_KEY }}
    cohere-key: ${{ secrets.COHERE_API_KEY }}
    llm-model: gpt-4o
    data-dir: data
    cache-pip: true
    cache-path: requirements.txt

# Your scoring step — the action sets all env vars and writes router data files.
- run: python llm/score_milestone.py
  env:
    ROUTER_DATA_DIR: ${{ steps.llm-router.outputs.data-dir }}
```

## How it works

1. **Vendor router** — copies `llm/router.py` (FreeModelsRouter) and the 4 JSON data files
   (`providers.json`, `models.json`, `free_models.json`, `unlimited.json`) into your workspace
   at the paths your `llm/score_milestone.py` expects.
2. **BOM defence** — strips any UTF-8 BOM from vendored JSON files, preventing silent parse errors.
3. **Set env vars** — writes all `*_API_KEY`, `LLM_MODEL`, and `LLM_ROUTER_ENABLED` variables
   to `$GITHUB_ENV`, making them available to all subsequent steps in the job.
4. **Outputs** — `data-dir`, `router-module-path`, `available-providers` for downstream consumers.

## Inputs

| Input | Default | Description |
|---|---|---|
| `router-version` | `bundled` | Tag/branch/SHA of neohiro/LLM to vendor from. Use `bundled` for the version pinned in this action. |
| `data-dir` | `data` | Relative or absolute path where the 4 JSON data files are written. |
| `python-version` | `3.12` | Python version to set up. |
| `cache-pip` | `true` | Whether to cache pip packages. |
| `cache-path` | `requirements.txt` | Path to requirements.txt for cache key. |
| `openai-key` | `''` | OpenAI API key. |
| `anthropic-key` | `''` | Anthropic API key. |
| `llm-model` | `gpt-4o` | Default LLM model. |
| `llm-router-enabled` | `true` | Enable FreeModelsRouter cascade. |
| `groq-key` … `cohere-key` | `''` | Free provider API keys (all optional). |

## Outputs

| Output | Description |
|---|---|
| `router-version` | Effective version (`bundled` or resolved from input). |
| `data-dir` | Absolute path to the data directory with JSON files. |
| `router-module-path` | Absolute path to the vendored `router.py` directory. |
| `available-providers` | Comma-separated provider names whose keys were supplied. |

## Replacing vendored files in existing repos

After adopting this action, you can delete:

```bash
rm llm/router.py
rm data/providers.json data/models.json data/free_models.json data/unlimited.json
```

The action writes them fresh on every run, with BOM stripping applied automatically.
