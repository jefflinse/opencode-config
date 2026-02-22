---
name: tui-prototype
description: Fast-track TUI prototyping workflow using tui-engineer for Bubble Tea applications, with TUI-specific scaffolding, build, and smoke testing
---

## What This Skill Does

Defines a rapid prototyping workflow specifically for terminal user interface applications built with the Charmbracelet ecosystem (Bubble Tea, Lipgloss, Bubbles). Routes implementation work to the tui-engineer agent instead of the generic builder, scaffolds a TUI-appropriate project structure, and includes TUI-specific verification (run in a real terminal, not a browser).

## When To Use

Use this when:
- Prototyping a new TUI application or interactive terminal tool
- Building a proof of concept for a Bubble Tea-based interface
- Exploring a TUI layout, component composition, or interaction pattern
- Spiking a TUI that integrates with an external service (LLM API, REST API, database, etc.)
- The user wants to quickly prove out a terminal-based idea

Do NOT use this when:
- Building a production-ready TUI feature (use feature-workflow with tui-engineer assigned in the plan)
- The application has no terminal UI component (use rapid-prototype instead)
- The work is purely CLI flags and output with no interactive TUI (use rapid-prototype instead)

## Workflow

### Phase 1: Quick Plan
**Agent**: planner

1. Clarify the goal — what TUI experience are we trying to prove out?
2. Identify the core screens or views needed (keep it to 1-3 for a prototype)
3. Identify external integrations (LLM APIs, databases, HTTP services) and their role
4. List the steps in implementation order
5. Flag hard blockers (missing dependencies, API keys needed, unclear UX requirements)

Keep the plan lightweight. For a TUI prototype, the plan should answer: what screens exist, what can the user do on each screen, and what data flows between them.

**Gate**: Plan must be reviewed and approved before proceeding.

### Phase 2: Branch
**Agent**: git-workflow

1. Create a prototype branch: `proto/<description>`

### Phase 3: Scaffold
**Agent**: tui-engineer

Create the minimal project structure for a Bubble Tea application:

1. Initialize `go.mod` if it doesn't exist
2. Create the directory layout:
   ```
   cmd/<app>/main.go          # tea.NewProgram entry point
   internal/tui/model.go      # Root model (Init, Update, View)
   internal/tui/styles.go     # Lipgloss style definitions
   ```
3. Add Charmbracelet dependencies: `go get github.com/charmbracelet/bubbletea github.com/charmbracelet/lipgloss github.com/charmbracelet/bubbles`
4. Wire up a minimal `main.go` → root model → empty view that compiles and runs
5. Verify: `go build ./...` succeeds and running the binary shows a terminal UI (even if empty)

For prototypes, keep the structure flat. Don't create `internal/tui/components/`, `internal/tui/screens/`, etc. unless there are genuinely multiple distinct components. Start with everything in `internal/tui/` and split later if needed.

If the prototype integrates with an external service (e.g., an LLM API), also create:
   ```
   internal/client/client.go  # API client with minimal interface
   ```

### Phase 4: Build
**Agent**: tui-engineer

Implement the prototype TUI following the plan:

1. Build the core screens/views identified in the plan
2. Use `bubbles` components wherever possible — don't write custom components for a prototype
3. Wire up keyboard navigation and basic interaction
4. For external integrations (LLM APIs, HTTP services):
   - Implement the happy-path client (connect, send request, get response)
   - For streaming APIs, implement the `tea.Cmd` → `tea.Msg` pattern to feed data into the model incrementally
   - Basic error handling: display the error in the TUI, don't crash
5. Run `go build ./...` and `go vet ./...` after each significant change
6. Focus on the core interaction loop working end-to-end, even if the UI is rough

Prototype priorities (in order):
1. The app launches and displays something
2. The core interaction works (user input → processing → visible result)
3. The layout is reasonable (not broken, content visible)
4. Edge cases are handled (empty state, error state)
5. The UX is polished (last priority — skip for prototype if time is tight)

### Phase 5: Smoke Test
**Agent**: tui-engineer

1. Run `go build ./...` — must compile cleanly
2. Run `go vet ./...` — no issues
3. Run `go test ./...` if tests already exist — don't break existing tests
4. Run the binary in a terminal and verify:
   - The TUI renders without visual corruption
   - Basic keyboard interaction works (navigation, input, quit)
   - The core use case works end-to-end (e.g., send a prompt, see a response)
   - Window resize doesn't panic (send `tea.WindowSizeMsg` mentally — check that the View function uses width/height from state, not hardcoded values)

TUI-specific things to watch for:
- Viewport overflow (content wider than terminal)
- Missing `tea.WindowSizeMsg` handling
- Hardcoded dimensions instead of dynamic sizing
- Spinner or loading state that never resolves
- Key bindings that conflict or don't work

### Phase 6: Wrap Up
**Agent**: git-workflow

1. Commit with a clear message: `proto: <description>`
2. Optionally create a lightweight PR if the user wants one
3. Present a brief summary:
   - What was built (screens, components, integrations)
   - What works (core interactions, happy path)
   - Known gaps and shortcuts taken (missing error handling, hardcoded values, unstyled components)
   - External dependencies and configuration needed (API keys, environment variables)
   - Recommended next steps if the prototype proves viable

## Principles

- **Screen-first thinking**: Start with what the user sees and works backward to the data. A TUI prototype that looks right and handles the core interaction is more convincing than clean architecture with a broken view
- **One screen at a time**: Get one screen working end-to-end before starting the next. A prototype with one polished screen beats three half-built ones
- **Real terminal, real test**: Bubble Tea behavior is not fully captured by unit tests. The smoke test is running the binary and interacting with it. If you can't run it, you can't verify it
- **Bubbles first**: Use existing `bubbles` components before writing custom ones. A `viewport.Model` with basic styling is better than a custom scrolling implementation for a prototype
- **Streaming is a first-class concern**: If the prototype involves streaming data (LLM responses, log tails, real-time updates), the streaming → `tea.Msg` → `Update` → `View` pipeline is the core architecture, not an afterthought. Get it working early
