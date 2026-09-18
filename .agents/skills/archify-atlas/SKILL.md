---
name: archify-atlas
description: Create and manage hierarchical diagram atlases with drill-down navigation. Build navigable project panoramas from L0 system overview through L5 implementation details.
---

# Archify Atlas

Create navigable diagram collections that enable progressive exploration from system overview to implementation details.

## Overview

An Atlas is a collection of Archify diagrams organized in a navigable hierarchy. Users start at the L0 system overview and can drill down into specific modules, business workflows, API sequences, and data flows.

### Hierarchy Levels

| Level | Name | Typical Type | Purpose |
|-------|------|--------------|---------|
| L0 | System | `architecture` | Overall system boundaries and major components |
| L1 | Module | `architecture` | Internal structure of a major component |
| L2 | Business | `workflow` / `lifecycle` | Business process or state machine within a module |
| L3 | Interaction | `sequence` | Cross-system API calls and message flows |
| L4 | API | `sequence` | Specific endpoint call chains |
| L5 | Data | `dataflow` + `sources` | Data lineage with code evidence |

## Directory Structure

```text
sdlc/artifacts/atlas/
  atlas.manifest.json           ← Atlas index and navigation metadata
  00-system.architecture.json   ← L0 entry point source
  00-system.architecture.html   ← L0 entry point rendered
  10-app.architecture.json      ← L1 module
  10-app.architecture.html
  11-app-recording.workflow.json    ← L2 business flow
  11-app-recording.workflow.html
  12-app-api-upload.sequence.json   ← L3/L4 API sequence
  12-app-api-upload.sequence.html
  20-backend.architecture.json  ← L1 module
  20-backend.architecture.html
  30-rag.dataflow.json          ← L5 data flow
  30-rag.dataflow.html
```

### Naming Convention

```text
<level><order>-<id>.<type>.json
<level><order>-<id>.<type>.html
```

- `<level>`: Two digits, first is hierarchy level (0-5), second for ordering within level
- `<id>`: Kebab-case identifier matching node ID in parent diagram
- `<type>`: Archify diagram type

## Atlas Manifest

The `atlas.manifest.json` file defines the atlas structure:

```json
{
  "schema_version": 1,
  "atlas_type": "atlas",
  "meta": {
    "title": "Recorder SDLC System Atlas",
    "description": "Complete system architecture with drill-down to implementation",
    "entry": "system",
    "repository": {
      "url": "https://github.com/org/recorder-sdlc",
      "revision": "abc123..."
    }
  },
  "diagrams": [
    {
      "id": "system",
      "type": "architecture",
      "level": 0,
      "title": "System Overview",
      "path": "00-system.architecture.html",
      "source_path": "00-system.architecture.json",
      "summary": "Recorder app, backend services, and RAG pipeline"
    },
    {
      "id": "app",
      "type": "architecture",
      "level": 1,
      "title": "Recorder App Module",
      "path": "10-app.architecture.html",
      "source_path": "10-app.architecture.json",
      "parent": "system",
      "parent_node": "app",
      "summary": "Mobile app internal architecture"
    },
    {
      "id": "app-recording",
      "type": "workflow",
      "level": 2,
      "title": "Recording Business Flow",
      "path": "11-app-recording.workflow.html",
      "source_path": "11-app-recording.workflow.json",
      "parent": "app",
      "parent_node": "recording-feature",
      "tags": ["recording", "audio", "business"],
      "summary": "From recording start to AI analysis completion"
    }
  ],
  "levels": {
    "L0": {
      "name": "System",
      "description": "Overall system boundaries",
      "preferred_types": ["architecture"]
    },
    "L1": {
      "name": "Module",
      "description": "Component internal structure",
      "preferred_types": ["architecture"]
    },
    "L2": {
      "name": "Business",
      "description": "Business processes and state machines",
      "preferred_types": ["workflow", "lifecycle"]
    },
    "L3": {
      "name": "Interaction",
      "description": "Cross-system communication",
      "preferred_types": ["sequence"]
    },
    "L4": {
      "name": "API",
      "description": "Specific API call chains",
      "preferred_types": ["sequence"]
    },
    "L5": {
      "name": "Data",
      "description": "Data lineage and implementation",
      "preferred_types": ["dataflow"]
    }
  }
}
```

## Authoring Drill-Down Navigation

### In Diagram JSON Source

Add `drill` to components/nodes that have child diagrams:

```json
{
  "id": "app",
  "type": "frontend",
  "label": "Recorder App",
  "drill": {
    "href": "./10-app.architecture.html",
    "diagram_type": "architecture",
    "label": "View app internals"
  }
}
```

Add `meta.atlas` to diagrams that are part of an atlas:

```json
{
  "meta": {
    "title": "Recorder App Architecture",
    "atlas": {
      "id": "recorder-sdlc",
      "diagram_id": "app",
      "parent": "./00-system.architecture.html",
      "parent_node": "app",
      "level": 1,
      "breadcrumb": [
        { "label": "System", "href": "./00-system.architecture.html" },
        { "label": "App", "href": "./10-app.architecture.html" }
      ]
    }
  }
}
```

### Cross-Reference Links

Use `links` for references that aren't drill-down relationships:

```json
{
  "id": "api-client",
  "type": "backend",
  "label": "API Client",
  "links": [
    {
      "target": "backend-api",
      "kind": "reference",
      "href": "./20-backend.architecture.html#focus=api-gateway",
      "label": "See backend API"
    }
  ]
}
```

## Generation Workflow

### 1. Create Atlas Manifest First

```bash
# Create atlas directory
mkdir -p sdlc/artifacts/atlas

# Create manifest with entry point
cat > sdlc/artifacts/atlas/atlas.manifest.json << 'EOF'
{
  "schema_version": 1,
  "atlas_type": "atlas",
  "meta": {
    "title": "Project Atlas",
    "entry": "system"
  },
  "diagrams": []
}
EOF
```

### 2. Generate L0 Entry Point

Create the system overview architecture diagram first:

```bash
# Validate and render L0
node archify/bin/archify.mjs validate architecture \
  sdlc/artifacts/atlas/00-system.architecture.json \
  --quality showcase --json

node archify/bin/archify.mjs deliver architecture \
  sdlc/artifacts/atlas/00-system.architecture.json \
  sdlc/artifacts/atlas/00-system.architecture.html \
  --quality showcase --json
```

### 3. Add Drill-Down Diagrams

For each node that needs drill-down:

1. **Update parent diagram** - Add `drill` field to the component
2. **Create child diagram** - With `meta.atlas` referencing parent
3. **Update manifest** - Add diagram entry with parent relationship
4. **Validate and deliver** - Both parent and child

### 4. Validate Atlas Integrity

```bash
# Validate all diagrams in atlas
for f in sdlc/artifacts/atlas/*.json; do
  type=$(basename "$f" | sed 's/.*\.\([a-z]*\)\.json/\1/')
  node archify/bin/archify.mjs validate "$type" "$f" --quality showcase --json
done

# Check all drill references exist
node archify/bin/archify.mjs atlas-check sdlc/artifacts/atlas/atlas.manifest.json
```

## Invariants

### Stable IDs

- Parent node ID must match child diagram's `meta.atlas.parent_node`
- Diagram `id` in manifest must match `meta.atlas.diagram_id`
- Use kebab-case IDs: `app-recording`, `backend-api`, `rag-pipeline`

### Navigation Consistency

- Every L1+ diagram must have a valid `parent` reference
- Every `drill.href` must point to an existing HTML file
- Breadcrumb must trace complete path from L0

### Diagram Limits

- L0: Maximum 12 primary components (showcase quality)
- L1-L2: Maximum 12 primary nodes per diagram
- L3-L4: Follow sequence diagram limits (reasonable participant count)
- L5: Follow dataflow limits (2-5 stages)

### File Organization

- All atlas files in one directory
- Relative paths within atlas (no absolute paths)
- Manifest at atlas root

## Agent Interaction Pattern

When user asks to drill into a component:

1. **Check manifest** - Does child diagram exist?
2. **If exists** - Open the child diagram
3. **If not exists** - Generate based on:
   - Parent node context
   - User intent (business flow? API? data?)
   - Appropriate diagram type for the level
4. **Update manifest** - Add new diagram entry
5. **Update parent** - Add `drill` field if not present

### Example Prompts

User: "进入 recorder-app 模块看内部结构"

Agent actions:
1. Read `atlas.manifest.json`
2. Find diagram with `parent_node: "recorder-app"`
3. If found: Open `10-app.architecture.html`
4. If not found: Generate L1 architecture diagram, update manifest

User: "展示录音功能的业务流程"

Agent actions:
1. Identify current context (app module)
2. Check for existing L2 workflow
3. Generate `11-app-recording.workflow.json` if needed
4. Add drill reference to parent, update manifest

## Index Page Generation

For immediate navigation without Viewer drill support, generate a static index:

```html
<!-- atlas.index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>Project Atlas</title>
  <style>
    body { font-family: system-ui; max-width: 800px; margin: 2rem auto; }
    .level { margin-left: 1.5rem; }
    a { color: #0066cc; }
    .type { color: #666; font-size: 0.9em; }
  </style>
</head>
<body>
  <h1>Project Atlas</h1>
  <nav>
    <div class="level-0">
      <a href="00-system.architecture.html">System Overview</a>
      <span class="type">(architecture)</span>
      <div class="level">
        <a href="10-app.architecture.html">Recorder App</a>
        <span class="type">(architecture)</span>
        <div class="level">
          <a href="11-app-recording.workflow.html">Recording Flow</a>
          <span class="type">(workflow)</span>
        </div>
      </div>
      <div class="level">
        <a href="20-backend.architecture.html">Backend Services</a>
        <span class="type">(architecture)</span>
      </div>
    </div>
  </nav>
</body>
</html>
```

Generate this from manifest:

```bash
node archify/bin/archify.mjs atlas-index \
  sdlc/artifacts/atlas/atlas.manifest.json \
  sdlc/artifacts/atlas/atlas.index.html
```

## Future: Viewer Drill Support

When Viewer supports drill navigation natively:

1. **Passport drill button** - Click component → "Enter" button in Semantic Passport
2. **Double-click drill** - Double-click component with `drill` → navigate
3. **Breadcrumb nav** - Show breadcrumb from `meta.atlas.breadcrumb`
4. **Back navigation** - "↑ Parent" button when `meta.atlas.parent` exists

Until then, use the index page or embed navigation hints in diagram cards.
