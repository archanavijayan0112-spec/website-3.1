<div align="center">

```
  █████╗ ██╗    ██╗███████╗██████╗
 ██╔══██╗██║    ██║██╔════╝██╔══██╗
 ███████║██║    ██║█████╗  ██████╔╝
 ██╔══██║██║    ██║██╔══╝  ██╔══██╗
 ██║  ██║██║    ██║███████╗██████╔╝
 ╚═╝  ╚═╝╚═╝    ╚═╝╚══════╝╚═════╝
 S C R A P E R  +  E X T R A C T O R
```

**AI-powered web scraping and structured data extraction API**

[![Python](https://img.shields.io/badge/Python-3.11+-3776ab?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![LangChain](https://img.shields.io/badge/LangChain-0.3-1c3c3c?style=flat-square)](https://langchain.com)
[![Playwright](https://img.shields.io/badge/Playwright-1.49-2ead33?style=flat-square&logo=playwright&logoColor=white)](https://playwright.dev)
[![License](https://img.shields.io/badge/License-MIT-7c5cfc?style=flat-square)](LICENSE)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ed?style=flat-square&logo=docker&logoColor=white)](Dockerfile)

[**Live Demo**](https://yourname.github.io/ai-web-scraper) · [**API Docs**](http://localhost:8000/docs) · [**Report Bug**](https://github.com/yourname/ai-web-scraper/issues)

</div>

---

## What is this?

Most scraping tools make you write CSS selectors or XPath expressions that break every time a website updates its layout.

This project takes a different approach: **you describe what you want in plain English**, and the AI figures out how to extract it — from any page, in any layout, every time.

```bash
# No selectors. Just a schema.
curl -X POST http://localhost:8000/api/v1/scrape/ \
  -d '{
    "url": "https://any-website.com/product",
    "schema_def": {
      "fields": [
        {"name": "title",    "description": "Product name"},
        {"name": "price",    "description": "Price with currency symbol"},
        {"name": "in_stock", "description": "Whether item is in stock", "field_type": "boolean"}
      ]
    }
  }'

# → {"title": "Widget Pro", "price": "$49.99", "in_stock": true}
```

---

## Features

| | Feature | Details |
|---|---|---|
| 🧠 | **AI extraction** | LangChain + GPT-4o-mini extracts typed fields from any page |
| ⚡ | **Dual scraping engines** | httpx for speed, Playwright for JS-rendered sites |
| 📋 | **Async batch jobs** | Scrape up to 100 URLs concurrently, track progress live |
| 🔄 | **Auto pagination** | Detects and follows `Next` links automatically |
| 📦 | **Multi-format export** | Download results as JSON, CSV, or styled Excel |
| 🔁 | **Retry + rate limiting** | Exponential backoff, configurable delays between requests |
| 🗄️ | **Persistent storage** | SQLite for dev, PostgreSQL for production |
| 🐳 | **Docker deploy** | One command spins up API + database + Redis |
| 📖 | **Auto Swagger docs** | Interactive docs at `/docs`, ReDoc at `/redoc` |
| ✅ | **Test suite** | 15 pytest tests with async support and mocked LLM |

---

## Quick Start

### Prerequisites

- Python 3.11+
- An [OpenAI API key](https://platform.openai.com/api-keys) (for AI extraction)

### Install

```bash
git clone https://github.com/yourname/ai-web-scraper.git
cd ai-web-scraper

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
playwright install chromium     # headless browser for JS pages
```

### Configure

```bash
cp .env.example .env
```

Edit `.env` and add your OpenAI key:

```env
OPENAI_API_KEY=sk-your-key-here
```

### Run

```bash
uvicorn app.main:app --reload
```

Visit **http://localhost:8000/docs** for the interactive Swagger UI.

---

## Docker Deploy

```bash
export OPENAI_API_KEY=sk-your-key-here

# Starts API + PostgreSQL + Redis
docker-compose up -d

# Check health
curl http://localhost:8000/health
# → {"status": "ok", "version": "1.0.0"}
```

---

## API Usage

### Single URL — Quick scrape (no AI needed)

```bash
curl "http://localhost:8000/api/v1/scrape/quick?url=https://news.ycombinator.com"
```

```json
{
  "title": "Hacker News",
  "status_code": 200,
  "duration_ms": 312,
  "extracted": {
    "title": "Hacker News",
    "headings": {"h1": ["Hacker News"]},
    "links": [...],
    "images": [...],
    "open_graph": {}
  }
}
```

---

### Single URL — AI extraction with schema

```bash
curl -X POST http://localhost:8000/api/v1/scrape/ \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://books.toscrape.com/catalogue/a-light-in-the-attic_1000/index.html",
    "schema_def": {
      "fields": [
        {"name": "title",       "description": "Book title",            "field_type": "string"},
        {"name": "price",       "description": "Price with currency",   "field_type": "string"},
        {"name": "rating",      "description": "Star rating out of 5",  "field_type": "number"},
        {"name": "in_stock",    "description": "Is the book in stock",  "field_type": "boolean"},
        {"name": "description", "description": "Book synopsis",         "field_type": "string"}
      ]
    }
  }'
```

```json
{
  "url": "https://books.toscrape.com/...",
  "page_title": "A Light in the Attic | Books to Scrape",
  "extracted_data": {
    "title": "A Light in the Attic",
    "price": "£51.77",
    "rating": 3,
    "in_stock": true,
    "description": "It's hard to imagine a world without A Light in the Attic..."
  },
  "extraction_tokens_used": 892
}
```

---

### JS-rendered pages (Playwright)

```bash
curl -X POST http://localhost:8000/api/v1/scrape/ \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://spa-website.com/data",
    "options": {
      "use_playwright": true,
      "wait_for_selector": ".data-table"
    },
    "schema_def": {
      "fields": [{"name": "rows", "description": "All table rows", "field_type": "list"}]
    }
  }'
```

---

### Batch scrape — async job

```bash
# Submit
curl -X POST http://localhost:8000/api/v1/scrape/batch \
  -H "Content-Type: application/json" \
  -d '{
    "urls": [
      "https://site.com/p/1",
      "https://site.com/p/2",
      "https://site.com/p/3"
    ],
    "schema_def": {
      "fields": [
        {"name": "headline", "description": "Article headline"},
        {"name": "author",   "description": "Author name"},
        {"name": "tags",     "description": "Article tags", "field_type": "list"}
      ]
    },
    "options": {"delay_seconds": 1.0}
  }'

# → {"job_id": "a3f2c1b8-...", "message": "Batch job queued with 3 URLs"}

# Poll status
curl http://localhost:8000/api/v1/jobs/a3f2c1b8-...

# → {"status": "running", "progress": 66, "scraped_urls": 2, "total_urls": 3}

# Get results
curl "http://localhost:8000/api/v1/jobs/a3f2c1b8-.../results?limit=10"
```

---

### Export results

```bash
# CSV
curl -X POST http://localhost:8000/api/v1/jobs/a3f2c1b8-.../export \
  -H "Content-Type: application/json" \
  -d '{"job_id": "a3f2c1b8-...", "format": "csv"}' \
  --output results.csv

# Excel
curl -X POST http://localhost:8000/api/v1/jobs/a3f2c1b8-.../export \
  -H "Content-Type: application/json" \
  -d '{"job_id": "a3f2c1b8-...", "format": "excel"}' \
  --output results.xlsx

# JSON
curl -X POST http://localhost:8000/api/v1/jobs/a3f2c1b8-.../export \
  -H "Content-Type: application/json" \
  -d '{"job_id": "a3f2c1b8-...", "format": "json"}' \
  --output results.json
```

---

### Extract from raw HTML (no scraping)

```bash
curl -X POST http://localhost:8000/api/v1/extract/ \
  -H "Content-Type: application/json" \
  -d '{
    "content": "<p>Apple iPhone 15 Pro — $999 — Available in Black Titanium</p>",
    "schema_def": {
      "fields": [
        {"name": "product", "description": "Product name"},
        {"name": "price",   "description": "Price in USD"},
        {"name": "color",   "description": "Color variant"}
      ]
    }
  }'
```

---

## Extraction Schema Reference

```json
{
  "fields": [
    {
      "name":        "field_name",      // key in output JSON
      "description": "What it contains", // plain English — this is what the AI reads
      "field_type":  "string",          // string | number | boolean | list | object
      "required":    true,              // optional, default true
      "example":     "some example"     // optional hint for the AI
    }
  ],
  "instructions": "Extra instructions for the AI extractor (optional)"
}
```

---

## Scraping Options

| Option | Type | Default | Description |
|---|---|---|---|
| `use_playwright` | `bool` | `false` | Use headless browser for JS-rendered pages |
| `wait_for_selector` | `string` | `null` | CSS selector to wait for before extracting |
| `custom_headers` | `object` | `null` | Additional HTTP headers (cookies, auth, etc.) |
| `follow_pagination` | `bool` | `false` | Auto-follow Next page links |
| `max_pages` | `int` | `5` | Max pages when following pagination (1–50) |
| `delay_seconds` | `float` | `1.0` | Polite delay between requests (0–10s) |

---

## Project Structure

```
ai-web-scraper/
│
├── app/
│   ├── main.py                     # FastAPI app + lifespan setup
│   │
│   ├── api/routes/
│   │   ├── scrape.py               # POST /scrape/, /batch, /quick
│   │   ├── extract.py              # POST /extract/, /extract/url
│   │   ├── jobs.py                 # GET/POST/DELETE /jobs/
│   │   └── health.py               # GET /health
│   │
│   ├── core/
│   │   ├── config.py               # All settings via pydantic-settings
│   │   ├── database.py             # Async SQLAlchemy engine + session DI
│   │   └── logging.py              # Structured logging (stdout + file)
│   │
│   ├── models/
│   │   ├── db_models.py            # ScrapeJob + ScrapedResult ORM models
│   │   └── schemas.py              # Pydantic v2 request/response schemas
│   │
│   └── services/
│       ├── scraper.py              # httpx + Playwright engines, retry, pagination
│       ├── extractor.py            # LangChain AI extraction + rule-based fallback
│       ├── job_runner.py           # Async concurrent batch runner with semaphore
│       └── exporter.py             # JSON / CSV / styled Excel export
│
├── tests/
│   └── test_scraper.py             # 15 pytest tests (async, mocked)
│
├── website/
│   └── index.html                  # Project landing page
│
├── .env.example                    # Environment variable template
├── .gitignore
├── Dockerfile
├── docker-compose.yml              # API + PostgreSQL + Redis
├── pytest.ini
├── requirements.txt
└── README.md
```

---

## Configuration Reference

All settings are read from environment variables or a `.env` file:

| Variable | Default | Description |
|---|---|---|
| `OPENAI_API_KEY` | — | **Required** for AI extraction |
| `LLM_MODEL` | `gpt-4o-mini` | OpenAI model (`gpt-4o`, `gpt-3.5-turbo`, etc.) |
| `LLM_TEMPERATURE` | `0.0` | Keep at 0 for deterministic extraction |
| `DATABASE_URL` | SQLite | Async DB URL (use `asyncpg` for PostgreSQL) |
| `REDIS_URL` | `redis://localhost:6379/0` | Redis for job queue |
| `MAX_CONCURRENT_SCRAPES` | `10` | Global concurrency cap |
| `DEFAULT_TIMEOUT` | `30` | HTTP request timeout in seconds |
| `MAX_RETRIES` | `3` | Retry attempts per URL |
| `PLAYWRIGHT_HEADLESS` | `true` | Run browser without a GUI |
| `PLAYWRIGHT_BROWSER` | `chromium` | `chromium`, `firefox`, or `webkit` |
| `EXPORT_DIR` | `./exports` | Directory for downloaded files |
| `DEBUG` | `false` | Enable SQLAlchemy echo + verbose logs |
| `WORKERS` | `4` | Uvicorn worker count |

---

## Running Tests

```bash
pytest tests/ -v

# With coverage
pip install pytest-cov
pytest tests/ -v --cov=app --cov-report=html
open htmlcov/index.html
```

**15 tests** covering:
- httpx scraping with mocked responses
- Title extraction and pagination link detection
- Retry logic on failures
- HTML cleaning and truncation
- AI extraction with mocked LLM
- Rule-based extraction (no API key)
- Pydantic schema validation (valid and invalid inputs)

---

## Roadmap

- [ ] Rate limiting per API key
- [ ] Webhook callbacks on job completion
- [ ] Proxy rotation support
- [ ] Scheduled recurring scrape jobs
- [ ] Output schema validation with Pydantic
- [ ] Streaming progress via Server-Sent Events
- [ ] Web UI dashboard

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Commit your changes: `git commit -m 'feat: add my feature'`
4. Push: `git push origin feat/my-feature`
5. Open a Pull Request

Please run `pytest tests/ -v` before submitting and add tests for new features.

---

## License

[MIT](LICENSE) — free to use, modify, and distribute.

---

<div align="center">
Built with <a href="https://fastapi.tiangolo.com">FastAPI</a> · <a href="https://langchain.com">LangChain</a> · <a href="https://playwright.dev">Playwright</a>
</div>
