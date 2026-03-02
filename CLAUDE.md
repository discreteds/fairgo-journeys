# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

User journey documentation for the **Fair Go** web SaaS expense-splitting application. This is a documentation-only repo — no application code, no tests, no build pipeline. The actual implementation lives in sibling repos (`fairgo-backend` for the FastAPI API, `fairgo-frontend` for the React SPA).

The goal: map how simple UI interactions trigger orchestrated sequences of backend operations (72 endpoints) with smart defaults, so the complex backend feels simple to end users.

## Local Development

The site uses **Docsify** (client-side markdown renderer). To preview locally:

```bash
npx docsify-cli serve .
# Opens at http://localhost:3000
```

No build step. Markdown files are rendered as-is. Deployed via GitHub Pages (`.nojekyll` disables Jekyll).

## Document Architecture

Four document types cross-reference each other using shared IDs:

| Type | Prefix | Location | Count | Purpose |
|------|--------|----------|-------|---------|
| Screen specs | S## | `screens/` | 17 | **Source of truth.** ASCII wireframes, component breakdowns, state variants, API orchestrations |
| Scenarios | SC## | `scenarios/` | 12 | End-to-end user narratives referencing screen IDs |
| Journey Rails | R## | `rails/` | 6 | Linear screen-ID sequences (shortest path through a workflow) |
| Appendix | A## | `appendix/` | 5 | Reference: orchestrations, smart defaults, RBAC matrix, API changes, gap changelog |

**S05 Event Dashboard** is the hub screen — every rail and scenario passes through it.

### Cross-validation principle

All three document types reference the same screen IDs (S01–S17), so they validate each other: screens define what exists, scenarios prove narrative completeness, rails prove navigation coherence.

### Root-level documents

- `00-overview.md` — Project purpose, design decisions, glossary
- `01-navigation-map.md` — Master screen graph with connection inventory
- `nav-map-interactive.md` — Mermaid flowchart of screen relationships
- `_sidebar.md` / `_navbar.md` / `_coverpage.md` — Docsify navigation
- `index.html` — Docsify configuration (plugins: search, pagination, copy-code, dark/light theme, Mermaid)

## Naming and Formatting Conventions

- **IDs are stable:** S05, SC01, R02, A03 — never renumber existing documents
- **Screen specs** include: header with screen name, purpose, wireframe (ASCII art in code blocks), component table, state variants (empty/loading/error/populated), role-aware sections (admin vs member), and orchestration details (exact API endpoints and call sequences)
- **Scenarios** are step-by-step narratives with `→ Screen: S##` references at each step
- **Rails** list screen transitions as `S## → S## → S##` with trigger descriptions
- **Appendix** docs use tables for reference data (orchestration chains, permission matrices, default rules)

## Domain Glossary

When editing docs, use the correct UI-facing terms:

| UI Term | Backend Concept | Meaning |
|---------|----------------|---------|
| Settlement Group | Primary Financial Group (PFG) | Who settles as one unit (colour-coded) |
| Person | PERSON | Financial participant (may or may not have an account) |
| Group | GROUP | Named collection of persons for consumption splitting |
| Expense | TRANSACTION + LINE_ITEM(s) | Container with one or more line items |
| Line Item | LINE_ITEM + LINE_ITEM_SPLIT(s) | Atomic financial unit with per-person weights |
| Balance | Position (PFG net) | How much a settlement group owes or is owed |

## Key Design Principles

- **Aggressive defaults:** Event creation = 1 field (name). Expense entry = 2 fields (description + amount). Default payer = current user. Default split = everyone equally.
- **Capture first, share last:** Expenses can be entered before people are added (split-pending state). Splits auto-assign when people are added.
- **Role-aware, not role-separated:** Admin and member see the same screens with different controls visible. UI hides; backend enforces.
- **Mobile-first responsive:** Stacked layouts expanding on desktop. Bottom nav (mobile) / sidebar (desktop).
