# Routing Quality Engineering

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

TDD-based methodology for systematically improving LLM skill routing accuracy. Takes your existing Agent from unreliable routing to 95%+ accuracy through a structured 4-layer approach.

Works with **any planner + skill architecture** where an LLM routes user intent to skill descriptions. Platform-agnostic: usable in Claude Code, Cursor, Windsurf, Gemini CLI, Codex CLI, or any tool that supports [SKILL.md](https://agentskills.io).

## Overview

```
Phase 0 → Layer 1 → Layer 2 → Layer 3 → Layer 4
discover   golden     ambiguity   boundary    integration
           cases      resolution  arbitration testing
```

Plus `/fixing-routing` for single-bug hotfixes at any time.

| Skill | Purpose |
|-------|---------|
| `/routing-layer1` | Golden cases — lock each skill's core scenarios (target: 95%+) |
| `/routing-layer2` | Ambiguity resolution — handle "looks like it could be either" inputs |
| `/routing-layer3` | Boundary arbitration — settle competing skill pairs with fallback rules |
| `/routing-layer4` | Integration testing — full regression across all layers (target: 95%+ overall, 98%+ golden) |
| `/fixing-routing` | 6-phase single-bug hotfix: evidence → root cause → spec → fix → test → record |

## Prerequisites

This plugin assumes you already have:

- A **planner** (LLM that routes user intent to skills)
- **Skills with `description` fields** that the planner reads
- A way to **run routing tests** (test file that sends inputs and checks which skill was selected)

If you're building an Agent from scratch, consider using [routing-setup-skills](https://github.com/guangyuniu8023-arch/routing-setup-skills) first to scaffold the Agent skeleton and build each skill, then come back here for quality improvement.

## Installation

### 1. Claude Code (Plugin mode)

```bash
git clone https://github.com/guangyuniu8023-arch/routing-quality-skills.git

# Launch Claude Code with the plugin
claude --plugin-dir ./routing-quality-skills
```

All 5 skills are immediately available as slash commands.

### 2. Cursor

```bash
git clone https://github.com/guangyuniu8023-arch/routing-quality-skills.git

# Copy skills into Cursor's skill directory
cp -r routing-quality-skills/skills/* .cursor/skills/

# Copy docs into your project (skills reference these docs)
cp -r routing-quality-skills/docs your-project/docs/routing-quality/
```

Then type `/routing-layer1` (or any skill name) in Cursor chat.

### 3. Windsurf

```bash
git clone https://github.com/guangyuniu8023-arch/routing-quality-skills.git

# Copy skills and docs into your project
cp -r routing-quality-skills/skills .windsurf/skills/
cp -r routing-quality-skills/docs your-project/docs/routing-quality/
```

Use `/routing-layer1` in Windsurf chat.

### 4. Gemini CLI

```bash
git clone https://github.com/guangyuniu8023-arch/routing-quality-skills.git

# Copy into your Gemini skills directory
cp -r routing-quality-skills/skills ~/.gemini/skills/
cp -r routing-quality-skills/docs your-project/docs/routing-quality/
```

### 5. Codex CLI / Other AI tools

SKILL.md is an [open standard](https://agentskills.io). Copy `skills/` to wherever your tool loads skills from, and place `docs/` in your project:

```bash
git clone https://github.com/guangyuniu8023-arch/routing-quality-skills.git

# Adapt paths to your tool
cp -r routing-quality-skills/skills /path/to/your/tool/skills/
cp -r routing-quality-skills/docs your-project/docs/routing-quality/
```

### 6. Manual (no tool integration)

You can use the methodology manually by reading the docs directly:

1. Read `docs/README.md` for the flow selection guide
2. Follow the relevant doc (e.g., `docs/layer1-golden-cases.md`) step by step
3. Apply the instructions to your project using any LLM assistant

## Usage

### Systematic improvement (Layer 1-4)

**Step 1 — Discovery (Phase 0)**

If `.routing-quality/config.md` doesn't exist yet, Layer 1 will auto-run Phase 0 first. This discovers your project structure, collects Baseline Intent for each skill, and generates the config file that all layers share.

**Step 2 — Golden cases (Layer 1)**

```
/routing-layer1
```

- Generates 3-5 golden test cases per skill from Baseline Intent (not from descriptions)
- Runs tests, reads descriptions only after failures, makes minimal fixes
- Iterates until >= 95% accuracy
- Saves snapshot to `.routing-quality/snapshots/v1-golden/`

**Step 3 — Ambiguity (Layer 2)**

```
/routing-layer2
```

- Builds ambiguity matrix (media type x intent type)
- Tests edge cases that could route to multiple skills
- Fixes descriptions, saves snapshot v2

**Step 4 — Boundary arbitration (Layer 3)**

```
/routing-layer3
```

- Identifies competing skill pairs
- Defines fallback rules for undecidable cases
- Tests boundary cases, saves snapshot v3

**Step 5 — Integration (Layer 4)**

```
/routing-layer4
```

- Merges all test cases from Layer 1-3
- Runs full regression (target: >= 95% overall, >= 98% golden)
- Generates final report

### Single bug hotfix

```
/fixing-routing
```

6-phase workflow: collect evidence → root cause (A-F taxonomy) → write spec → execute → test → record. When systemic issues are found, recommends switching to the Layer 1-4 flow.

## Key Principles

- **Iron Law**: Don't read descriptions before writing tests. Don't modify descriptions without a failing test.
- **Minimal change**: Only change what's needed to make failing cases pass. No drive-by rewrites.
- **REFACTOR**: After all green, clean up descriptions (remove redundancy, check cross-skill consistency, control length) while keeping tests green.
- **Platform-agnostic**: All docs are tool-independent. Works with any LLM coding assistant.

## Project Structure

```
routing-quality-skills/
├── .claude-plugin/
│   └── plugin.json              # Claude Code plugin metadata
├── skills/
│   ├── routing-layer1/          # Layer 1: Golden cases
│   │   └── SKILL.md
│   ├── routing-layer2/          # Layer 2: Ambiguity resolution
│   │   └── SKILL.md
│   ├── routing-layer3/          # Layer 3: Boundary arbitration
│   │   └── SKILL.md
│   ├── routing-layer4/          # Layer 4: Integration testing
│   │   └── SKILL.md
│   └── fixing-routing/          # Single bug hotfix
│       └── SKILL.md
├── docs/
│   ├── README.md                # Flow selection guide
│   ├── phase0-discovery.md      # Project config + Baseline Intent
│   ├── layer1-golden-cases.md   # TDD: RED → GREEN → REFACTOR
│   ├── layer2-ambiguity-cases.md
│   ├── layer3-boundary-cases.md
│   ├── layer4-integration.md
│   ├── fixing-routing.md        # 6-phase hotfix
│   └── references/
│       ├── root-cause-taxonomy.md   # A-F failure classification
│       ├── description-rules.md     # 4 rules for description changes
│       └── templates.md             # Evidence cards, specs, snapshots
├── LICENSE
└── README.md                    # This file
```

## License

[MIT](LICENSE)
