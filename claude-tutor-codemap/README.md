# Claude Tutor — Interactive Code Map

A single-file interactive visualization of the [claude-tutor](https://github.com/WorldWideStudios/claude-tutor) codebase architecture. Open `claude-tutor-codemap.html` in any browser — no build step, no dependencies.

## What It Shows

The code map renders all 27 source files as draggable nodes on an SVG canvas, organized into 7 architectural layers with dependency arrows between them.

### Layers

| Color | Layer | Files |
|-------|-------|-------|
| Blue | CLI & Entry | `index.ts`, `preflight.ts`, `update.ts`, `important.ts` |
| Purple | Authentication & API | `auth.ts`, `logging.ts` |
| Green | State & Data | `storage.ts`, `types.ts` |
| Orange | AI & Intelligence | `agent.ts`, `curriculum.ts`, `system-prompt.ts`, `questions.ts`, `golden-code.ts`, `tools.ts` |
| Cyan | Terminal UI | `display.ts`, `input.ts`, `mode.ts` |
| Yellow | Core Loop | `tutor-loop.ts`, `project-lifecycle.ts`, `git.ts`, `utils.ts` |
| Red | Loop Managers | `GoldenCodeManager.ts`, `CommandExecutor.ts`, `InputHandler.ts`, `agent-caller.ts`, `SegmentLifecycleManager.ts`, `ScrollableViewer.ts` |

## Features

### Node Graph
- **Click a node** to open a floating detail panel with the file's purpose, key exports, code snippets, and dependency lists
- **Drag nodes** to rearrange the layout
- **Hover** to highlight all incoming and outgoing connections
- **Double-click** a node to open the comment modal directly

### Detail Panel
Each node's detail panel shows:
- **Layer** badge with color
- **Purpose** — what the file does
- **Key Exports** — the public API
- **Code** — syntax-highlighted TypeScript snippets showing function signatures, class structures, schemas, and patterns
- **Imports From / Imported By** — clickable dependency links that navigate between nodes

### View Controls
- **Presets** — quickly filter to Full System, AI & Intelligence, Terminal UI, Core Loop, or State & Data Flow
- **Layer toggles** — show/hide any of the 7 module groups independently
- **Data flow overlays** — toggle animated paths showing the Initialization Flow (green) or Learning Loop Flow (orange)
- **Search** (Cmd+K) — find files by name, function, or concept
- **Zoom** — scroll wheel, +/- buttons, fit-to-view, reset

### Comments & Prompt
- Add comments on any node via the detail panel or double-click
- Comments appear in the sidebar and auto-generate a natural-language prompt
- Copy the prompt to paste back into Claude for targeted feedback

### Minimap
A minimap in the top-right corner shows the full graph with a viewport indicator.

## How It Works

### Architecture

The HTML file is entirely self-contained (~1900 lines). No external dependencies.

```
HTML structure:
  Topbar          — stats, tech badges, search
  Sidebar         — presets, layer toggles, flow overlays, comments
  Canvas Area     — SVG graph + floating detail panel + minimap + zoom controls
  Prompt Bar      — collapsible prompt output with copy button
  Comment Modal   — overlay for adding notes to nodes
```

### Rendering

The graph is rendered as SVG inside a `<g>` transform group. `render()` clears and rebuilds all nodes and connections from the `NODES` and `CONNECTIONS` arrays on every state change. The transform group handles pan (`translate`) and zoom (`scale`).

### Event Handling

All mouse interaction uses **event delegation** on the `<svg>` element rather than per-node listeners. A `findNodeId(el)` helper walks up the DOM from the event target to find the `data-node` attribute. This is critical because `render()` destroys and recreates SVG elements — delegation on the stable parent avoids stale-reference bugs.

```
graphEl.addEventListener('click', (e) => {
  const nodeId = findNodeId(e.target);  // walk up to [data-node]
  if (nodeId && !state.dragMoved) selectNode(nodeId);
  else if (!nodeId) closeDetail();
});
```

### State

A single `state` object drives all rendering:

```javascript
const state = {
  layers: { cli: true, auth: true, ... },  // layer visibility
  selectedNode: null,      // currently selected node id
  hoveredNode: null,       // currently hovered node id
  comments: [],            // user comments array
  showFlowInit: false,     // init flow overlay
  showFlowLoop: false,     // loop flow overlay
  zoom: 1,                 // current zoom level
  panX: 0, panY: 0,       // current pan offset
  ...
};
```

### Data Model

Each node carries its metadata inline:

```javascript
{ id: 'agent', label: 'agent.ts', subtitle: 'src/agent.ts',
  layer: 'ai', x: 580, y: 160,
  desc: 'Claude SDK integration with streaming...',
  exports: ['runAgentTurn()', ...],
  snippets: [{ title: 'Agent Turn', code: '...' }] }
```

Connections are directional edges:

```javascript
{ from: 'agent', to: 'tools', type: 'dep' }
```

Flow overlays are ordered arrays of node ids:

```javascript
const FLOW_INIT = ['index', 'update', 'storage', 'preflight', ...];
const FLOW_LOOP = ['inputhandler', 'cmdexec', 'agentcaller', ...];
```

### Code Snippets

Snippets use inline HTML with span classes for syntax highlighting:
- `.kw` — keywords (red) — `export`, `async`, `function`, `const`, `if`
- `.fn` — function names (purple) — `runAgentTurn`, `loadConfig`
- `.tp` — types (blue) — `Promise`, `Curriculum`, `string`
- `.str` — strings (light blue) — `'tutor'`, `'build'`
- `.const` — constants/params (orange) — `messages`, `systemPrompt`
- `.cm` — comments (dim gray) — `// Fail-silent`

## About Claude Tutor

Claude Tutor is an AI-powered CLI tool that teaches software engineering through hands-on TypeScript coding. It combines:

- **Dynamic curriculum generation** — Claude creates personalized 4-6 segment learning paths
- **Three interaction modes** — Tutor (character-by-character Typer Shark), Block (free-form), Discuss (Q&A)
- **Tool-enabled AI** — syntax checking, code review, git operations, segment completion
- **Persistent state** — resume from the exact point you left off

Built with TypeScript, Anthropic Claude SDK, Zod, Commander.js, and Chalk.
