# SoS Regulation Assistant

## Overview

This repository contains the code and documentation for a Mississippi Artificial Intelligence Innovation Hub Proof of Concept focused on **multi-state Secretary of State regulatory intelligence**. The PoC was developed to explore whether retrieval-augmented generation (RAG) over administrative rules could help agency staff and researchers find, compare, and cite regulations across several southeastern states. The project demonstrates feasibility within a limited prototype environment and is **not a production-ready solution**.

## Agency Problem

Regulatory rules for dental boards, medical licensure boards, and real estate commissions are published across many state websites with different formats (PDF, HTML, DOCX, and SPA-driven portals). Staff need a single place to ask natural-language questions, see cited sources, and compare requirements between states without manually searching each code system.

## PoC Scope and Demonstrated Capabilities

- **RAG chat assistant** — Streamlit UI (`src/app.py`) queries Amazon Bedrock Knowledge Bases with citations and optional multi-turn context.
- **Automated crawler** — Scrapy + Playwright pipeline (`src/sos_crawler/`) for **MS, AL, AR, GA, LA, TN, and TX**, focused on three agency types per state.
- **State scope controls** — Sidebar checkbox grid to include or exclude states per query.
- **Post-crawl tooling** — QA and enrichment CLI for manifests and knowledge-package JSONL.
- **CI crawl workflow** — Scheduled GitHub Actions run (artifacts only; no regulatory corpora committed to git).

## Prerequisites

- **Python 3.12+**
- **[uv](https://github.com/astral-sh/uv)** (recommended) or pip
- **AWS account** with Bedrock Knowledge Base access — **required for the RAG app only**
- **Playwright Chromium** — **required only if you run crawlers for MS, AL, or TX**

## Quickstart

Follow these steps to run the project locally without reading other files first.

### 1. Clone and install

```bash
git clone https://github.com/spicyneutrino/AI-Innovation-Phase-1.git
cd AI-Innovation-Phase-1
uv sync
```

### 2. Configure environment

```bash
cp .env.example .env
```

Edit `.env` and set at minimum:

| Variable | Purpose |
|----------|---------|
| `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` | AWS credentials (omit if using IAM roles) |
| `AWS_DEFAULT_REGION` | e.g. `us-east-1` — must match your Bedrock KB |
| `BEDROCK_KB_ID` | Your Knowledge Base ID |
| `APP_PASSWORD` | Password for the Streamlit login screen |

On **Streamlit Cloud**, put the same keys in `.streamlit/secrets.toml` (gitignored). Never commit `.env` or real production secrets.

### 3. Run the RAG assistant

```bash
uv run streamlit run src/app.py
```

Open the URL in the terminal (default `http://localhost:8501`). Sign in with `APP_PASSWORD`, select state scope in the sidebar (or clear all to search the full knowledge base), then ask questions or use the suggested prompts.

### 4. Optional — smoke-test the crawler

HTTP-only Arkansas is a fast check that the crawler toolchain works:

```bash
uv run sos-crawler crawl --states AR --mode designated
```

For Playwright states (MS, AL, TX), install the browser first:

```bash
uv run playwright install chromium
uv run sos-crawler crawl --states AL --mode designated --max-retries 0
```

Outputs appear under `var/sos_crawler/` (logs, downloads, manifests). This directory is gitignored.

## Common tasks

| Task | Command / link |
|------|----------------|
| Full quickstart (above) | Steps 1–4 in this README |
| All CLI flags and command combinations | [docs/DETAIL.md](docs/DETAIL.md#cli-reference) |
| Run all states in parallel (default 4 workers) | `uv run sos-crawler crawl` — see [parallel execution](docs/DETAIL.md#parallel-execution) |
| Designated agencies only (dental, medical, real estate) | `uv run sos-crawler crawl --mode designated --states AL AR TX` |
| Crawl + QA + enrichment (CI style) | `uv run sos-crawler crawl --run-qa --run-enrichment --max-retries 2` |
| Sequential / low-memory crawl | `uv run sos-crawler crawl --max-workers 1` |
| Debug one spider | `uv run python -m scrapy crawl arkansas` — see [DETAIL.md](docs/DETAIL.md#direct-scrapy-debug-single-spider) |
| Post-process existing output | `uv run sos-crawler qa` then `uv run sos-crawler enrich` |
| Docker crawler | [docs/setup.md](docs/setup.md#docker) and [DETAIL.md](docs/DETAIL.md#automation-ci-docker-lambda) |
| Per-state spider notes | [docs/DETAIL.md](docs/DETAIL.md#per-state-spiders) |

## Architecture Overview

See [docs/architecture.md](docs/architecture.md) for a short component diagram. For end-to-end data flow, pipeline order, parallelism, and environment variables, see the **[detailed operations guide](docs/DETAIL.md)**.

## Repository Structure

```
├── README.md
├── LICENSE
├── .env.example
├── CHANGELOG.md
├── docs/                 # Architecture, setup, DETAIL, data, limitations, testing
├── src/
│   ├── app.py            # Streamlit RAG UI
│   ├── rag_engine.py     # Bedrock RetrieveAndGenerate wrapper
│   ├── style.css         # UI theme
│   └── sos_crawler/      # Crawler package (spiders, pipelines, tools)
├── scripts/              # S3 upload and metadata helpers
├── .github/workflows/    # Scheduled crawler CI
├── Dockerfile            # Crawler container (Playwright)
└── pyproject.toml        # uv / package metadata
```

Generated crawl output lives under `var/sos_crawler/` (gitignored). See [docs/DETAIL.md — Runtime outputs](docs/DETAIL.md#runtime-outputs).

## Configuration

Copy [.env.example](.env.example) to `.env` for local development. Crawler-only variables (`SOS_CRAWLER_RUNTIME_DIR`, `PLAYWRIGHT_HEADLESS`, etc.) are documented in [docs/DETAIL.md — Environment variables](docs/DETAIL.md#environment-variables).

## Data Notes

**This repository does not include real regulatory data.** Indexing, S3 layout, and Bedrock sync are operator responsibilities. Details: [docs/data-notes.md](docs/data-notes.md).

## Usage

1. Start the app: `uv run streamlit run src/app.py`
2. Sign in with the configured password.
3. Use the **Scope** sidebar to select states (or clear all to search the full knowledge base without a state filter).
4. Ask questions in the chat or use the suggested prompts on the welcome screen.

**Crawler (optional):**

```bash
uv run playwright install chromium   # if crawling MS, AL, or TX
uv run sos-crawler crawl --states MS AL AR --mode designated --run-qa --run-enrichment
```

## Documentation

| Document | Contents |
|----------|----------|
| **[docs/DETAIL.md](docs/DETAIL.md)** | **Comprehensive guide:** diagrams, CLI flags, parallel runs, spiders, outputs, debugging |
| [docs/setup.md](docs/setup.md) | Streamlit Cloud, AWS, Docker, distrobox, troubleshooting |
| [docs/architecture.md](docs/architecture.md) | Short system architecture |
| [docs/data-notes.md](docs/data-notes.md) | Data policy and indexing workflow |
| [docs/limitations.md](docs/limitations.md) | PoC scope and production gaps |
| [docs/testing.md](docs/testing.md) | Manual validation and sample queries |

## Testing and Evaluation

Manual validation steps, sample evaluation queries, and CI notes: [docs/testing.md](docs/testing.md). Crawler QA commands and log inspection: [docs/DETAIL.md — Validation and debugging](docs/DETAIL.md#validation-and-debugging).

## Limitations

This PoC was developed within a limited timeline and controlled environment. It may contain simplified workflows, mock integrations, limited testing coverage, and prototype user interfaces.

See [docs/limitations.md](docs/limitations.md) for security, scope, and production gaps.

## Disclaimer

This repository contains code and supporting materials developed as part of a Mississippi Artificial Intelligence Innovation Hub Proof of Concept project. The contents are provided for prototype demonstration purposes. They are not production ready by default and may include simplified workflows, incomplete security guardrails, placeholder integrations, or reduced controls appropriate only for a Proof-of-Concept environment.

Do not use this software with production data or in production environments without additional architecture, security, privacy, testing, and stakeholder review.

## License

Released under the [MIT License](LICENSE).

## Contributors

**SoS Innovation Hub Team** (placeholder — update with final attribution before Hub sign-off).
