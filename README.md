# PromptOpt- Automatic Prompt Optimization Platform

## Overview

This repository contains **PromptOpt**, an Automatic Prompt Optimization (APO) platform that iteratively improves LLM prompts without changing the underlying model. Submit a base prompt, choose how to evaluate it, and the system runs a generate → score → converge → export loop, producing a measurably better prompt at the end.

The project is inspired by real-world APO research and tooling such as **ProTeGi (Microsoft)**, **APE (Automatic Prompt Engineer)**, **Evidently AI**, and **AutoPDL**, adapted into a self-contained full-stack web app for experimentation and learning.

## Key Concepts

PromptOpt is built around an iterative optimization loop with two evaluation modes:

- **Job Initializer** — Validates the base prompt, dataset/criteria, and run configuration before kicking off optimization
- **Variant Generator** — Uses an LLM to generate multiple rewritten candidate prompts each iteration
- **Dataset Mode** — Scores variants against labeled examples using accuracy or LLM-as-judge
- **Datasetless Mode** — Scores variants against plain-English criteria rules when no labeled data is available
- **Score Collector + Convergence Check** — Aggregates scores per iteration and decides whether to stop early or continue
- **Ranker + Best Prompt Store** — Picks the winning prompt across all iterations and persists it
- **Export API** — Lets you pull the optimized prompt out into your own application

**Two evaluation modes:**
- **Dataset mode** — labeled examples scored by accuracy or LLM-judge
- **Datasetless mode** — plain-English criteria rules, no examples needed

## Inspiration & References

- **ProTeGi (Microsoft)** — Prompt Optimization with Textual Gradients
- **APE** — Automatic Prompt Engineer
- **Evidently AI** — LLM evaluation patterns
- **AutoPDL** — Automated prompt design loops
- UI reference: Weights & Biases, Braintrust, PromptLayer, Linear, Vercel

## Tech Stack

- **Frontend:** React + Vite, React Router, Recharts
- **Backend:** FastAPI + SQLite
- **LLM:** Google Gemini (`gemini-2.0-flash`) via `google-generativeai`
- **Design system:** Arctic Light — Fraunces (headings), DM Sans (body/UI), IBM Plex Mono (code/data)

## Repository Structure

Initial planning and context files in the directory:
├── flowchart-v2.svg          # Backend pipeline architecture diagram
├── prompt.md                 # Per-feature implementation prompts (14 features)
├── roadmap.md                # Build order, feature list, file structure
├── progress.md               # Session-by-session progress tracker
└── promptopt/
├── backend/              # FastAPI service
│   ├── main.py
│   ├── database.py
│   ├── requirements.txt
│   ├── core/             # Optimization pipeline modules
│   │   ├── job_initializer.py
│   │   ├── variant_generator.py
│   │   ├── scorer.py
│   │   ├── criteria_scorer.py
│   │   ├── optimizer.py
│   │   └── ranker.py
│   └── routers/
│       ├── runs.py
│       └── export.py
└── frontend/             # React + Vite app
├── package.json
├── index.html
└── src/
├── App.jsx
├── components/
├── hooks/
└── lib/

## Setup

### Prerequisites

```bash
# Backend dependencies
cd promptopt/backend
pip install -r requirements.txt

# Frontend dependencies
cd ../frontend
npm install
```

### API Keys

PromptOpt uses **Google Gemini** as the LLM for variant generation and scoring. You need a Gemini API key.

- Gemini API Key: [Google AI Studio](https://aistudio.google.com/app/apikey)

Create a `.env` file in `promptopt/backend/`:

```bash
GOOGLE_API_KEY=your_gemini_api_key_here
```

### Running the Application

```bash
# Start the backend (from promptopt/backend)
uvicorn main:app --reload
# Backend runs at http://localhost:8000  · API docs at http://localhost:8000/docs

# In another terminal, start the frontend (from promptopt/frontend)
npm run dev
# Frontend runs at http://localhost:5173
```

The frontend talks to the backend over CORS at `http://localhost:5173 → http://localhost:8000`.

## How It Works
User Input → Job Initializer → Variant Generator
├── [with dataset]  → Dataset Scorer → Accuracy Evaluator
└── [no dataset]    → Criteria Scorer
└── Score Collector → Converged?
├── No  → back to Variant Generator
└── Yes → Ranker → Best Prompt Store → Export API → UI

1. Submit a base prompt and pick a mode (dataset or datasetless)
2. Configure max iterations, variants per iteration, and an early-stop threshold
3. The optimizer generates N candidate prompts per iteration, scores each one, and tracks the best
4. When the score plateaus or hits the threshold, the run stops and the winning prompt is saved
5. Export the optimized prompt via the API or copy it from the UI
