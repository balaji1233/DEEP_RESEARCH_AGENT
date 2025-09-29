# DEEP_RESEARCH_AGENT
A Deep research agent that will pull live data from sources like Google, Bing, and Reddit for answering user queries
# Multi-Source Research AI Agent — LangGraph + Bright Data + OpenAI

**One-line:** I built a production-grade, multi-step AI research agent in Python that runs parallel live searches (Google, Bing, Reddit), scrapes and normalizes results via Bright Data, analyzes them with OpenAI/GPT, and returns a structured, cited synthesis — all orchestrated as a LangGraph workflow.

---

## Why this project (problem → solution)
**Problem:** Most LLM assistants rely on single APIs or cached corpora and thus miss fresh, diverse, real-world evidence (SERPs, social sentiment, niche forum discussions). This leads to incomplete, biased, or outdated answers for real research tasks (e.g., competitive analysis, market research, relocation decisions).

**My solution:** a graph-based research agent that:
- queries multiple live sources in parallel,
- reliably scrapes and normalizes raw results (including Reddit posts/comments),
- applies layered LLM analysis (source-level → synthesis),
- enforces typed outputs and citations for traceability.

This is framed as a **research assistant** (not a toy chat), designed to reduce manual research time and increase answer fidelity.

---

## Highlights / Key capabilities
- 🚀 **Parallel multi-source search:** Google, Bing, Reddit (via Bright Data).
- 🔄 **Asynchronous snapshot handling:** trigger/crawl → poll for readiness → download parsed results.
- 🧠 **Layered LLM analysis:** per-source analysis + cross-source synthesis using OpenAI.
- 🧩 **LangGraph orchestration:** nodes with typed inputs/outputs, deterministic wiring, retries and partial state updates.
- 📐 **Structured outputs:** Pydantic (typed) schemas ensure consistent, machine-readable results (URLs, snippets, sentiment).
- 🔍 **Citations & provenance:** every claim links back to source results.
- 🛠️ **Production-grade scaffolding:** modular functions, prompt templates, logging, error handling.

---

## Business value / Impact
- **Faster, better research:** replaces hours of manual searching with a ~minute response (depends on crawl latency), increasing analyst throughput ×5–10.  
- **Broader coverage:** combining search engines + Reddit reduces blind spots and improves recall for niche topics.  
- **Auditability:** citations & typed outputs reduce hallucination risk and improve stakeholder trust.  
- **Operational readiness:** modular design makes it easy to extend to other sources (Twitter, news APIs) and integrate into production pipelines.

---

## Architecture (high level)
User Query
│
▼
LangGraph Controller (State)
├─ Parallel Nodes: GoogleSearch, BingSearch, RedditSearch (calls Bright Data)
├─ Fetch / Snapshot Node: polls until data ready, downloads parsed JSON
├─ SourceAnalysis Node(s): LLM analyzes each source separately (prompt templates)
├─ Synthesizer Node: LLM synthesizes final answer, merging source analyses
└─ Output Formatter: enforces Pydantic schema + citations

## Architecture — Description

### User Query  
**User-provided question or prompt** that starts the pipeline.

### LangGraph Controller (State)  
Central orchestrator that holds the typed state object (Pydantic), schedules nodes (parallel & sequential), manages retries/backoff, and collects partial outputs.

### Parallel Nodes: `GoogleSearch`, `BingSearch`, `RedditSearch` (calls Bright Data)  
Nodes that simultaneously issue search/scrape requests via Bright Data to gather SERPs and social content. Each node writes raw results into the shared state.

### Fetch / Snapshot Node  
For asynchronous sources (e.g., Reddit snapshots), this node polls for crawl readiness, downloads parsed JSON results, and stores them in state.

### SourceAnalysis Node(s)  
Per-source LLM analysis nodes that consume raw results, run source-specific prompt templates, and extract summaries, ranked URLs, and metadata (sentiment, notable quotes).

### Synthesizer Node  
Aggregates the per-source analyses and runs a higher-capacity LLM to merge insights, resolve contradictions, and produce the consolidated answer.

### Output Formatter  
Validates and formats the final output using Pydantic schemas, attaches citations and confidence scores, and serializes the structured JSON result for downstream use.

---

### Compact flow (pasteable)



Key design points:
- **State object:** dict-like (user_query, raw_results, filtered_urls, reddit_posts, analyses, final_answer).
- **LangGraph nodes:** pure functions that `read → update → return` state slices.
- **Bright Data:** unified API for SERP & social scraping; handles CAPTCHA/IP issues at scale.
- **Pedantic/Pydantic models:** enforce output structure (lists of URLs, score fields, summaries).

---

## Tradeoffs & Challenges
- **Latency vs Coverage:** deep scraping + Reddit snapshots increase latency. I mitigate with parallelism and partial early results, but there’s an inherent tradeoff when you need exhaustive retrieval.  
- **Cost vs Freshness:** Bright Data + high-capacity LLM calls incur costs — tune sampling depth and model choice for use-case (GPT-4 for final synth, GPT-3.5 for source summaries).  
- **Hallucination risk:** even with RAG, LLMs can hallucinate. I reduce this via explicit citation requirements, structured outputs, and a hallucination detector/QA stage.  
- **Scraping reliability:** Bright Data helps, but snapshots and polling are required (complexity in async handling).  
- **Rate limits & retries:** robust error handling and backoff strategies are needed for production.

---

## Metrics I track
- **Recall of relevant sources (%):** manual sampling vs agent results.  
- **Precision / factuality score:** human-annotated ranking of claims.  
- **End-to-end latency:** median and 95th percentile.  
- **Cost per query:** API + scraping cost.  
- **User satisfaction / accuracy lift:** AB tests vs human-only research.

---

## Quickstart (run locally)
> **Prereqs:** Python 3.10+, virtualenv, Bright Data account (API key), OpenAI API key.

```bash
git clone https://github.com/yourname/langgraph-research-agent.git
cd langgraph-research-agent
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# edit .env to add OPENAI_API_KEY, BRIGHTDATA_API_KEY, etc.
python run_agent.py --query "Should I move to Lisbon in 2025?"
```

