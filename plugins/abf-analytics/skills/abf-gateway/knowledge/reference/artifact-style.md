# Artifact Style Guide

Default visual style for any rendered artifact this plugin produces - an inline HTML/React dashboard, an exported Word or PDF report, an Excel export, or a standalone chart image. Plain chat-markdown replies are out of scope here; those follow `../contracts/response-discipline.md` as they already do.

Chart type selection is owned by `visuals-playbook.md` in this same folder - use its per-analysis defaults. This guide governs only how a chosen chart, table, or page is rendered: colors, fonts, layout.

## Document properties (exported documents)

- Orientation: Landscape
- Size: Letter (11" x 8.5")
- Margins: 0.75" all sides
- Font family: Sans-serif (Calibri, Arial, or similar)

HTML artifacts don't have fixed page geometry - use the CSS variables below instead.

## Color palette

| Role | Color | Usage |
|------|-------|-------|
| Primary dark | `#1A1446` | Table headers, page titles, heading text, section bars |
| Accent gold | `#FFD000` | Cover bar, highlights, KPI labels, accent lines |
| Teal | `#06748C` | Charts primary series, data visualizations, secondary cards |
| Light teal | `#78E1E1` | Chart secondary series, allocation bars |
| Grey | `#c0bfc0` | Borders, disabled elements, footnote text |
| White | `#FFFFFF` | Card backgrounds, body text area |
| Light grey | `#F2F2F2` | Page background, alternating table rows |

Chart color assignment order (use in sequence for multi-series charts):
1. `#06748C` (teal - primary series)
2. `#1A1446` (navy - secondary series)
3. `#FFD000` (gold - accent/highlight)
4. `#78E1E1` (light teal - tertiary)
5. `#c0bfc0` (grey - other/remainder)

## Typography hierarchy

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| Page title | 28pt | Light | `#1A1446` |
| Section header | 18pt | Bold | `#1A1446` |
| KPI number | 24pt | Bold | `#1A1446` |
| KPI label | 10pt | Regular | `#716b75` |
| Body text | 11pt | Regular | `#141413` |
| Table header | 10pt | Bold | `#FFFFFF` (on `#1A1446` background) |
| Table data | 10pt | Regular | `#141413` |
| Footnote | 8pt | Regular | `#c0bfc0` |

## Page types

### Cover page
- Full-bleed dark background (`#1A1446`)
- Deal/transaction name in large white sans-serif
- Subtitle in gold (`#FFD000`)
- "As of {period}" in gold below subtitle
- "Confidential & Proprietary" footer bar in gold

### KPI summary page
- Horizontal KPI strip: bordered cards in a row (4-6 cells)
  - Each cell: large bold number + small label below
  - Thin top border in teal (`#06748C`)
- Body text below: key takeaways as bullet points
- Footnotes at bottom in small grey text

### Data table page
- Header row: dark background (`#1A1446`), white text
- Data rows: alternating white / light grey (`#F2F2F2`)
- Borders: thin grey (`#c0bfc0`)
- Numbers right-aligned, text left-aligned
- Dollar values with $ prefix, percentages with % suffix

### Chart page
- Chart title in section header style
- Chart rendered as embedded image (matplotlib for Word/PDF, SVG/canvas for artifacts)
- Supporting data table below the chart
- Narrative context paragraph below

## Rendering rules

- Red flags render as a callout box with a thin red left border
- Narrative fragments render as indented block quotes with source attribution
- Delta indicators: green arrow for positive >15%, red arrow for negative >15%
- Missing data: show "-" in the cell, never leave blank
- Page numbers bottom-left, "Confidential" footer center (exported documents only)

## Artifact-specific CSS variables

For HTML artifacts:

```css
:root {
  --color-gold: #FFD000;
  --color-teal: #06748C;
  --color-navy: #1A1446;
  --color-light-teal: #78E1E1;
  --color-grey: #c0bfc0;
  --bg-dashboard: #f2f2f2;
  --bg-card: #ffffff;
  --card-radius: 8px;
  --card-shadow: 0 1px 3px rgba(0,0,0,0.08);
  --btn-primary: #2D72EA;
  --btn-secondary: #EAF1FD;
  --icon-bg: #f2f2f2;
  --icon-color: #716b75;
  --text-primary: #141413;
  --text-muted: #716b75;
  --text-on-dark: #ffffff;
  --status-positive: #22c55e;
  --status-negative: #ef4444;
  --status-neutral: #6b7280;
}
```
