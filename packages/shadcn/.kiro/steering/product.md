# Product Overview

shadcn is a CLI tool for adding UI components to projects. It enables developers to initialize projects with a design system and add pre-built components from registries.

## Core Functionality

- **init**: Initialize dependencies, configure Tailwind CSS, add utility functions, and set up CSS variables
- **create**: Create new projects with custom design systems via an interactive website
- **add**: Add individual components to projects with automatic dependency installation
- **search**: Search available components across registries
- **view**: View component details and code
- **diff**: Compare local components with registry versions
- **migrate**: Migrate components between versions or styles
- **build**: Build registry files for custom component registries
- **mcp**: Model Context Protocol integration for AI assistants

## Key Concepts

- **Registries**: Component sources (built-in or custom) that provide reusable UI components
- **Styles**: Visual themes (e.g., "default", "new-york") that define component appearance
- **Base Colors**: Color schemes (neutral, slate, zinc, stone, gray) for theming
- **CSS Variables**: Optional theming approach using CSS custom properties
- **components.json**: Configuration file that stores project settings, aliases, and registry information
