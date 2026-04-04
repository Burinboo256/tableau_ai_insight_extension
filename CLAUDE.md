# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Deployment

Push to `main` branch on GitHub → GitHub Actions (`.github/workflows/deploy.yml`) automatically deploys to GitHub Pages.

Live URL: `https://burinboo256.github.io/tableau_ai_insight_extension/`

There is no build step — `index.html` is served as-is.

## Architecture

This is a **single-file Tableau Dashboard Extension** — all UI, styles, and logic live in `index.html`. There is no framework, bundler, or package manager.

**Two files matter:**
- `index.html` — entire application (HTML + CSS + JS in one file)
- `manifest.trex` — Tableau extension manifest; the `<source-location>` URL must match the deployed GitHub Pages URL

**Tableau Extensions API** is loaded from CDN:
```html
<script src="https://cdn.jsdelivr.net/npm/@tableau/extensions-api/lib/tableau.extensions.1.latest.min.js"></script>
```

## Key JS Architecture (inside `index.html`)

**State variables** (top of `<script>`):
- `tableauExt` — set after `initializeAsync()` succeeds; `null` means sandbox/mock mode
- `cachedData` — `{columns: string[], rows: object[]}` loaded from the selected worksheet
- `selectedFields` — `Set` of field names chosen by the user for AI analysis
- `currentMode` — `'kpi' | 'aggregate' | 'raw'`

**Tableau init flow:**
1. `initializeAsync()` → success: `tableauExt` set, `populateWorksheets()` fills dropdowns with real worksheet names
2. `initializeAsync()` → failure: status shows "Sandbox Mode", `populateMockWorksheets()` used instead

**Worksheet data loading** (`loadWorksheet()`):
- Tries `getSummaryDataReaderAsync()` first (API 1.10+), falls back to `getSummaryDataAsync()`
- On success: calls `processTblData()` → sets `cachedData`, renders field tags and preview table
- On error: shows error in status badge (does NOT silently fall back to mock data)

**Compare tab** (`loadCompareSheets()` → `fetchWorksheetData()`):
- Also uses the same dual-API approach via `fetchWorksheetData(wsName)`
- Falls back to `getMockData()` only when `tableauExt` is null

**AI analysis** calls OpenRouter API with the selected fields from `cachedData`, using model/temperature/language from settings panel. Settings are persisted to `localStorage`.

## Tableau Extension Setup (for testing)

1. Download `manifest.trex` from this repo
2. In Tableau Desktop: drag an **Extension** object onto a Dashboard → select `manifest.trex`
3. Click **Allow** on the trust dialog
4. Status badge should show "เชื่อมต่อแล้ว" (not "Sandbox Mode")

The extension requires Tableau Desktop 2018.2+ and the manifest permission `<permission>full data</permission>` for `getSummaryDataReaderAsync` to work.
