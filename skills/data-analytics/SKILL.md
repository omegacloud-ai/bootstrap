---
name: data-analytics
description: Use when building data analytics, BI dashboards, reports, CSV parsing, EDA, aggregation, or data visualization in React Next.js - enforces clarification-first analysis, schema validation, reproducible Papa Parse/Arquero/Lodash pipelines, and Recharts visuals
---

# Data Analytics

## Overview

Act as a senior data analyst and data visualization engineer. Produce analytics pipelines, dashboards, and reports that are correct, clear, reproducible, maintainable, and trustworthy enough for production data teams.

## Non-Negotiables

- If metrics, dimensions, time grain, aggregation logic, filters, business rules, schemas, or expected outputs are unclear, ask clarifying questions before writing code.
- Never guess data semantics, KPI definitions, or business logic. State assumptions only when unavoidable and low-risk.
- Use only this stack unless explicitly asked otherwise: Papa Parse for ingestion, Arquero for analytics/aggregation, Lodash for shaping, Recharts for visualization, React Next.js for UI.
- Do not add libraries, state managers, charting tools, hallucinated schemas, fake data, or made-up metrics.
- Explicitly convert values to the expected schema before analysis or rendering.
- Test each pipeline stage on small sample data before integration and verify output types, row shape, and schema.

## Required Workflow

1. Define the business goal, analytical objective, intended decision, and success criteria.
2. Create a data contract: expected columns, types, nullable fields, units, grain, primary keys, metric formulas, and output schema.
3. Inspect available data: first 5-10 records, columns, nulls, duplicate candidates, units, grain, inferred types, and coverage.
4. Validate schema; ask before proceeding if fields or semantics are ambiguous.
5. Parse raw data with Papa Parse when needed.
6. Clean data: trim fields, normalize categories, handle missing values, flag duplicates, and convert types.
7. Run EDA: distributions, outliers, missingness, cardinality, time coverage, partial periods, inactive entities, zero values, and data quality risks.
8. Transform and aggregate with Arquero.
9. Shape final UI/report data with Lodash.
10. Analyze results and validate assumptions.
11. Visualize with Recharts only when the chart fits the data semantics.

## Clarification Checklist

Ask when missing:

- What business question should this answer?
- Which entities and dimensions matter: SKU, category, location, customer, channel, cohort, region, time?
- Which metrics are authoritative, and how are they calculated?
- What time grain and timezone should be used?
- How should partial periods, missing values, zeros, inactive records, returns, cancellations, and duplicates be handled?
- Which filters, drill-downs, comparisons, and output format are required?
- What schema and data types should the final output expose?

## Analytical Rigor

- When appropriate, propose at least 10 plausible analytical approaches or hypotheses.
- When appropriate, suggest at least 10 high-impact charts, reports, tables, or dashboard views supported by the available data.
- Prefer simple, explainable transformations over clever opaque logic.
- Keep parsing, validation, cleaning, aggregation, shaping, and rendering as deterministic pure functions where practical.

## Visualization Rules

- Choose charts by semantics: line/area for trends, bar for ranked categories, stacked bar for composition, histogram-like summaries for distributions, scatter for relationships.
- Label axes, units, legends, tooltips, empty states, and filters clearly.
- Avoid misleading encodings: improper aggregation, mixed units, unexplained truncated axes, overloaded colors, and partial-period comparisons.
- Include intuitive filters and drill-downs such as SKU, category, location, status, and time.
- Use restrained color to highlight critical items, low stock, high turnover, inactive items, zero stock, exceptions, and data quality warnings.
- Optimize UI for scanning, clarity, and performance.

## Scale and Stress Rules

- If CSV size is unknown, over 25-50 MB, over 100k rows, or user mentions large exports, default to Papa Parse streaming/step callbacks, worker mode where practical, progress state, cancellation, and early aggregation.
- Avoid materializing full object arrays unless the dataset is known to fit comfortably in memory.
- Keep validation two-phase: cheap required-field/type coercion first, then deeper checks on samples, failing columns, or user-requested rules.
- Capture Papa Parse diagnostics: parse errors, delimiter/header issues, duplicate or empty headers, inconsistent row lengths, malformed quotes, embedded newlines, and suspicious encodings.
- Handle coercion explicitly: dates with timezone/grain, currency/percent strings, thousands separators, locale ambiguity, empty strings, `NaN`, `Infinity`, and negative formats like `(123)`.
- Minimize per-cell string churn; use tight loops, cached conversions, and controlled category dictionaries to prevent near-duplicate cardinality blowups.
- Keep EDA bounded: missingness, min/max, top-k categories, capped warnings, sampled examples, and approximate or bucketed summaries instead of unbounded row flags.
- Reduce early with Arquero: drop unused columns, bucket time before charting, aggregate before joins/shaping, and avoid high-cardinality groupbys unless required.
- Use Lodash only for final small presentation objects; avoid repeated cloning, sorting, nested grouping, or full-dataset reshaping.
- Cap Recharts inputs: downsample or aggregate large series, set a max point count per series, disable animations for dense visuals, avoid per-point custom elements/tooltips, and prefer coarser grains for interaction.

## Audit and Quality Gates

- Track row counts at each stage: parsed, rejected, cleaned, deduplicated, aggregated, joined, and rendered.
- Report data quality: missingness, invalid rows, duplicates, unexpected categories, outliers, partial periods, parse errors, and assumptions.
- For multi-table analytics, declare each table's grain and validate join cardinality before aggregation to avoid double-counting.
- Ask only blocker questions needed for correctness; do not stall on non-semantic UI preferences.
- Protect sensitive data and exports: avoid unnecessary retention, redact PII when appropriate, and neutralize spreadsheet formula injection for exported values starting with `=`, `+`, `-`, or `@`.

## Implementation Pattern

Use this order: `parseRawData()` with Papa Parse, `validateSchema()`, `normalizeRows()` with explicit conversions, `runEdaChecks()`, `aggregateRows()` with Arquero, `shapeForView()` with Lodash, then Recharts rendering.

## Testing

Test parsing, schema validation, type conversion, aggregation, final shaping, and chart input separately. Include missing values, duplicated rows, malformed numbers, empty datasets, zeros, inactive items, and partial periods. Before completion, run available lint/build/test commands and inspect rendered charts or serialized chart data.

## Common Mistakes

Coding before KPI clarification; parsing huge files into arrays by default; treating CSV strings as typed values; aggregating incompatible grains; hiding data quality issues; rendering Recharts before proving input schema; adding unapproved libraries.
