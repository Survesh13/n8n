# AI-Powered Job Search Automation

**An end-to-end workflow that scrapes job listings, filters them by fit using an LLM, tailors application materials automatically, and tracks everything — with zero manual effort after launch.**

---

## The Problem

Job searching at scale means repeating the same three steps hundreds of times: find listings, figure out which ones are actually worth applying to, and tailor your resume for each one. It's repetitive, time-consuming, and easy to burn out on — exactly the kind of process that should be automated.

## What It Does

This workflow runs the entire job-search loop end-to-end:

1. **Scrapes job listings** from a saved LinkedIn search using an Apify scraping actor, pulling structured data (title, description, company) for each posting.
2. **Loads a base resume** from Google Docs to use as the reference point for every downstream decision.
3. **Evaluates job fit with an LLM** — each listing's description is passed to GPT-4.1 alongside the resume, and the model returns a structured verdict (fit / not a fit) based on the candidate's actual skills and experience.
4. **Filters out mismatches** automatically, so only genuinely relevant roles move forward.
5. **Tailors the resume per role** — for each matching job, GPT-4.1 rewrites Resume content to better emphasise the skills and experience most relevant to that specific posting.
6. **Generates a new document** for each tailored resume via the Google Docs API.
7. **Logs the outcome** — job title, company, fit verdict, and generated document link — to a Google Sheet for tracking and follow-up.

## Architecture

```
Manual Trigger
    → Set Job Search URL
    → Scrape Jobs (Apify)
    → Loop Over Items (batch processing)
        → Get Resume (Google Docs)
        → Check Relevance (OpenAI GPT-4.1)
        → Filter (fit / not fit)
        → Customise Resume (OpenAI GPT-4.1)
        → Create Document (Google Docs)
        → Append Row (Google Sheets)
    → Loop
```

## Tech Stack

| Component | Tool |
|---|---|
| Orchestration | [n8n](https://n8n.io) (self-hosted) |
| Job scraping | [Apify](https://apify.com) — LinkedIn Jobs Scraper actor |
| Fit evaluation & resume tailoring | OpenAI API (GPT-4.1-mini) |
| Resume storage & generation | Google Docs API |
| Application tracking | Google Sheets API |

## Key Design Decisions

**Why LLM classification instead of keyword matching?**
Keyword filters miss context — a job requiring "3+ years SQL" vs. "SQL preferred" reads identically to a keyword matcher but very differently to a human reviewer. Passing the full job description and resume to an LLM lets the fit decision account for that nuance, at the cost of needing a structured, constrained output format (a strict JSON verdict) to keep results usable downstream.

**Why batch processing (`Loop Over Items`) instead of a single bulk call?**
Processing jobs one at a time keeps each fit-check and resume-tailoring call scoped to a single job description, which both improves output quality (no cross-contamination between postings) and makes failures isolated and easy to debug.

**Guardrails on resume tailoring**
Because an LLM is rewriting resume content, controlling for factual accuracy matters — the model is only permitted to reframe and reprioritize *existing* resume content for relevance, not invent new experience. This is enforced through prompt constraints rather than blind trust in the output. *(Update this section with the specific constraints you used in the "Customise Resume" prompt.)*

## Results

- Reduced time spent screening and tailoring per application from ~20 minutes to under 1 minute.
- Applied to 25 roles with a 90% response rate.

## What I'd Improve Next

- Add a manual approval step before auto-submitting or sending, to keep a human in the loop on the highest-stakes actions
- Expand fit evaluation to return a confidence score rather than a binary verdict, to support a "maybe" tier
- A/B test tailored vs. untailored resumes to quantify actual impact on response rate
