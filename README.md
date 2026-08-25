<!-- Explains how to install and use the graphRun model-authoring agent skill. -->
# graphRun Model Authoring Skill

An agent skill for inspecting, authoring, validating, simulating, mapping to implementation, and explaining distributed-system models through the graphRun MCP.

The skill teaches coding agents the safe graphRun authoring workflow: live discovery, graph and contract operations, reference bindings, compare-and-swap tokens, executable scenarios, deployment views, validation, and failure recovery.

## Install

Install into the current project for Codex:

```bash
npx skills add graphrun/skills \
  --skill author-graphrun-models \
  --agent codex
```

Install globally:

```bash
npx skills add graphrun/skills \
  --skill author-graphrun-models \
  --agent codex \
  --global
```

List the skills discoverable in this repository without installing:

```bash
npx skills add graphrun/skills --list
```

Update an existing installation:

```bash
npx skills update author-graphrun-models
```

## Requirements

- A coding agent supported by the `skills` CLI.
- A configured graphRun MCP connection with access to `/mcp/v2`.
- A graphRun Agent Access Key whose grant covers the diagrams and tools needed for the task.

Installing this repository adds the agent instructions only. It does not configure the MCP endpoint or credentials.

## Skill contents

```text
.agents/skills/author-graphrun-models/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── branding.md
    ├── contracts.md
    ├── deployment.md
    ├── diagnostics.md
    ├── graph-authoring.md
    ├── implementation-mapping.md
    ├── journeys.md
    └── workflow.md
```

The main skill contains the mandatory workflow and safety rules. Reference files are loaded only when their authoring surface is relevant.
