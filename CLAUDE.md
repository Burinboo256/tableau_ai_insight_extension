# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development

No build step. Serve `index.html` directly for local testing:

```bash
python3 -m http.server 8000
# open http://localhost:8000
```

`manifest.trex` is in `.gitignore` — each developer keeps their own local copy with their own `<source-location>` URL. Do not commit it.

## Deployment

Push to `main` branch → GitHub Actions (`.github/workflows/deploy.yml`) deploys to GitHub Pages automatically.

Live URL: `https://burinboo256.github.io/tableau_ai_insight_extension/`

## Architecture

Single-file Tableau Dashboard Extension — all HTML, CSS, and JS live in `index.html`. No framework, bundler, or package manager.

**Tableau Extensions API** loaded from CDN:
```html
<script src="https://cdn.jsdelivr.net/gh/tableau/extensions-api@main/lib/tableau.extensions.1.latest.min.js"></script>
```

## Key State (top of `<script>` in `index.html`)

| Variable | Type | Purpose |
|---|---|---|
| `tableauExt` | object\|null | Set after `initializeAsync()` succeeds; `null` = sandbox/mock mode |
| `cachedData` | `{columns, rows}` | Data loaded from the selected worksheet |
| `selectedFields` | `Set<string>` | Field names the user has toggled on for AI analysis |
| `currentMode` | `'kpi'|'aggregate'|'raw'` | Controls how many rows and which slice is sent to AI |
| `lastResult` | object\|null | Last parsed AI JSON response (used for export) |
| `lastCompareResult` | object\|null | Last parsed compare AI JSON response |
| `history` | array | Persisted to `localStorage` as `ai_insight_history` |

## Data Flow

**Tableau init:**
1. `initializeAsync()` success → `tableauExt` set, `populateWorksheets()` fills all three worksheet dropdowns
2. `initializeAsync()` failure → Sandbox Mode, `populateMockWorksheets()` used

**Worksheet loading** (`loadWorksheet()`):
- Tries `getSummaryDataReaderAsync()` (API 1.10+), falls back to `getSummaryDataAsync()`
- On success → `processTblData()` → sets `cachedData`, renders field tags and preview
- On error → shows error in status badge; does NOT silently fall back to mock data

**Compare tab** (`loadCompareSheets()` → `fetchWorksheetData(wsName)`):
- Same dual-API pattern as above
- Falls back to `getMockData()` only when `tableauExt` is null

## AI Call Architecture

All AI requests go through `callAI(apiKey, model, systemPrompt, userPrompt, maxTokens, temperature)`.

- **OpenRouter / OpenAI / Anthropic / Google**: POST to their respective chat completion endpoints with an OpenAI-compatible body
- **Ollama**: POST to `http://localhost:11434/api/chat`
- All providers expect the response parsed by `parseJSON()` which strips markdown fences before `JSON.parse()`

**Analyze flow** (`runAnalysis()`):
1. `buildDataString()` slices `cachedData` by mode and selected fields into a pipe-delimited string
2. Optional PII masking applied to the string before sending
3. Full prompt assembled with language instruction, optional data context from `runUnderstand()`, optional word limit instruction, and the data string
4. AI returns JSON: `{insight, suggestions[], followup_questions[], confidence, caveat}`
5. Result rendered into three tabs; saved to `history`

**Schema understanding** (`runUnderstand()`):
- Sends column names + 5 sample rows
- AI returns JSON: `{dataset_type, domain, key_metrics[], dimensions[], data_quality_notes, analysis_context, suggested_questions[]}`
- `suggested_questions` are rendered as dynamic prompt chips in the Analyze tab

**Settings** are read from `localStorage` on load via `loadSettingsFromStorage()` and written by `saveSettings()`.

## Tableau Extension Setup (for testing)

1. Create a local `manifest.trex` with `<source-location>` pointing to `http://localhost:8000/index.html`
2. In Tableau Desktop: drag an **Extension** object onto a Dashboard → select `manifest.trex`
3. Click **Allow** on the trust dialog
4. Status badge should show "เชื่อมต่อแล้ว" (not "Sandbox Mode")

Requires Tableau Desktop 2018.2+. The `<permission>full data</permission>` in the manifest is required for `getSummaryDataReaderAsync` to work.
