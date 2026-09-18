# Atlas Example: Recorder SDLC

This example demonstrates the Atlas pattern for hierarchical diagram navigation.

## Structure

```text
atlas/
├── atlas.manifest.json         # Atlas index and navigation metadata
├── atlas.index.html            # Static navigation page
├── 00-system.architecture.json # L0: System overview
├── 10-app.architecture.json    # L1: App module
├── 20-app-recording.workflow.json  # L2: Recording business flow
└── README.md
```

## Hierarchy

```text
L0 System Overview (architecture)
├── L1 Recorder App (architecture)
│   └── L2 Recording Workflow (workflow)
│       └── L3 Upload Sequence (sequence)
└── L1 Backend Services (architecture)
```

## Key Features Demonstrated

### 1. Drill Navigation

Components with `drill` field link to child diagrams:

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

### 2. Atlas Metadata

Diagrams include `meta.atlas` for navigation context:

```json
{
  "meta": {
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

### 3. Cross-Reference Links

Components can reference related diagrams:

```json
{
  "id": "sync",
  "links": [
    {
      "target": "backend",
      "kind": "reference",
      "href": "./11-backend.architecture.html",
      "label": "Backend sync endpoint"
    }
  ]
}
```

## Rendering

```bash
# Validate all diagrams
node bin/archify.mjs validate architecture examples/atlas/00-system.architecture.json --quality showcase --json
node bin/archify.mjs validate architecture examples/atlas/10-app.architecture.json --quality showcase --json
node bin/archify.mjs validate workflow examples/atlas/20-app-recording.workflow.json --quality showcase --json

# Render all diagrams
node bin/archify.mjs deliver architecture examples/atlas/00-system.architecture.json examples/atlas/00-system.architecture.html --quality showcase --json
node bin/archify.mjs deliver architecture examples/atlas/10-app.architecture.json examples/atlas/10-app.architecture.html --quality showcase --json
node bin/archify.mjs deliver workflow examples/atlas/20-app-recording.workflow.json examples/atlas/20-app-recording.workflow.html --quality showcase --json
```

## Navigation (Current)

Until Viewer supports native drill navigation:

1. Open `atlas.index.html` as the entry point
2. Click diagram links to navigate
3. Use browser back button or index page to return

## Navigation (Future)

When Viewer supports drill navigation:

1. Open any diagram in the atlas
2. Click components with drill icons in Semantic Passport
3. Use breadcrumb navigation to go up levels
4. Double-click drillable components to navigate
