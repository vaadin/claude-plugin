# Vaadin Claude Plugin

> [!IMPORTANT]
> **This marketplace is deprecated and will eventually be retired.**
> Please switch to the new [vaadin/agent-marketplace](https://github.com/vaadin/agent-marketplace),
> which serves the same skills and MCP tools for both **Claude Code** and **Codex**.
>
> ```shell
> /plugin marketplace add vaadin/agent-marketplace
> /plugin install vaadin-skills@vaadin-marketplace
> ```
>
> If you previously installed from this repository, remove it and re-add the new
> marketplace:
>
> ```shell
> /plugin marketplace remove vaadin-marketplace
> /plugin marketplace add vaadin/agent-marketplace
> ```

Skills and tools for building high-quality Vaadin 25 applications with Java/Flow.

## Overview

This plugin provides Claude with deep knowledge about Vaadin 25 development patterns, best practices, and common pitfalls. It covers layouts, responsive design, component composition, forms, testing, data binding, and reactive state management. The plugin also integrates with the Vaadin MCP server for real-time documentation lookups.

## Skills

The skills and MCP server distributed by this marketplace live in the [vaadin/agent-skills](https://github.com/vaadin/agent-skills) repository. See that repository for the full list of skills and their documentation.

## MCP Integration

The plugin bundles the Vaadin MCP server for real-time documentation lookups. See the [vaadin/agent-skills](https://github.com/vaadin/agent-skills) repository for details on the available MCP tools.

## Target Version

- **Vaadin 25** (Java/Flow development model)
- **Spring Boot 4** / **JUnit 6** compatible

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) version 1.0.33 or later (run `claude --version` to check)

## Installation

Install from the new [vaadin/agent-marketplace](https://github.com/vaadin/agent-marketplace) (recommended):

1. Add the marketplace:

   ```shell
   /plugin marketplace add vaadin/agent-marketplace
   ```

2. Install the plugin:

   ```shell
   /plugin install vaadin-skills@vaadin-marketplace
   ```

   Or open the plugin manager with `/plugin`, go to the **Discover** tab, and select **vaadin-skills** to install interactively.

> The legacy `vaadin/claude-plugin` marketplace still works for now, but is deprecated and will eventually be retired. Prefer the marketplace above.

## Usage

The skills and MCP tools activate automatically based on conversation context. Ask about any Vaadin development topic and Claude will use the relevant skill along with the Vaadin MCP server for up-to-date documentation.

### Examples

- "Set up Aura theme with a custom color palette"
- "Create a responsive master-detail view"
- "Add form validation with Binder"
- "Write UI unit tests for my view"
- "Wrap a third-party Web Component for use in Flow"

### Updating

Update the marketplace catalog by running:

```shell
/plugin marketplace update vaadin-marketplace
```

Or enable auto-updates in the plugin manager under the **Marketplaces** tab.
