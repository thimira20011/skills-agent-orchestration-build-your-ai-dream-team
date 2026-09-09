# Project Pulse Dashboard Implementation Plan

## Summary

Build Mona's Project Pulse as a small, dependency-free static dashboard in GitHub Codespaces. It will present active projects, owners, statuses, recent activity, and priority or risk through accessible, responsive project cards.

The work is coordinated through GitHub Copilot CLI using the custom agents in `.github/agents/`. The Orchestrator coordinates and verifies the work; specialist agents do not stage, commit, or push changes.

## Agent responsibilities and file assignments

| Agent | Responsibilities | File assignments |
| --- | --- | --- |
| **Orchestrator** | Coordinates the workflow through GitHub Copilot CLI, delegates explicit scopes, manages dependencies, prevents conflicting edits, integrates the result, and performs final verification. | Coordination only |
| **Planner** | Researches the brief and repository, identifies requirements, risks, dependencies, edge cases, and validation needs, then produces this plan. | `docs/project-pulse-plan.md` |
| **Designer** | Defines information hierarchy, semantic structure, accessibility requirements, visual language, responsive behavior, status and priority treatment, spacing, typography, and card layout. | `app/styles.css` |
| **Coder** | Implements the static dashboard, connects the page to the stylesheet and JSON data, renders project cards and error states, and creates the runnable VS Code configuration. | `app/index.html`, `app/project-data.json`, `.vscode/launch.json` |

The Designer and Coder must not modify one another's files. The Orchestrator should pass the Designer's selector and accessibility contract to the Coder before the Coder finalizes `app/index.html`.

## Ordered implementation steps

### 1. Coordinate the Codespace workflow

**Owner:** Orchestrator

The Orchestrator reads `.github/project-pulse-brief.md`, `.github/agents/*.agent.md`, and relevant Codespace configuration, then delegates work with explicit file scopes. The workflow is run through GitHub Copilot CLI in the Codespace.

### 2. Complete the plan

**Owner:** Planner
**File:** `docs/project-pulse-plan.md`

The plan establishes the static, dependency-free approach, exact file assignments, agent responsibilities, dependencies, parallel and sequential work, risks, edge cases, and validation expectations.

### 3. Produce the design contract

**Owner:** Designer
**File:** `app/styles.css`

Create a polished, accessible visual system with:

- A dashboard shell using `.dashboard`.
- A responsive project-card grid using `.project-card`.
- Readable spacing, `border-radius`, and `box-shadow`.
- Status badges and priority treatment that do not rely on color alone.
- Clear typography hierarchy, focus states, sufficient contrast, and sensible narrow-screen behavior.
- Wrapping for long project names, owners, and activity text.

The Designer reports selector and markup expectations to the Orchestrator.

### 4. Create data and launch configuration

**Owner:** Coder
**Files:** `app/project-data.json`, `.vscode/launch.json`

Create valid JSON with a top-level `projects` array. Every project includes:

- `name`
- `owner`
- `status`
- `recentActivity`
- `priority`

Include multiple representative projects and use deterministic, local data.

Create strict JSON launch configuration with the exact name `Run Project Pulse Dashboard`. It should use `python3 -m http.server 5500`, set `cwd` to `${workspaceFolder}/app`, serve the `app` directory, and open `http://localhost:%s/index.html` through `serverReadyAction`.

### 5. Implement the dashboard page

**Owner:** Coder
**File:** `app/index.html`
**Dependency:** Designer's selector, hierarchy, and accessibility contract

Implement a semantic static page that:

- Uses the visible title `Project Pulse`.
- References `styles.css` and loads `project-data.json`.
- Uses accessible landmarks and heading structure.
- Provides a `.dashboard` container and one `.project-card` per project.
- Displays every required project field.
- Includes a contributor-oriented summary.
- Provides loading, empty-data, and fetch-error states.
- Uses safe DOM updates when inserting JSON values.
- Avoids external frameworks and runtime dependencies.

Because local JSON loading requires HTTP, validate through the configured server rather than opening the file with `file://`.

### 6. Integrate and review

**Owner:** Orchestrator
**Dependencies:** Planner, Designer, and Coder outputs

Review all four implementation files together and verify that HTML selectors match CSS, JSON fields match rendering logic, all required fields are displayed, the relative JSON path works, the launch configuration opens `index.html`, and the first viewport presents a dashboard rather than a directory listing. Reassign only the affected file if integration corrections are needed.

## Dependencies

1. Repository and brief research must precede planning.
2. The Planner must complete `docs/project-pulse-plan.md` before implementation delegation.
3. The Designer must define the markup and selector contract before the Coder finalizes `app/index.html`.
4. The Coder must connect the page to the completed data schema and design contract.
5. The Orchestrator must review the integrated files before launch testing.
6. Final validation occurs after all four implementation files exist.

## Parallel work decisions

After the plan is complete, these tasks can run in parallel because they have non-overlapping scopes:

- Designer implements `app/styles.css`.
- Coder creates and validates `app/project-data.json`.
- Coder creates and validates `.vscode/launch.json`.

The Coder's `app/index.html` work must wait for the Designer's selector and accessibility contract. Integration review, launch testing, and final validation must wait for all implementation files.

## Validation expectations

### Plan validation

Confirm the plan exists and names the required roles, concepts, and paths:

```bash
test -f docs/project-pulse-plan.md
grep -Eiq 'Project Pulse' docs/project-pulse-plan.md
grep -Eiq 'Designer' docs/project-pulse-plan.md
grep -Eiq 'Coder' docs/project-pulse-plan.md
grep -Eiq 'dependencies' docs/project-pulse-plan.md
grep -Eiq 'parallel' docs/project-pulse-plan.md
grep -Eiq 'validation' docs/project-pulse-plan.md
grep -Eiq 'app/index\.html' docs/project-pulse-plan.md
grep -Eiq 'app/styles\.css' docs/project-pulse-plan.md
grep -Eiq 'app/project-data\.json' docs/project-pulse-plan.md
grep -Eiq '\.vscode/launch\.json' docs/project-pulse-plan.md
```

### File, syntax, and data validation

```bash
test -f app/index.html
test -f app/styles.css
test -f app/project-data.json
test -f .vscode/launch.json
python3 -m json.tool app/project-data.json >/dev/null
python3 -m json.tool .vscode/launch.json >/dev/null
```

Validate the required data shape:

```bash
python3 - <<'PY'
import json

with open("app/project-data.json", encoding="utf-8") as file:
    data = json.load(file)

assert isinstance(data.get("projects"), list)
assert data["projects"], "projects must contain at least one project"
required = {"name", "owner", "status", "recentActivity", "priority"}
for project in data["projects"]:
    assert required.issubset(project), project
PY
```

### Content and integration validation

Confirm `app/index.html` contains `Project Pulse`, references both local assets, and uses `project-card`, status, recent activity, and priority. Confirm `app/styles.css` contains `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`. Confirm `.vscode/launch.json` contains `Run Project Pulse Dashboard`, `index.html`, `python3 -m http.server 5500`, `${workspaceFolder}/app`, and `http://localhost:%s/index.html`.

### HTTP and browser validation

Run the configured server and request all local assets:

```bash
python3 -m http.server 5500 --directory app >/tmp/project-pulse-http.log 2>&1 &
server_pid=$!
trap 'kill "$server_pid" 2>/dev/null || true' EXIT
curl --fail http://127.0.0.1:5500/index.html >/dev/null
curl --fail http://127.0.0.1:5500/styles.css >/dev/null
curl --fail http://127.0.0.1:5500/project-data.json >/dev/null
```

Run **Run Project Pulse Dashboard** from VS Code Run and Debug and confirm the browser opens `index.html` and displays the dashboard UI rather than a directory listing.

### Repository validation

Run the existing repository validator:

```bash
bash scripts/validate-exercise.sh
```

## Risks and edge cases

- `fetch("project-data.json")` can fail under `file://`; always use HTTP validation.
- The launch URL must include `/index.html`, not only the server root.
- Invalid JSON must be caught with `python3 -m json.tool`.
- Missing fields, an empty project array, and fetch failures need visible, contributor-friendly states.
- Port `5500` may already be in use; stale servers should be stopped or the conflict reported.
- Long text must wrap without breaking the card grid.
- Status and priority must remain understandable without color alone.
- Designer and Coder must not edit the same file concurrently.
- Required selectors, field names, title, launch name, and target path should remain stable for deterministic workflow checks.

## Assumptions

- A vanilla HTML/CSS/JavaScript implementation is appropriate because the repository has no frontend framework or package manifest.
- Python 3 and `curl` are available in the Codespace.
- The Designer owns the stylesheet while the Coder owns the page, data, and launch configuration.
- A project `summary` field may be added, but the five required fields remain consistent.
- Git operations are performed by the learner through Copilot CLI prompts after the work is complete.
