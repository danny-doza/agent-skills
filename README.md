# Agent Skills & Plugins

A curated collection of reusable **Claude/Cowork skills and plugins** designed to extend agent capabilities with structured, task-specific intelligence.

This repository provides both:
- **Standalone skills** (prompt-driven capabilities)
- **Plugin packages** (deployable bundles for marketplace/CLI use)

## Installation

### Option 1: Install a Skill (Recommended)

Use the Skills CLI to install a specific skill directly from this repository:

```bash
npx skills add danny-doza/agent-skills --skill <skill-name>
```

### Option 2: Install via Plugin Marketplace

If using Claude/Cowork plugin system:

```bash
/plugin marketplace add danny-doza/agent-skills
/plugin install <plugin-name>@doza-skills
```

## Skills Available

### anki

Provides seamless integration with Anki via AnkiConnect, enabling the assistant to interact directly with your Anki collection.