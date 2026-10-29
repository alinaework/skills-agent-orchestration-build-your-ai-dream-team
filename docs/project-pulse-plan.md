# Project Pulse Dashboard Implementation Plan

## Goal and repository context

Build Mona's Project Pulse as a lightweight static dashboard that lets contributors scan project names, owners, status, recent activity, priority or risk, and a short summary. Use `app/project-data.json` as the source of project records, render them in `app/index.html`, and provide a responsive, accessible visual design in `app/styles.css`. The dashboard must run from the VS Code **Run Project Pulse Dashboard** launch configuration and open the page rather than a directory listing.

The brief and exercise steps are the product requirements. The repository currently has no `app/` implementation, package manifest, framework, or app-specific test runner. Existing patterns favor a small Codespaces exercise using prebuilt custom agents, so keep the implementation dependency-free and do not add a build system.

## Agent responsibilities and file assignments

| Agent | Assigned file(s) | Deliverable and responsibility |
| --- | --- | --- |
| **Designer** | `app/styles.css` | Define the visual direction and responsive layout: clear dashboard hierarchy, readable project cards and spacing, status and priority treatments, legible typography, focus indicators, and sufficient contrast. Coordinate the agreed markup hooks and semantics with Coder before styling. |
| **Coder** | `app/project-data.json` | Create strict JSON with a top-level `projects` array and representative, consistent records. Include required `name`, `owner`, `status`, `recentActivity`, and `priority` fields; include a short `summary` field to meet the brief's contributor-friendly-summary goal. |
| **Coder** | `app/index.html` | Create semantic page structure titled **Project Pulse**, link `styles.css`, load `project-data.json`, and render project cards with the `project-card` class. Include understandable loading, empty, and fetch/parse error states; derive displayed cards and any summaries from loaded records. |
| **Coder** | `.vscode/launch.json` | Add strict JSON configuration named **Run Project Pulse Dashboard**. Serve the `app/` directory with `python3 -m http.server 5500`, set `cwd` to `${workspaceFolder}/app`, and use `serverReadyAction` to open `http://localhost:%s/index.html`. This opens the dashboard page rather than the server's directory listing. |

File ownership is exclusive: Designer owns CSS; Coder owns HTML, data, and launch configuration. The Orchestrator confirms the shared contract, assigns these scopes, coordinates integration, and reports what was validated. Planner defines the phases, dependencies, risks, and validation; Planner does not implement application files.

## Shared implementation contract

1. Use a top-level `projects` array. Every item has the five required fields from the brief. Add `summary` as a short plain-text field because the brief explicitly asks for a contributor-friendly summary, although it is not in the required-field list.
2. Before parallel implementation, agree on the permitted status and priority values, their user-facing labels, and stable CSS hooks. The brief does not prescribe enum values; choose a small, meaningful set and use it consistently in data, rendering, and styling. Keep status distinct from priority/risk.
3. Use `app/project-data.json` fetched over HTTP. Do not rely on `file://` access: browsers commonly block fetches of local JSON files.
4. HTML must provide a named page heading, a semantic main/dashboard region, a location for status/feedback messages, and a repeatable project-card structure. Use text labels as well as color for status and priority. Populate all dynamic values as text, not executable markup.
5. CSS selectors must include `.dashboard` and `.project-card`, as required by the Designer definition and exercise checks.

## Sequencing, dependencies, and parallel work

| Phase | Owner | Work and completion gate |
| --- | --- | --- |
| 1. Confirm requirements and contract | Orchestrator with Designer and Coder | Review `.github/project-pulse-brief.md` and this plan. Agree fields (including `summary`), status/priority values, HTML sections, CSS hooks, and launch behavior. This is the prerequisite for parallel work. |
| 2. Create the data source | Coder | Create and parse-check `app/project-data.json`; include multiple representative projects with all contract fields. The HTML renderer depends on this schema. |
| 3. Implement page and styling | Coder and Designer, in parallel after Phase 1 | Coder builds `app/index.html` against the agreed schema/hooks. Designer builds `app/styles.css` against the same hooks and responsive/accessibility requirements. Their file scopes are disjoint, so they can work in parallel. Coder should finish the schema before wiring rendering; the page and stylesheet can then progress concurrently. |
| 4. Configure preview | Coder | Create `.vscode/launch.json` using the required Python HTTP server command, app working directory, fixed port, and ready action. Confirm the selected VS Code debugger supports the configuration in the Codespaces environment. |
| 5. Integrate and validate | Orchestrator coordinates; Coder and Designer validate their scopes | Run the launch configuration, verify real data renders, check responsive/accessibility behavior and failure states, and resolve any contract mismatch with the owning agent. Complete this only after all four assigned files are integrated. |

**Dependencies:** JSON schema → HTML rendering; agreed HTML hooks and page sections → stylesheet; `cwd` and server startup behavior → launch URL/ready action; all four files and a running local preview → end-to-end browser validation.

**Parallel decision:** Parallelize only the CSS and HTML implementation after Phase 1 locks the shared contract. Keep JSON definition before renderer integration, and keep final launch configuration / preview verification after server behavior is confirmed. If the schema or markup hooks need to change, pause dependent edits, agree the change with the other owner, and then resume. Do not have both agents edit the same file.

## Edge cases and handling

- **No projects:** Show a clear empty-state message instead of a blank dashboard.
- **Unavailable, invalid, or malformed JSON:** Show a useful error state; do not show fabricated data or leave the page looking successfully loaded.
- **Missing fields or unrecognized status/priority:** Avoid rendering broken or misleading badges. Use a neutral fallback label or flag the record clearly; never depend on color alone.
- **Long names, summaries, owner names, or activity:** Allow wrapping and preserve card layout without clipping or horizontal overflow.
- **Narrow viewport, zoom, keyboard, and reduced space:** Keep content usable at mobile widths and with keyboard navigation; provide visible focus and avoid interaction or information conveyed only by color.
- **Server port already occupied or launch not ready:** Make the failure diagnosable. Keep launch port and `serverReadyAction` pattern aligned with the actual server output; stop the preview server after validation.
- **Data safety:** Render JSON-sourced values as text. Do not interpolate untrusted strings into HTML markup.

## Validation expectations

1. Parse `app/project-data.json` as strict JSON. Verify the top-level `projects` array and required fields (plus the planned `summary`) for every record; check that status and priority values match the agreed sets.
2. Parse `.vscode/launch.json` as strict JSON with no comments. Confirm the exact launch name, `python3 -m http.server 5500`, `cwd` of `${workspaceFolder}/app`, and a `serverReadyAction` that opens `http://localhost:%s/index.html`.
3. From VS Code Run and Debug, run **Run Project Pulse Dashboard**. Confirm that the browser opens the dashboard, not a directory listing, and that the browser fetches `project-data.json` successfully.
4. Verify each card and any summary values are data-driven and show the required project fields. Exercise empty, fetch-failure, malformed-data, and unknown-value behavior where practical.
5. Inspect browser console/network output for failed requests or uncaught errors.
6. Check a desktop and narrow/mobile viewport for readable hierarchy, wrapping, and no clipped content or horizontal overflow. Check keyboard navigation, visible focus, semantic headings/regions, text labels for status/priority, and sufficient contrast.
7. Record the browser and launch checks and any remaining limitation in the Orchestrator's final handoff. No application-specific automated test command exists at planning time; use focused JSON/config parsing plus manual browser checks unless implementation adds a justified test mechanism.

## Assumptions and uncertainty

- `python3` is available in the Codespace, and the launch configuration can use a VS Code Python-capable debugger to run `python3 -m http.server 5500`. The repository does not currently declare a Python debugger extension or another app preview mechanism. Coder should verify that launch support in the target Codespace before finalizing `launch.json`; if unavailable, coordinate the smallest compatible launch setup with Orchestrator without changing the required command, app working directory, launch name, or browser URL.
- The exact project examples and status/priority enum values are not specified by the brief. Coder should choose representative values during Phase 1, record the contract in the implementation handoff, and keep data and rendering consistent.
- `summary` is an additional planned field: the brief asks for a short contributor-friendly summary, while its required field list names only five fields. Treat the five named fields as mandatory and `summary` as additive.
- Validation is expected in the Codespaces/VS Code browser workflow described by the exercise; the repository does not include app-specific automated browser tests.
