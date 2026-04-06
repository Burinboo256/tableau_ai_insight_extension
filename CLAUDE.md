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

## Tab Structure

5 tabs: `วิเคราะห์` → `ผลลัพธ์` → `เปรียบเทียบ` → `ตั้งค่า` → `ประวัติ`

- **วิเคราะห์** — worksheet checkboxes, mode, field selector, prompt, analyze button
- **ผลลัพธ์** — always-visible result panel with 3 sub-tabs (Insight / Suggestions / Follow-up) + export buttons; auto-switched to after `runAnalysis()` completes
- **เปรียบเทียบ** — side-by-side two-worksheet compare
- **ตั้งค่า** — AI provider, model, API key, system prompt, PII masking, log folder
- **ประวัติ** — searchable/filterable history with per-entry and bulk export

## Key State (top of `<script>` in `index.html`)

| Variable | Type | Purpose |
|---|---|---|
| `tableauExt` | object\|null | Set after `initializeAsync()` succeeds; `null` = sandbox/mock mode |
| `cachedData` | `{name, columns, rows}[]` | Array of loaded worksheets — multi-sheet since refactor |
| `selectedFields` | `Set<string>` | Union of field names toggled on across all loaded sheets |
| `currentMode` | `'kpi'\|'aggregate'\|'raw'` | Controls row slice sent to AI |
| `lastResult` | object\|null | Last parsed AI JSON (used by all export functions) |
| `lastCompareResult` | object\|null | Last parsed compare AI JSON |
| `logDirHandle` | FileSystemDirectoryHandle\|null | Folder handle for JSONL log file; persisted via IndexedDB |
| `history` | array | Persisted to `localStorage` as `ai_insight_history` |

## Data Flow

**Tableau init:**
1. `initializeAsync()` success → `tableauExt` set, `populateWorksheets()` renders checkbox list + fills compare selects
2. `initializeAsync()` failure → Sandbox Mode, `populateMockWorksheets()` used

**Multi-worksheet loading** (`loadWorksheet()`):
- Reads all checked `.ws-check` checkboxes via `getCheckedWS()`
- Loads each worksheet in sequence using `getSummaryDataReaderAsync()` (API 1.10+, `maxRows:0` = all rows) or `getSummaryDataAsync({maxRows:0})` fallback
- Appends each as `{name, columns, rows}` into `cachedData` array
- Field pool shows union of all columns across sheets; `buildDataString()` outputs labeled sections: `### SheetName (N rows)\ncol1 | col2\n...`
- On error → shows error badge; does NOT fall back to mock data

**Compare tab** (`loadCompareSheets()` → `fetchWorksheetData(wsName)`):
- Same dual-API pattern; falls back to `getMockData()` only when `tableauExt` is null

## AI Call Architecture

All AI requests go through `callAI(apiKey, model, systemPrompt, userPrompt, maxTokens, temperature)`.

- **OpenRouter / OpenAI / Anthropic / Google**: OpenAI-compatible chat completion endpoints
- **Ollama**: `POST http://localhost:11434/api/chat`

**Analyze flow** (`runAnalysis()`):
1. `buildDataString()` — labeled per-sheet sections, sliced by mode + selected fields
2. PII masking applied if configured
3. Prompt instructs AI to return all 3 sections (insight ≥ 5 points, suggestions ≥ 3, followup ≥ 3)
4. AI must return JSON: `{insight, suggestions[], followup_questions[], confidence, caveat}`
5. After render, `writeLogEntry()` writes to JSONL log file if folder is set
6. Auto-switches to `panel-result` tab

**Follow-up Re-gen** (`regenFollowup()`):
- Button in Follow-up tab header
- Sends a second AI call with `lastResult.insight` + `lastResult.suggestions` as context
- Requests ≥ 5 follow-up questions; updates `lastResult.followup_questions` on success

**Schema understanding** (`runUnderstand()`):
- Sends all sheets' column names + 3 sample rows each
- AI returns `{dataset_type, domain, key_metrics[], dimensions[], data_quality_notes, analysis_context, suggested_questions[]}`
- `suggested_questions` rendered as dynamic prompt chips

## JSON Parsing Pipeline (`parseJSON()`)

LLM responses are often malformed. Pipeline tries in order:
1. Direct `JSON.parse`
2. `escapeNewlinesInStrings()` — escapes actual `\n`/`\t` inside string values (common Ollama issue)
3. Extract first `{...}` block, then parse
4. `repairJSON()` — auto-closes unclosed `"`, `]`, `}` from truncated responses
5. Repair on extracted block
6. `extractFieldsWithRegex()` — regex-pulls `insight`, `confidence`, `caveat`, `suggestions` individually when full parse fails

If all steps fail, `runAnalysis()` strips the JSON wrapper and renders the insight text directly via `markdownToHtml()` with a fallback warning badge.

## Rendering

`markdownToHtml(text)` converts AI output to styled HTML:
- `## Header` → bold accent-colored div
- `• / - / *` bullet lines → indented divs
- Blank lines → spacer divs

`renderInsight()` normalises the `insight` field: if AI returns an object (e.g. per-sheet map) it joins entries as `## key\nvalue` before rendering.

## Export

`downloadFile(content, filename, mime)` uses `dispatchEvent(new MouseEvent('click'))` — required for Tableau Desktop WebView compat. No `window.open` fallback (blocked in iframe).

Export functions (`exportAs`, `exportAll`, `exportCompare`) read worksheet name from `cachedData.map(s=>s.name).join(', ')` — **not** from any `wsSelect` element (removed in multi-sheet refactor).

History exports: `exportHistory('json'|'csv')` for all entries; `exportHistoryItem(i, 'json'|'html')` and `copyHistoryItem(i)` per entry.

## Log File (File System Access API)

`pickLogFolder()` → `window.showDirectoryPicker()` → handle stored in IndexedDB via `_saveHandleToIDB()`.
`restoreLogFolder()` called on DOMContentLoaded to restore persisted handle.
`writeLogEntry(entry)` appends to `ai-insight-log-YYYY-MM.jsonl` after each analysis.

## Settings Persistence

All settings saved to `localStorage` with `ai_` prefix via `saveSettings()` / `loadSettingsFromStorage()`. Includes: `apiKey`, `modelSelect`, `systemPrompt`, `outputLang`, `maskSensitive`, `providerSelect`, `ollamaEndpoint`, `customModel`.

## Tableau Extension Setup (for testing)

1. Create a local `manifest.trex` with `<source-location>` pointing to `http://localhost:8000/index.html`
2. In Tableau Desktop: drag an **Extension** object onto a Dashboard → select `manifest.trex`
3. Click **Allow** on the trust dialog
4. Status badge should show "เชื่อมต่อแล้ว" (not "Sandbox Mode")

Requires Tableau Desktop 2018.2+. The `<permission>full data</permission>` in the manifest is required for `getSummaryDataReaderAsync` to work.
