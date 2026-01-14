# Exciting Blog (Fuwari Template)

## Project Overview

This project is a static blog based on the **Fuwari** template, built with **Astro**, **Svelte**, and **Tailwind CSS**. It is configured to deploy to **Cloudflare Pages**.

The blog features a responsive design, light/dark mode, search functionality (via Pagefind), and extended Markdown capabilities.

## Tech Stack

*   **Framework:** [Astro](https://astro.build/) (v5)
*   **UI Framework:** [Svelte](https://svelte.dev/) (v5)
*   **Styling:** [Tailwind CSS](https://tailwindcss.com/) (v3)
*   **Deployment Adapter:** [Cloudflare](https://docs.astro.build/en/guides/deploy/cloudflare/)
*   **Search:** [Pagefind](https://pagefind.app/)
*   **Comments:** [Giscus](https://giscus.app/)
*   **Package Manager:** `pnpm`

## Key Files & Directories

*   **`astro.config.mjs`**: Main Astro configuration, including integrations (Tailwind, Svelte, Expressive Code, Swup) and Markdown plugins.
*   **`src/config.ts`**: Site-specific configuration (Title, Author, Navigation, Social Links, License). **Edit this to customize the blog.**
*   **`src/content/posts/`**: Directory containing blog posts in Markdown format.
*   **`src/content/config.ts`**: Defines the data collections and schema (Zod) for posts.
*   **`src/pages/`**: Astro pages and routing logic.
*   **`src/components/`**: Reusable Astro and Svelte components.
*   **`src/layouts/`**: Page layouts (e.g., `MainGridLayout`, `BlogPostLayout`).
*   **`scripts/new-post.js`**: Script to generate new blog post files.

## Development Workflow

### Prerequisites

*   Node.js (>= 20)
*   pnpm (>= 9)

### Core Commands

| Command | Description |
| :--- | :--- |
| `pnpm dev` | Starts the local development server at `localhost:4321`. |
| `pnpm build` | Builds the production site to the `dist/` directory and indexes search. |
| `pnpm preview` | Previews the production build locally. |
| `pnpm new-post <name>` | Creates a new blog post file in `src/content/posts/`. |
| `pnpm check` | Runs Astro check to validate code. |
| `pnpm format` | Formats code using Biome. |
| `pnpm lint` | Lints code using Biome. |

## Configuration

To customize the blog, primarily edit **`src/config.ts`**.

### Site Config (`siteConfig`)
*   `title`: Blog title.
*   `subtitle`: Blog subtitle.
*   `lang`: Main language (e.g., `zh_CN`, `en`).
*   `themeColor`: Hue value for the primary color.
*   `banner`: Configuration for the top banner image.

### Profile Config (`profileConfig`)
*   `avatar`: Path to avatar image.
*   `name`: Author name.
*   `bio`: Author bio.
*   `links`: Social media links.

### Navigation (`navBarConfig`)
*   Define links for the top navigation bar.

## Content Management

### Creating Posts

Run the helper script:
```bash
pnpm new-post my-new-post
```
This creates a new markdown file in `src/content/posts/` with the current year (e.g., `src/content/posts/2026/my-new-post.md`).

### Frontmatter Schema

Each post requires the following YAML frontmatter:

```yaml
---
title: My Post Title
published: 2026-01-14 # Date
description: A short description.
image: ./cover.jpg # Optional cover image
tags: [Tag1, Tag2]
category: CategoryName
draft: false # Set to true to hide from production
---
```

### Extended Markdown

The project supports:
*   **Admonitions**: `:::note`, `:::tip`, etc.
*   **GitHub Cards**: `::github{repo="owner/repo"}`
*   **Expressive Code**: Enhanced code blocks with line numbers and copy buttons.
*   **Math**: KaTeX support for equations.
