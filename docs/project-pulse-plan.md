# Project Pulse implementation plan

## Summary

Build a small, dependency-free static Project Pulse dashboard for Mona's
contributors. The first view should make active projects easy to scan by
showing each project's name, owner, status, recent activity, and priority or
risk in readable project cards. The result must be a polished, accessible
frontend rather than a directory listing, and it must be runnable from VS
Code through **Run Project Pulse Dashboard**.

The repository has no existing `app/` implementation, package manifest, or
frontend build system. The authoritative product requirements are
`.github/project-pulse-brief.md` and the checks in
`.github/workflows/3-step.yml`; the implementation should therefore use plain
HTML, CSS, JSON, and the Python standard-library HTTP server.

## File assignments and ownership

| File | Primary owner | Assignment |
| --- | --- | --- |
| `app/index.html` | Coder, incorporating Designer's handoff | Create the semantic dashboard shell, exact `Project Pulse` title, accessible headings and landmarks, stylesheet link, JSON data reference, and rendering logic that creates one visible `.project-card` per project and displays `status`, `recentActivity`, and `priority`. Keep data loading and failure states explicit. |
| `app/styles.css` | Designer | Define the visual system and responsive layout, including `.dashboard` and `.project-card`, readable spacing and typography, status/priority treatments, `border-radius`, `box-shadow`, keyboard/focus states, contrast, and narrow-to-wide viewport behavior. Designer owns this file and does not edit markup or data. |
| `app/project-data.json` | Coder | Create valid JSON with a top-level `projects` array. Every project object must contain non-empty `name`, `owner`, `status`, `recentActivity`, and `priority` values, with multiple representative projects so the card layout can be reviewed. |
| `.vscode/launch.json` | Coder | Create strict JSON with a configuration named `Run Project Pulse Dashboard`; serve from `${workspaceFolder}/app` with `python3 -m http.server 5500`, and use `serverReadyAction` to open `http://localhost:%s/index.html`, not the directory root. |

The Planner owns this plan only. The Orchestrator owns delegation, conflict
resolution, integration review, and final reporting; it should not edit the
implementation files. No agent should stage, commit, or push.

## Designer responsibilities

1. Review the brief and establish the information hierarchy: page title and
   short purpose statement first, then a responsive collection of project
   cards, with status and priority visually prominent but also available as
   text.
2. Specify semantic and accessible markup expectations for the Coder:
   landmarks, heading order, meaningful labels, readable status text, visible
   keyboard focus, sufficient color contrast, and no meaning conveyed by
   color alone.
3. Implement only `app/styles.css` after agreeing on the markup hooks. Use the
   required `.dashboard` and `.project-card` selectors and ensure cards remain
   legible on small screens, including long project names and activity text.
4. Report the selectors, responsive assumptions, and accessibility decisions
   to the Orchestrator for the Coder's integration pass.

## Coder responsibilities

1. Create `app/project-data.json` using the required schema and realistic,
   contributor-friendly sample content. Do not add a second data shape or
   require a server API.
2. Create `app/index.html` using the Designer's agreed hooks. Link
   `styles.css`, reference `project-data.json`, and render data-driven cards
   rather than hard-coding only one project. The rendered markup must include
   the literal `project-card` class and expose the required field names or
   corresponding labels in the UI.
3. Handle fetch, malformed JSON, an absent/invalid `projects` array, and an
   empty array with an explicit in-page message; do not silently show a
   successful-looking empty dashboard. Preserve usable static structure while
   data is loading.
4. Create `.vscode/launch.json` as strict JSON with no comments. Use a
   deterministic port and `cwd` of `${workspaceFolder}/app`; ensure the
   server-ready URL ends in `/index.html`.
5. Validate the four assigned files and resolve integration issues without
   changing unrelated repository files.

## Dependencies and integration contract

- There are no third-party runtime or build dependencies. The browser only
  needs the local HTML, CSS, and JSON files; preview needs Python 3, which is
  already used by repository validation commands.
- `app/index.html` depends on the `styles.css` selectors supplied by
  Designer, the `projects` schema supplied in `project-data.json`, and a
  running HTTP server for `fetch`; opening the HTML directly with a
  `file://` URL is not a valid data-loading test.
- `.vscode/launch.json` depends on the final app directory and its
  `index.html`; its server URL and working directory must match the files
  Coder creates.
- Designer may work on `app/styles.css` in parallel with Coder's
  `app/project-data.json` and the initial `app/index.html` structure because
  those files do not overlap. The Coder must receive the Designer's selector
  and markup contract before the final markup/rendering pass.
- The launch configuration is independent of visual styling but must be
  integrated after `app/index.html` exists so its target can be verified.

## Ordered implementation phases

### Phase 1: Orchestrator kickoff and contracts

The Orchestrator confirms the brief, this plan, and the four file scopes.
Designer records the semantic structure, required hooks, responsive behavior,
and accessibility criteria. Coder records the JSON schema and rendering
contract before editing. No implementation files are changed by the
Orchestrator.

### Phase 2: Parallel specialist work

- **Designer:** implement `app/styles.css` only, including `.dashboard`,
  `.project-card`, polished card treatment, status/priority variants,
  responsive layout, and focus/contrast states.
- **Coder:** implement `app/project-data.json`, the initial
  `app/index.html`, and `.vscode/launch.json`, using the brief's required
  fields and launch settings.

Parallel work is safe because Designer owns only CSS while Coder owns the
other three files. If markup hooks are still uncertain, Designer should first
send the contract and wait for agreement rather than guessing or editing HTML.

### Phase 3: Sequential integration

1. Coder reviews Designer's completed CSS selectors against the HTML and
   adjusts only `app/index.html` hooks/classes as needed.
2. Coder runs the page through the HTTP server and verifies that
   `project-data.json` produces multiple visible cards with all required
   fields.
3. Designer reviews the integrated rendering at narrow and wide viewport
   sizes, checking hierarchy, wrapping, focus visibility, contrast, and
   status/priority clarity. Any corrections remain within the assigned CSS
   file unless an actual markup defect is found.
4. Orchestrator reviews all four files together, validates the launch
   configuration and data shape, and records any remaining limitations before
   handoff.

## Edge cases and failure handling

- Empty, missing, or malformed JSON must produce a clear error or
  “no projects” message in the page, not a blank screen or fabricated data.
- A project with a long name, owner, status, priority, or recent activity must
  wrap without overflowing or making the card unreadable.
- Unknown future status or priority values should retain readable text and a
  neutral visual treatment rather than breaking the layout.
- Multiple projects must remain distinguishable at mobile widths; cards may
  stack, but content must not require horizontal scrolling.
- Loading, fetch failure, and empty states must be readable and accessible.
- The UI must not depend on color alone for status/risk; labels or text
  should remain visible in grayscale or for color-vision differences.
- Keyboard users need visible focus indicators and a logical reading order.
- The preview must be opened at `/index.html`; serving the app directory root
  alone risks showing a directory listing and is considered a failure.
- Port 5500 may already be occupied. Stop the conflicting process or use the
  repository-specified launch configuration consistently; do not silently
  change the documented URL without updating the launch contract.

## Validation expectations and commands

Run these checks from the repository root after integration:

```bash
python3 -m json.tool app/project-data.json >/dev/null
python3 -m json.tool .vscode/launch.json >/dev/null
grep -Fq 'Project Pulse' app/index.html
grep -Fq 'styles.css' app/index.html
grep -Fq 'project-data.json' app/index.html
grep -Fq 'project-card' app/index.html
grep -Fq 'status' app/index.html
grep -Fq 'recentActivity' app/index.html
grep -Fq 'priority' app/index.html
grep -Fq '.dashboard' app/styles.css
grep -Fq '.project-card' app/styles.css
grep -Fq 'border-radius' app/styles.css
grep -Fq 'box-shadow' app/styles.css
grep -Fq 'Run Project Pulse Dashboard' .vscode/launch.json
grep -Fq 'index.html' .vscode/launch.json
```

Use this Python check to verify the data contract, rather than only checking
that the JSON parses:

```bash
python3 - <<'PY'
import json
from pathlib import Path

data = json.loads(Path("app/project-data.json").read_text())
assert isinstance(data.get("projects"), list) and data["projects"], "projects must be non-empty"
required = {"name", "owner", "status", "recentActivity", "priority"}
for project in data["projects"]:
    assert required <= project.keys(), f"missing project fields: {project}"
    assert all(str(project[field]).strip() for field in required), project
PY
```

Perform an HTTP smoke test so browser `fetch` behavior is exercised:

```bash
python3 -m http.server 5500 --directory app >/tmp/project-pulse-http.log 2>&1 &
server_pid=$!
trap 'kill "$server_pid" 2>/dev/null || true' EXIT
sleep 1
curl --fail --silent http://localhost:5500/index.html >/dev/null
curl --fail --silent http://localhost:5500/project-data.json >/dev/null
```

Finally, launch **Run Project Pulse Dashboard** from VS Code Run and Debug,
confirm the browser opens `http://localhost:5500/index.html` and shows the
Project Pulse cards instead of a directory listing, then stop the server.
Review the browser console for failed requests or uncaught rendering errors,
and manually check keyboard focus and responsive behavior at a narrow
viewport. The repository's existing baseline check may also be run with:

```bash
bash scripts/validate-exercise.sh
```

That baseline script validates repository scaffolding and expected exercise
phrases; it is supplementary to the app-specific JSON, HTTP, and visual
checks above.

## Completion criteria

The work is complete when all four assigned files exist, the JSON and launch
configuration parse, the dashboard visibly renders multiple data-backed
cards, required fields and styling hooks are present, error/loading states
are explicit, and the VS Code launch configuration opens `index.html` on
port 5500. No unrelated files are changed, and no git staging, commit, or
push is performed as part of implementation.
