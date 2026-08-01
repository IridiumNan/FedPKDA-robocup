# FedPKDA UI Component Inventory

Source of truth: current redesigned `coordinator/event_center.html`.

## Foundation
- Colors: `src/styles/design-system/colors.css`
- Spacing, radius, shadow, typography: `src/styles/design-system/variables.css`
- Cross-page component rules: `src/styles/design-system/theme.css`

## Components

### PageHeader
Use for every page title area. Contains title, subtitle, and right-side status/actions.
Tokens: `--text-primary`, `--text-secondary`, `--layout-header-min-height`.
Do: one clear primary action. Do not: stack unrelated badges.

### CommonCard
Base elevated panel. Used for charts, tables, topology, model status, summaries.
Tokens: `--card-radius`, `--shadow-card`, `--bg-elevated`.
States: hover lifts by 3px and increases shadow.

### MetricCard
Dashboard KPI card with icon, label, number, trend/status.
Variants: primary, success, warning, danger, info via semantic color tokens.

### StatusBadge / RiskTag
Compact state labels. Always pair color with text.
Variants: success, warning, danger, info.

### ChartContainer
Shared chart shell with title, legend/status and SVG/chart content.
Tooltip style: dark translucent surface with 12px radius.

### Table
Table header is muted, sticky where needed, row hover uses primary soft background.
Rows with click handlers must be keyboard reachable.

## Gaps / decisions
- This is a static HTML project, so components are documented snippets plus shared CSS classes rather than React components.
- Future React migration can map these snippets 1:1 to `PageHeader`, `CommonCard`, `MetricCard`, `StatusBadge`, `RiskTag`, and `ChartContainer` components.
