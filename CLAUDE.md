# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Trading Dominion Tools is a lightweight, browser-based financial analysis toolkit deployed as static HTML files on GitHub Pages. The project consists of three self-contained HTML applications with no build process, backend, or external dependencies beyond CDN-hosted libraries.

**Repository:** https://github.com/TradingDominion/tools
**Deployment:** GitHub Pages (static files)
**Tech Stack:** Pure HTML/CSS/JavaScript with CDN libraries

## Project Structure

```
tools/
├── index.html                  # Landing page with navigation
├── portfolio-analyzer.html     # Main portfolio analysis tool (v7.26)
└── pv-converter.html          # PortfolioVisualizer data converter
```

Each HTML file is completely self-contained with inline CSS and JavaScript. There are no separate asset directories, configuration files, or build artifacts.

## Core Applications

### Portfolio Analyzer ([portfolio-analyzer.html](portfolio-analyzer.html))

A comprehensive portfolio performance analysis tool (~1,600 lines) that processes CSV files client-side and generates interactive visualizations.

**Key Features:**
- Multi-file CSV upload and merging with automatic date/price column detection
- Portfolio composition ("combo") builder with weighted allocation
- Financial metrics: CAGR, Sharpe ratio, Sortino ratio, Volatility, Profit factor, Ulcer Index, Calmar ratio
- Interactive charts: cumulative returns (simple & log scale), drawdowns, correlation matrices
- Date range filtering with quick range buttons (10Y, 5Y, 1Y, YTD, All)
- CSV/XLS export capabilities
- LocalStorage persistence for user preferences (combo weights, transposed views)

**Core Functions:**
- `handleUpload()` - CSV processing pipeline with PapaParse
- `computeReturns()` - Calculates simple and log returns from price data
- `computeStats()` - Calculates all financial metrics for the statistics table
- `drawCharts()` - Renders all Plotly.js visualizations
- `corrMatrix(negOnly)` - Computes correlation matrices
- `injectComboIntoMerged()` - Applies portfolio weights to create combined series
- `withinRange()` - Filters data based on date range selection

**Data Flow:**
```
CSV Upload → Parse → Auto-detect date/price columns → Merge timeseries →
Calculate returns (simple & log) → Compute statistics → Render charts & tables
```

**State Management:**
Global variables maintain application state:
- `rawSeries` - Raw price/return data by ticker
- `merged` - Unified timeseries across all tickers
- `retSimple`, `retLog` - Simple and log return calculations
- `equity` - Equity curves for each ticker
- `drawdowns` - Drawdown series
- `labels` - Active ticker list
- `comboWeights` - Portfolio allocation weights (persisted to localStorage)
- `comboEnabled` - Whether combo portfolio is active

### PV Converter ([pv-converter.html](pv-converter.html))

Converts PortfolioVisualizer monthly returns data to CSV format (~570 lines).

**Key Features:**
- Text paste input or Excel file upload
- Flexible date parsing (multiple formats)
- Return percentage conversion to decimal
- CSV generation with last business day of month dates

**Core Functions:**
- `convertData()` - Processes pasted text data
- `handleExcelFile()` - Processes Excel uploads
- `parseFlexibleDate()` - Multi-format date parser
- `generateAndDownloadCSV()` - Creates downloadable CSV

## Dependencies (CDN)

All dependencies are loaded via CDN links (no package.json or node_modules):

**Portfolio Analyzer:**
- PapaParse v5.4.1 - CSV parsing
- Day.js v1 (with customParseFormat, advancedFormat, minMax plugins) - Date manipulation
- Plotly.js v2.35.2 - Interactive charts
- XLSX v0.18.5 - Excel file I/O
- Google Fonts (Roboto, Montserrat)

**PV Converter:**
- XLSX v0.18.5 - Excel file processing

## Development Workflow

### Local Testing

Simply open HTML files in a browser:
```bash
# Windows
start index.html

# macOS
open index.html

# Linux
xdg-open index.html
```

Or use a local web server for more realistic testing:
```bash
# Python 3
python -m http.server 8000

# Node.js (if http-server is installed globally)
npx http-server -p 8000

# Then visit: http://localhost:8000
```

### Available MCP Servers

This project has access to the following MCP (Model Context Protocol) servers for enhanced development capabilities:

#### Playwright
Browser automation and testing capabilities for validating the HTML applications:
- Navigate to local or deployed URLs
- Take screenshots and snapshots of the applications
- Test interactive features (file uploads, button clicks, form interactions)
- Verify responsive design across different viewport sizes
- Monitor console logs and network requests
- Validate chart rendering and data visualization

**Common use cases:**
- Visual regression testing after UI changes
- Automated testing of CSV upload and processing flows
- Verification of chart interactions and responsiveness
- Cross-browser compatibility validation

#### Context7
Access to up-to-date library documentation for the CDN dependencies:
- PapaParse - CSV parsing syntax and options
- Day.js - Date manipulation methods and plugins
- Plotly.js - Chart types, layout options, and interactivity
- XLSX - Excel file reading/writing APIs

**Common use cases:**
- Look up Plotly chart customization options
- Verify Day.js date formatting patterns
- Check PapaParse configuration for edge cases
- Reference XLSX methods for export functionality

### Making Changes

Since there's no build process, edit HTML files directly:

1. **Styling**: Modify CSS within `<style>` tags
   - Dark theme uses CSS custom properties defined in `:root`
   - Colors: `--bg`, `--fg`, `--accent`, `--green`, `--red`, etc.
   - Responsive breakpoints at 1050px

2. **Functionality**: Edit JavaScript within `<script>` tags
   - All code is vanilla ES6+ JavaScript
   - No transpilation or bundling
   - Browser APIs: FileReader, localStorage, Blob

3. **Testing**: Refresh browser to see changes immediately

### Version Tracking

Version numbers are manually updated in the HTML title tags:
- Current: Portfolio Analyzer v7.26
- Update version in `<title>` when making significant changes

### Deployment

The repository is configured for GitHub Pages. To deploy:

```bash
git add .
git commit -m "Description of changes"
git push origin main
```

Changes will be live at the GitHub Pages URL automatically.

## Architecture Principles

1. **Self-Contained**: Each HTML file includes all CSS, JavaScript, and markup
2. **No Build Process**: Direct file editing with immediate browser refresh
3. **Client-Side Only**: All processing happens in the browser, no backend
4. **Stateless**: Data exists only in memory during session (except localStorage preferences)
5. **Progressive Enhancement**: Works without JavaScript for basic navigation
6. **Dark Theme**: All pages use dark color schemes optimized for finance professionals

## Code Patterns

### Event-Driven Architecture
User interactions trigger event handlers that update global state and re-render UI:
```javascript
file.addEventListener('change', handleUpload);
// → Parse CSV → Update merged data → Recompute stats → Redraw charts
```

### Data Processing Pipeline
1. File upload → PapaParse
2. Field detection → Auto-identify date/price columns via `pickPriceField()`
3. Normalization → Convert to unified timeseries
4. Calculation → Returns, statistics, correlations
5. Visualization → Plotly charts and HTML tables

### State Updates
After modifying global state, always call the appropriate update functions:
```javascript
// After changing date range:
updateAll();  // Recomputes stats and redraws all charts

// After changing combo weights:
recomputeCombo();  // Recalculates portfolio and updates display

// After upload:
updateDownloadState();  // Enables/disables export buttons
```

## Common Modifications

### Adding New Financial Metrics

Edit the `computeStats()` function in [portfolio-analyzer.html](portfolio-analyzer.html):
1. Add calculation logic (~line 411)
2. Update `renderStatsTable()` to display new metric (~line 512)
3. Consider adding to export functionality if needed

### Customizing Charts

Modify `drawCharts()` function (~line 446):
- Plotly layout options control appearance
- Traces define data series and styles
- Responsive sizing via `autosize: true`

### Changing Color Scheme

Update CSS custom properties in `:root`:
```css
:root {
  --bg: #0b0f14;      /* Main background */
  --fg: #e5edf5;      /* Text color */
  --accent: #5ac8fa;  /* Primary accent */
  /* ... etc */
}
```

## Data Formats

### Portfolio Analyzer CSV Input
Expected format (auto-detected):
- Date column: Various formats supported (YYYY-MM-DD, MM/DD/YYYY, etc.)
- Price/Return column: Numeric values
- Multiple files merged by date intersection

### PV Converter Output
Generated CSV format:
```csv
Date,Return
2023-01-31,0.0234
2023-02-28,-0.0156
```

## Important Notes

- **No npm/yarn**: Do not create package.json or use Node.js tooling
- **No transpilation**: Write browser-compatible ES6+ only
- **No modules**: All code in global scope or IIFEs
- **Version in title**: Update version number in `<title>` for significant changes
- **Testing**: Always test in multiple browsers (Chrome, Firefox, Safari, Edge)
- **Mobile**: Responsive design tested down to 320px width
- **Performance**: Large datasets (10,000+ rows) may slow down; Plotly handles up to ~50k points well

## Git Workflow

Current branch: `main`
Remote: https://github.com/TradingDominion/tools

Standard workflow:
```bash
git status                           # Check current state
git add portfolio-analyzer.html      # Stage changes
git commit -m "Description"          # Commit with clear message
git push origin main                 # Deploy to GitHub Pages
```

Recent commit style examples:
- "Change rows from newest first to oldest first"
- "Update index.html"
- "Create pv-converter.html"
