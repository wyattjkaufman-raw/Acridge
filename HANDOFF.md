# ACRIDGE — Complete Project Handoff

Date: 2026-06-15 | Version: 20260614-ui6

## 1. Project Overview

Acridge is an AI-powered real estate intelligence SPA. It is a pure vanilla JavaScript application, no framework, no build step, no bundler, no TypeScript. It renders entirely in the browser from a single app.js file.

- Path: /Users/wyattkaufman/aceridge/
- Backup copy: /Users/wyattkaufman/Downloads/aceridge-copy/
- Preview server: Python HTTP server on port 7821 (launch via .claude/launch.json to serve_aceridge.py)

Architecture in one sentence: A global mutable state object drives a single render() function that regenerates all HTML on every state change. Event delegation via data-action attributes handles all user interaction.

## 2. File Structure

Directory layout:

- aceridge/
  - index.html (28 lines) - entry, cache-busting params
    - app.js (9,034 lines / 403KB) - entire application
      - src/
          - styles.css (7,555 lines / 149KB) - all CSS
              - datasetRegistry.js - data registry module
                  - propertySchema.js - property schema definitions
                      - legalGuardrails.js - legal/compliance guardrails
                          - apiConnectors.js - API connector stubs (all mocked)

                          Cache busting: Both styles.css and app.js are loaded with a version query parameter (v=20260614-ui6). Bump this on every deploy by updating both the link and script tags in index.html.

                          ## 3. Current Product State - All Tabs

                          ### Tab 1: Dashboard

                          - Hero: Unified headline "Real Estate Intelligence. Built on Solid Ground." plus subheadline (same across all 5 personas, no longer persona-specific)
                          - Portfolio snapshot card: Canvas-based mini portfolio chart (same component as Portfolio page); range tabs 1D/1W/1M/3M/1Y; hover scrubbing with value overlay in top-left; 2x2 metric mini-grid below chart; "View full portfolio" CTA
                          - Role-specific brief: Short paragraph from the active persona's brief property
                          - Market intelligence feed: Mocked news/signal items

                          ### Tab 2: Analyze

                          - Property input to full analysis panel
                          - Blue text fixed: scenario-grid, rent-band, verdict-detail-grid, offer-grid, and risk-grid all have correct dark-on-light contrast (no more light blue on white backgrounds)
                          - Verdict badge, cash flow table, risk grid, offer strategy section

                          ### Tab 3: Map

                          - Leaflet map with role-specific layers
                          - Neighborhood Intel sidebar: Trend chart fully rewritten - 400x180 viewBox, green line/area fill, correct proportions, 260px container height, year labels, non-scaling stroke
                          - Neighborhood scoring grid

                          ### Tab 4: Portfolio

                          Layout (stacked, not two-column):

                          - Full-width canvas chart section (portfolio-hero-chart) with value overlay top-left
                          - 4-column metrics row (portfolio-metrics-row)
                          - Intelligence/insights row (portfolio-intelligence-row)
                          - Holdings list with property detail drawer
                          - Range tabs: 1D / 1W / 1M / 3M / YTD / 1Y / ALL
                          - Hover scrubbing on chart updates value/delta/date overlays

                          ### Tab 5: Intel

                          - Market intelligence feed (mocked signals, trends, headlines)

                          ### Tab 6: Scenarios

                          - What-if scenario modeler (rate changes, rent changes, etc.)

                          ### Tab 7: Settings / Role Switcher

                          - 5 persona cards: Buyer, Investor, Developer, Agent, Corporate
                          - Switching persona updates all tabs immediately

                          ## 4. Personas (5 roles)

                          - Buyer: Home Buyer - Dashboard line: "Seattle - Buyer's Market - Rates at 6.8%" - Verdict: Fair Value
                          - Investor: Investor - Dashboard line: "Seattle - Tight Rental Market - Cap rates at 5.1%" - Verdict: Strong Investment
                          - Developer: Developer - Dashboard line: "Seattle - Permit Volume +7.4% - Zoning watch active" - Verdict: Good Deal
                          - Agent: Agent / Broker - Dashboard line: "Seattle - DOM 24 - Price cuts +5.6%" - Verdict: Fair Value
                          - Corporate: Corporate - Dashboard line: "US West - Allocation watch - HPI +3.9%" - Verdict: Good Deal

                          ## 5. Design System

                          CSS custom properties (defined around line 4896 in styles.css):

                          - ink: #151515 - Primary text, headers
                          - forest: #1f3a2e - Dark green brand anchor
                          - forest-2: #2f5a46 - Lighter green accent
                          - surface: #fffdf9 - Card/panel backgrounds
                          - paper: #f7f4ef - Page background
                          - border: #e3ded6 - Dividers, card borders
                          - muted: #696761 - Secondary text, labels
                          - graphite: #4a4845 - Mid-weight text
                          - navy: #1a2640 - Dark blue accent (use sparingly)
                          - radius: 8px - Standard border radius

                          Typography:

                          - Body/UI: Inter, ui-sans-serif, system-ui, sans-serif
                          - Value display (money, metrics): Georgia, Times New Roman, serif - intentional, critical to the premium feel
                          - Hero h1: clamp(28px, 4.2vw, 60px) with line-height 1.08
                          - Chart value overlay strong: clamp(32px, 3.4vw, 52px) (main) / clamp(22px, 2.4vw, 30px) (mini dashboard)

                          Key color rules:

                          - Never put light blue (#93c5fd) on light backgrounds - it fails WCAG contrast
                          - Green (#1f7a54) is the chart line color
                          - All chart text uses the muted variable fill
                          - Georgia serif for all dollar values and key metrics

                          ## 6. Canvas Portfolio Chart - Architecture

                          Both charts (Portfolio page + Dashboard mini) use the same function createPortfolioChart(canvas, series, range, opts).

                          opts parameter includes:

                          - mini (boolean): true = mini margins (top:60), false = full (top:100)
                          - valueEl (element): element to update with current value
                          - deltaEl (element): element to update with delta
                          - dateEl (element or null): element to update with date (null for mini)

                          Controller pattern:

                          - createPortfolioChart() returns a controller with a destroy() method
                          - Controllers stored in module-scope vars: portfolioChartController, dashPortfolioChartController
                          - disposePortfolioChart() destroys both before re-render
                          - syncPortfolioChart() wires up the Portfolio page chart
                          - syncDashPortfolioChart() wires up the Dashboard mini chart
                          - Both are called at end of render(), after syncLeafletMap()

                          Chart shell CSS notes:

                          - .portfolio-chart-shell uses position relative (the value overlay uses position absolute), height min(440px, 54vh), min-height 280px
                          - .snap-chart-shell uses position relative, height min(180px, 26vh), min-height 120px
                          - .chart-value-overlay is positioned absolute, top 14px, left 18px, z-index 2, pointer-events none

                          Critical: The chart shell must have an explicit height (not just min-height) for canvas height:100% to resolve to a non-zero pixel value. If the canvas goes blank, check that the shell has a real computed height.

                          ## 7. CSS Architecture

                          The CSS file is append-only layers - later rules win at same specificity:

                          - Lines 1-4891: Original dark theme base
                          - Lines 4895-6273: Premium light redesign (main theme)
                          - Lines 6273-6899: Portfolio dark overrides
                          - Lines 6900-7135: Portfolio light overrides
                          - Lines 7136-7219: Dead CSS - the .portfolio-two-col block (layout replaced, can be deleted)
                          - Lines 7220-7349: Blue text contrast fixes
                          - Lines 7350-7555: This session's additions (chart, hero, trend chart, map)

                          Specificity trap: The dark-theme base (lines 1-4891) still exists and creates baseline rules. Light-theme overrides must be at equal or higher specificity to win. When adding new styles, append at the end of the file, never insert into the dark-theme block.

                          Key scoping rules to remember:

                          - ".drawer-chart-grid .chart-box svg" sets height to 90px and must stay scoped to .drawer-chart-grid, otherwise it collapses ALL .chart-box SVGs including the Map trend chart
                          - ".chart-box.medium" sets height to 260px and must override the default (was 190px); this is the Map panel trend chart container

                          ## 8. Key Function Map (app.js)

                          Data layer (top of file):

                          - portfolioSeries (around line 1): Legacy 12-point monthly array (used in drawer sparklines)
                          - marketTrend (around line 92): 6-point array of year/price pairs for the Map trend chart
                          - roles (around line 394): 5 persona objects
                          - tabs (around line 477): Tab definitions
                          - neighborhoods (around line 1006): About 25 Seattle neighborhoods with scores/metrics
                          - state (around line 2131): All mutable app state

                          State fields (key ones): activeTab, roleId, portfolioRange (Portfolio page chart range), dashPortfolioRange (Dashboard mini chart range, added this session), portfolioSort, portfolioDrawerPropertyId, mapNeighborhood, plus many more.

                          Render pipeline:

                          - render() (around line 8208): Main render - updates the root element, then calls sync functions
                          - syncLeafletMap() (around line 8290): Post-render Leaflet initialization
                          - syncPortfolioChart() (around line 6450): Wires portfolio page canvas chart
                          - syncDashPortfolioChart() (around line 6670): Wires dashboard mini canvas chart
                          - disposePortfolioChart() (around line 6657): Destroys both chart controllers

                          Tab render functions: dashboard() for Dashboard, analyze() for Analyze, mapTab() for Map, portfolio() for Portfolio, intel() for Intel, scenarios() for Scenarios.

                          Portfolio sub-renderers:

                          - renderPortfolioHeroChart() - full-width chart section
                          - renderPortfolioMetricsRow() - 4-col metrics grid
                          - renderPortfolioIntelligenceRow() - intelligence/insights feed
                          - renderPortfolioHoldingsList() - holdings table
                          - renderPropertyDetailDrawer() - slide-in property detail
                          - renderPortfolioEmptyState() - empty state for no properties
                          - renderPortfolioMetricsStrip() - dead code, defined but no longer called

                          Chart utilities:

                          - createPortfolioChart(canvas, series, range, opts) - shared canvas chart factory
                          - currentPortfolioSeries(range) - returns time-sliced series for range
                          - portfolioRangeDelta(series) - computes value/pct/positive for a series
                          - formatPortfolioPointDate(date, range) - formats hover date string
                          - trendChart() - SVG trend chart for Map panel (400x180 viewBox)

                          ## 9. Event Delegation Pattern

                          All events go through a single document click listener that reads data-action attributes.

                          Actions relevant to charts:

                          - data-action="portfolio-range": sets state.portfolioRange to the target's data-range and calls render()
                          - data-action="dash-portfolio-range": sets state.dashPortfolioRange to the target's data-range and calls render()

                          ## 10. Known Issues

                          Minor / low priority:

                          - Dead CSS block (lines 7136-7219): .portfolio-two-col styles are fully orphaned. Safe to delete but harmless.
                          - Dead JS: renderPortfolioMetricsStrip() is defined but never called.
                          - Dark theme base CSS still present: The original dark theme (lines 1-4891) adds complexity. The light theme relies on overriding it rather than replacing it. Long-term, a clean rewrite would be better.

                          Architecture / future work:

                          - All data is mocked: No live API connections. src/apiConnectors.js has stubs only.
                          - No auth: No login, no user identity, no persistence.
                          - No routing: URL does not update with tab changes; back button doesn't work.
                          - Single-file JS: At 9,034 lines, app.js is getting large. No module system.
                          - portfolioSeries legacy: The original 12-point monthly array is still used in drawer sparklines. The new currentPortfolioSeries() is the canonical path forward.

                          ## 11. Current Priorities (ranked by impact)

                          High:

                          - Live data connections - connect apiConnectors.js to a real property data source (most valuable feature unlock)
                          - Map improvements - add more neighborhood data, make layer toggles actually filter Leaflet markers
                          - Analyze page - make property input smarter (address autocomplete, MLS lookup)

                          Medium:

                          - Mobile responsive pass - layout mostly works but hasn't been systematically tested at under 768px
                          - Dark mode toggle - the dark theme base is already there, just needs a switcher and a class toggle
                          - Portfolio drawer - property detail drawer could be richer (charts inside it are minimal)

                          Low:

                          - CSS cleanup - remove dead .portfolio-two-col block, remove dead renderPortfolioMetricsStrip()
                          - Code splitting - break app.js into modules
                          - URL routing - push tab state to URL for bookmarkability

                          ## 12. Known Gotchas / Traps

                          Chart canvas goes blank: The chart shell needs a real computed height, not just min-height. canvas height:100% only resolves if the parent has an explicit height. Check that .portfolio-chart-shell and .snap-chart-shell have height: min(...) in addition to min-height.

                          ".chart-box svg" height:90px must stay scoped: This rule lives in the portfolio drawer section. If it ever loses its .drawer-chart-grid scope, it will flatten ALL chart boxes including the Map panel trend chart. The rule must remain ".drawer-chart-grid .chart-box svg".

                          SVG distortion with preserveAspectRatio="none": Any SVG chart using preserveAspectRatio="none" must have a viewBox aspect ratio that closely matches its container. The trend chart uses 400x180 (about 2.22:1), which matches the 260px tall, roughly 580px wide container reasonably well. Don't change the viewBox without checking the container dimensions.

                          IIFE in template literals: The dashboard portfolio snapshot uses an inline immediately-invoked function expression inside a template literal to compute snap, snapDelta, and snapLatest. This is intentional - it avoids polluting the outer dashboard() function scope. Don't "clean up" this pattern.

                          const canvas re-declaration in preview_eval: When debugging in preview_eval, re-declaring "const canvas" across multiple eval calls fails with "Identifier already declared". Wrap each eval in an IIFE.

                          Event delegation action names: data-action="dash-portfolio-range" (dashboard mini chart) vs data-action="portfolio-range" (portfolio page chart). They're different - don't conflate them.

                          ## 13. Notes for Anyone Working On This Codebase

                          Do not change:

                          - The render-on-every-state-change pattern - it's intentional, not a bug
                          - Georgia serif for value display - it's the design identity
                          - The CSS append-only architecture - don't insert into the dark theme block
                          - preserveAspectRatio="none" on the trend chart SVG - removing it will cause stretching
                          - vector-effect="non-scaling-stroke" on SVG paths - removing it makes stroke widths scale non-uniformly
                          - The IIFE pattern in the dashboard snapshot section

                          Design philosophy: Premium, data-dense, institutional - this is meant to feel like a Bloomberg/Robinhood for real estate, not a consumer app. White space is intentional - don't fill gaps with cards; use space to signal confidence. Typography hierarchy matters - Georgia for numbers, Inter for labels; mixing them is part of the system. Green is the brand signal - #1f7a54 for positive/active; never use pure saturated green. No light blue on light backgrounds - #93c5fd is strictly for dark-theme accents only.

                          Product vision: Acridge's core thesis is that real estate decisions are made on bad information. The product aggregates and interprets signals (market trends, neighborhood data, portfolio performance, deal analysis) through 5 professional lenses, giving every user the intelligence layer that institutional players have. Every feature should answer: "What should I do next, and why?"

                          Preferred patterns:

                          - Append CSS at end of styles.css - never modify existing blocks
                          - Use data-action plus event delegation for all new interactions
                          - Add new state fields to the state object (around line 2131) with sensible defaults
                          - After every significant feature change: bump cache version in index.html, commit to git

                          ## 14. Git State

                          5 commits on main (most recent first):

                          1. UI polish and chart unification (canvas charts, portfolio layout, map trend fix, hero copy)
                          2. Portfolio page redesign, blue contrast pass
                          3. Initial dark-to-light theme redesign
                          4. Core app implementation
                          5. Project init

                          ## 15. Starting a New Session

                          Start the preview server with: python3 /Users/wyattkaufman/.claude/serve_aceridge.py which serves at http://localhost:7821. Alternatively, use the launch.json config in .claude/launch.json.

                          First thing to check in a new session: Open the preview, switch through all 5 personas, and click through all 7 tabs to confirm nothing is broken before making changes.

                          Before any CSS change: Know which layer you're in. Always append to the end of styles.css. Check the line count first (wc -l src/styles.css) to understand the current state.

                          Before any JS change: Search for the function you're modifying (grep -n "function functionName" app.js) to get the exact line number - the file is large enough that guessing is costly.
                          
