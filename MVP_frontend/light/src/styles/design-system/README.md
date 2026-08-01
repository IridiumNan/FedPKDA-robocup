# FedPKDA Design System

Extracted from the redesigned Event Center dashboard and applied across the FedPKDA static frontend.

## Color Tokens

| Token | Value | Usage |
| --- | --- | --- |
| `--primary-color` | `#2563EB` | Primary actions, selected navigation, main chart line |
| `--success-color` | `#10B981` | Healthy / online / running state |
| `--warning-color` | `#F59E0B` | Watch / delayed / needs attention state |
| `--danger-color` | `#EF4444` | High risk / offline / alert state |
| `--bg-color` | `#F8FAFC` | Dashboard background |
| `--bg-elevated` | `rgba(255,255,255,0.86)` | Translucent card surface |
| `--surface-color` | `rgba(255,255,255,0.92)` | Stronger panels and modals |
| `--text-primary` | `#111827` | Page title, KPI numbers, primary labels |
| `--text-secondary` | `#64748B` | Body text and secondary metadata |
| `--text-muted` | `#94A3B8` | Eyebrows, captions, table header text |

## Component Rules

### Card
- Radius: `16px`
- Padding: `16px` default, `20px` for dense summary panels
- Shadow: `--shadow-card`
- Hover: `translateY(-3px)` plus `--shadow-card-hover`
- Surface: translucent white over `#F8FAFC`

### Button
- `primary`: blue gradient, primary action only
- `ghost`: neutral bordered button
- `danger`: low-saturation red surface, not solid red by default
- States: hover lift, active scale, visible `focus-visible`

### Badge / RiskTag
- Always include text; color is never the only status signal
- Variants: success, warning, danger, info
- Shape: pill radius `999px`

### Table
- Header: muted background, small uppercase-like label weight
- Rows: compact spacing, primary-soft hover
- Clickable rows must be keyboard reachable

### Chart
- Primary line: `--primary-color`
- Semantic lines and dots use success / warning / danger tokens
- Tooltip: dark translucent surface, `12px` radius, subtle shadow

### Modal
- Use `.ds-modal`
- Radius `16px`, translucent white, heavy overlay shadow

## Layout Rules

- Sidebar width: `248px`
- Content padding: `24px 28px 32px`
- Header min-height: `72px`
- Section gap: `18px`
- KPI grid: 4 columns desktop, 2 columns under 1440px, 1 column mobile
- Card gap: `16px`

## Page Template

```html
<body class="layout risk-dashboard fed-dashboard">
  <aside class="sidebar">...</aside>
  <main class="main">
    <div class="page-header">...</div>
    <div class="grid-4">... KPI cards ...</div>
    <section class="card">...</section>
  </main>
</body>
```
