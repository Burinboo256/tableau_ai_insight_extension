# AI Insight Analyzer — Tableau Dashboard Extension

Analyze Tableau worksheet data with AI to generate actionable insights, compare sheets, and export reports.

**Live URL:** https://burinboo256.github.io/tableau_ai_insight_extension/

---

## Features

### Analyze Tab
- **Worksheet selection** — connects to Tableau Dashboard worksheets or falls back to sandbox/mock mode automatically
- **Auto-refresh** — re-loads data and re-runs analysis when Tableau filters change, marks are selected, or both
- **3 data modes**
  - `KPI / Summary` — top 30 rows, focused on headline metrics
  - `Aggregate` — top 50 rows, grouped/aggregated data
  - `Raw Data` — configurable row limit (default 50) with Top N or Random sampling
- **Field selector** — toggle individual columns on/off; select all or clear with one click
- **Data preview table** — shows first 5 rows with optional anomaly highlighting (values beyond ±2σ flagged in red/green)
- **Token estimator** — live estimate of prompt size before sending to AI
- **AI schema understanding** — one-click scan that lets the AI read the schema and sample data, then auto-generates context-aware prompt suggestions
- **Prompt templates** — 4 built-in templates (Summary, Trend, Action, Compare); AI-generated dynamic suggestions appear after schema scan
- **Word limit control** — optional cap on response length (100 / 250 / 500 / 750 / unlimited or custom), scoped to all sections, Insight only, or Suggestions only
- **Temperature slider** — controls AI creativity from Precise (0) to Creative (1.0), default 0.4
- **Structured AI output** with 3 result tabs:
  - `Insight` — main analysis with confidence level and caveats
  - `Suggestions` — prioritized action items (High / Medium / Low) with reason and linked metric
  - `Follow-up` — questions to investigate next, each with rationale and data needed; click any to send it as a new prompt

### Compare Tab
- **Side-by-side worksheet comparison** — load any two worksheets simultaneously
- **Mini preview** of both sheets before analysis
- **3 comparison prompt templates** — KPI comparison, performance evaluation, trend differences
- **Structured comparison output** — summary, key differences table (metric / Sheet A / Sheet B / interpretation), winner assessment, recommendations
- **Export** comparison result as Copy Text, HTML report, or JSON

### Settings Tab
- **AI provider selection**
  - OpenRouter (multi-model proxy, default)
  - OpenAI (direct)
  - Anthropic Claude (direct)
  - Google Gemini (direct)
  - Ollama (local, no API key required)
- **Model selection** — pre-configured list covering Claude Sonnet 4.5, Claude 3.5 Sonnet, Claude 3 Haiku, GPT-4o, GPT-4o Mini, Gemini Pro 1.5, Gemini Flash 1.5, Llama 3.1 70B, Llama 3.1 8B; custom model name input for Ollama or any other model
- **Ollama endpoint** — configurable local URL (default `http://localhost:11434`)
- **Output language** — Thai only, English only, or Thai + English bilingual
- **System prompt** — fully editable AI persona and instruction; defaults to a Thai business/hospital data analyst role
- **Sensitive data masking** — None / mask Patient Name & HN / mask all PII before sending to AI
- All settings persisted to `localStorage`

### History Tab
- Full log of every analysis and comparison run
- **Search** by worksheet name or prompt text
- **Filter** by mode (KPI / Aggregate / Raw / Compare)
- **Clear all** history
- Each entry is collapsible and shows time, worksheet, mode, and result

### Export (per result section)
| Format | Available for |
|--------|--------------|
| Copy Text | Insight, Suggestions, Follow-up, Compare |
| Copy Markdown | Insight, Suggestions, Follow-up |
| Download HTML report | Insight, Suggestions, Follow-up, Compare, Full report |
| Download JSON | Insight, Suggestions, Follow-up, Compare, Full report |

---

## Setup in Tableau Desktop

1. Download `manifest.trex` from this repo
2. Open Tableau Desktop → open the dashboard you want to extend
3. Drag an **Extension** object onto the dashboard from the left panel
4. Click **"My Extensions"** → select `manifest.trex`
5. Click **Allow** on the trust dialog
6. Go to the Settings tab and enter your API key

> Get a free API key at [openrouter.ai/keys](https://openrouter.ai/keys)

**Requirements:** Tableau Desktop 2018.2+. The `full data` permission in the manifest is required for `getSummaryDataReaderAsync` to work.

---

## Local Development / Testing

No build step needed. Serve `index.html` directly:

```bash
python3 -m http.server 8000
```

Then set `manifest.trex` `<source-location>` to `http://localhost:8000/index.html`. The extension also works in sandbox mode (without Tableau) using built-in mock data.

---

## Deployment

Push to the `main` branch → GitHub Actions (`.github/workflows/deploy.yml`) deploys automatically to GitHub Pages.

---

## Files

| File | Description |
|------|-------------|
| `index.html` | Entire application — HTML, CSS, and JS in one file |
| `manifest.trex` | Tableau extension manifest (extension ID, permissions, source URL) |

---

## Notes

- API key is stored in browser `localStorage` — suitable for personal/testing use only
- Do not use real patient data until deployed on a private server with proper access controls
- The `<source-location>` URL in `manifest.trex` must match the deployed URL; update it before sharing
