# Project Pulse Dashboard – Implementation Plan

## Goal
Build **Project Pulse**, a static dashboard showing which projects are active, who owns each, status, recent activity, and priority. Polished project cards, status badges, responsive layout. Runs from the VS Code **Run Project Pulse Dashboard** launch configuration and opens `index.html`.

## Conventions
- Brief: `.github/project-pulse-brief.md`; agent roles: `.github/agents/*.agent.md`.
- Required strings (checked by workflows):
  - `index.html`: `Project Pulse`, `styles.css`, `project-data.json`, `project-card`, `status`, `recentActivity`, `priority`
  - `styles.css`: `.dashboard`, `.project-card`, `border-radius`, `box-shadow`
  - `project-data.json`: `projects`, `name`, `owner`, `status`, `recentActivity`, `priority`
  - `launch.json`: `Run Project Pulse Dashboard`, `index.html`; strict JSON
- Plain HTML/CSS/JSON, no build tools; Python's built-in HTTP server.

## File assignments

| File | Owner | Purpose | Contents |
| --- | --- | --- | --- |
| `app/project-data.json` | Coder | Single source of data | Strict JSON; top-level `projects` array of 4–6 items with `name`, `owner`, `status`, `recentActivity`, `priority` (optional `summary`). Status: `On Track`, `At Risk`, `Blocked`, `Completed`. Priority: `High`, `Medium`, `Low`. Include each status at least once. |
| `app/styles.css` | Designer | Visual system | `:root` color/spacing variables; `.dashboard` container; responsive `.project-grid` (`repeat(auto-fill, minmax(280px, 1fr))`); `.project-card` with `border-radius`, `box-shadow`, hover/`:focus-within`; `.status-badge` + `.status--on-track/at-risk/blocked/completed`; `.priority-badge` + `.priority--high/medium/low`; `:focus-visible` outline; 4.5:1 contrast; `@media (max-width: 600px)`; `prefers-reduced-motion`. |
| `app/index.html` | Coder (Designer reviews) | Markup + rendering | `lang="en"`, viewport meta, title `Project Pulse`, link to `styles.css`; `<header>` with `<h1>`, `<main class="dashboard">`, summary area, `.project-grid`. Inline script fetches `project-data.json`, renders an `<article class="project-card">` per project (`<h2>` name; owner, status, recentActivity, priority). Use `textContent`, not `innerHTML`. Loading, error, and empty states. |
| `.vscode/launch.json` | Coder | Run from VS Code | `version` `0.2.0`; one `node-terminal` config named `Run Project Pulse Dashboard`, command `python3 -m http.server 5500`, `cwd` `${workspaceFolder}/app`, `serverReadyAction` opening `http://localhost:%s/index.html`. |

No other files (no separate JS file).

## Designer responsibilities
- Owns `app/styles.css`.
- Defines the class-name contract: `.dashboard`, `.dashboard-header`, `.summary`, `.project-grid`, `.project-card`, `.project-card__title`, `.project-card__meta`, `.status-badge`, `.status--*`, `.priority-badge`, `.priority--*`, `.state-message`.
- Reading order and hierarchy; badges use text, not color alone.
- Accessibility: contrast, visible focus, works at 320px and 200% zoom.
- Reviews finished `index.html`; changes go through the Coder.

## Coder responsibilities
- Owns `app/project-data.json`, `app/index.html`, `.vscode/launch.json`.
- Uses the Designer's class names exactly.
- `fetch` with `response.ok` check and clear in-page error; handles missing fields and unknown values gracefully.
- Ensures all required strings appear literally in files.
- Runs validation and reports changes, checks, and risks.

## Dependencies
1. Data schema and class-name contract come first.
2. `index.html` depends on the JSON schema and the class names.
3. `styles.css` depends only on the class-name contract and allowed values.
4. `launch.json` depends only on `app/index.html` existing at run time.
5. Final validation depends on all four files.

## Parallel work decisions
- **Phase 0 (sequential):** agree schema, allowed values, class names.
- **Phase 1 (parallel):** Designer → `styles.css`; Coder → `project-data.json`, then `index.html`, then `launch.json`. Each file has one owner, so no conflicts.
- **Phase 2 (sequential):** Designer reviews rendered page and markup; fixes routed to owners.
- **Phase 3 (sequential):** validation.
- Never: two agents editing one file; validating before all files exist.

## Validation expectations
**Automated**
- `python3 -m json.tool app/project-data.json` and `python3 -m json.tool .vscode/launch.json` succeed.
- Every project has `name`, `owner`, `status`, `recentActivity`, `priority`.
- Required strings present in each file.
- `cd app && python3 -m http.server 5500`; `curl` returns 200 for `index.html` and `project-data.json`.

**Manual**
- Launch config opens `http://localhost:5500/index.html` showing the dashboard.
- One card per project with name, owner, status badge, recent activity, priority badge.
- Responsive: one column at 375px, multiple at 1280px, no horizontal scroll at 320px.
- Accessible: visible focus, readable badges without color, ordered headings, no serious contrast issues.
- Opening via `file://` shows the friendly error.

**Acceptance criteria**
- All four files exist and pass the checks above.
- Title and `h1` are `Project Pulse`; cards use `project-card`.
- Launch config is strict JSON, runs from `app/`, opens `index.html`.

## Edge cases
- `file://` fetch failure → error message pointing to the launch config.
- Port 5500 in use → stop earlier server.
- Python output buffering may delay browser open → consider `python3 -u`.
- Missing field/unknown value → show "—" or neutral badge.
- Empty `projects` → empty-state message.
- Long text wraps (`overflow-wrap: anywhere`).
