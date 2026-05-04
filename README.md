# Vaadin Claude Plugin

Skills and tools for building high-quality Vaadin 25 applications with Java/Flow.

## Overview

This plugin provides Claude with deep knowledge about Vaadin 25 development patterns, best practices, and common pitfalls. It covers layouts, responsive design, component composition, forms, testing, data binding, and reactive state management. The plugin also integrates with the Vaadin MCP server for real-time documentation lookups.

## Skills

| Skill | Triggers when you ask about... |
|-------|-------------------------------|
| **theming** | Theme selection (Aura vs Lumo), color customization, dark mode, design tokens, component variants, utility classes |
| **frontend-design** | Visual design, styling, animations, polished component compositions, making views look distinctive |
| **vaadin-layouts** | HorizontalLayout, VerticalLayout, alignment, spacing, padding, flex-grow, layout sizing issues |
| **responsive-layouts** | Mobile support, breakpoints, media queries, container queries, responsive utility classes |
| **reusable-components** | Custom components, Composite, component APIs, HasValue, HasComponents, encapsulation |
| **forms-and-validation** | Binder, form fields, validation, converters, required fields, cross-field validation |
| **ui-unit-testing** | Browserless tests, BrowserlessTest, component testers, fast view testing |
| **testbench-testing** | End-to-end tests, TestBench, page objects, ElementQuery, browser testing |
| **data-providers** | Grid data binding, lazy loading, filtering, sorting, Spring Pageable integration |
| **third-party-components** | Web Component wrapping, React component integration, @Tag, @NpmPackage, ReactAdapterComponent, DOM events, property sync |
| **signals** | Reactive state, ValueSignal, NumberSignal, effects, computed signals, shared state |
| **views-and-navigation** | Views, routes, navigation, AppLayout, router layouts, SideNav, URL parameters, master-detail |

## MCP Integration

This plugin includes the Vaadin MCP server, which provides:

- `search_vaadin_docs` — search the official Vaadin documentation
- `get_component_java_api` — get Java API docs for any Vaadin component
- `get_component_styling` — get styling/theming docs for components
- `get_full_document` — retrieve complete documentation pages
- `get_vaadin_version` — check the latest Vaadin version

## Target Version

- **Vaadin 25** (Java/Flow development model)
- **Spring Boot 4** / **JUnit 6** compatible

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) version 1.0.33 or later (run `claude --version` to check)

## Installation

### Option 1: Install from marketplace (recommended)

1. Add the marketplace:

   ```shell
   /plugin marketplace add vaadin/claude-plugin
   ```

2. Install the plugin:

   ```shell
   /plugin install vaadin-claude@vaadin-claude-plugin
   ```

   Or open the plugin manager with `/plugin`, go to the **Discover** tab, and select **vaadin-claude** to install interactively.

### Option 2: Install from local directory

Clone the repository and load it directly:

```bash
git clone https://github.com/vaadin/claude-plugin.git
claude --plugin-dir ./claude-plugin
```

## Usage

The skills and MCP tools activate automatically based on conversation context. Ask about any Vaadin development topic and Claude will use the relevant skill along with the Vaadin MCP server for up-to-date documentation.

### Examples

- "Set up Aura theme with a custom color palette"
- "Create a responsive master-detail view"
- "Add form validation with Binder"
- "Write UI unit tests for my view"
- "Wrap a third-party Web Component for use in Flow"

### Updating

If you installed from the marketplace, update by running:

```shell
/plugin marketplace update vaadin-claude-plugin
```

Or enable auto-updates in the plugin manager under the **Marketplaces** tab.
