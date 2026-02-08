# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Platform Pragmatist is a blog/content site built on **Astroplate** (Astro v5 + Tailwind CSS v4 + TypeScript). It uses Astro's content collections for Markdown/MDX content, React for interactive components, and deploys to Netlify.

## Commands

- `yarn dev` — Start dev server (runs theme generator in watch mode + JSON generation + Astro dev concurrently)
- `yarn build` — Production build (theme generator + JSON generation + Astro build)
- `yarn preview` — Preview production build locally
- `yarn check` — Run Astro type checking (`astro check`)
- `yarn format` — Format `src/` with Prettier (includes astro and tailwind plugins)

## Architecture

### Content System

Content lives in `src/content/` with collections defined in `src/content.config.ts`. Each collection uses Astro's `glob` loader and Zod schemas:

- **blog/** — Blog posts (title, author, categories, tags, date, draft)
- **authors/** — Author profiles with social links
- **homepage/** — Homepage banner and features (file prefixed with `-`)
- **about/**, **contact/** — Single-page content (use `-index.md`)
- **pages/** — Generic pages (privacy policy, elements)
- **sections/** — Reusable sections (call-to-action, testimonial)

Files prefixed with `-` (e.g., `-index.md`) are list/index pages fetched via `getListPage()`. Non-prefixed files are individual entries fetched via `getSinglePage()`. Both helpers are in `src/lib/contentParser.astro`.

### Configuration

JSON config files in `src/config/` control site behavior — not code changes:

- **config.json** — Site metadata, settings (search, pagination, dark mode, disqus, GTM), navigation button, contact form
- **menu.json** — Main and footer navigation links
- **social.json** — Social media links with react-icons icon names
- **theme.json** — Colors (light/dark), font families, font size scale

### Build-time Code Generation

Two scripts run before Astro builds:

1. **`scripts/themeGenerator.js`** — Reads `src/config/theme.json` and generates `src/styles/generated-theme.css` with CSS custom properties (`@theme` block). In dev mode, watches for changes. **Do not edit `generated-theme.css` directly.**
2. **`scripts/jsonGenerator.js`** — Reads blog markdown files, extracts frontmatter, and writes `.json/posts.json` and `.json/search.json` for client-side search.

### Path Aliases (tsconfig.json)

- `@/components/*` → `src/layouts/components/*`
- `@/shortcodes/*` → `src/layouts/shortcodes/*`
- `@/helpers/*` → `src/layouts/helpers/*`
- `@/partials/*` → `src/layouts/partials/*`
- `@/*` → `src/*`

### Layout Structure

`src/layouts/` is organized into layers:

- **Base.astro** — HTML shell (head meta, fonts, GTM, header/footer, search modal)
- **PostSingle.astro** — Blog post detail layout
- **components/** — Reusable Astro components (BlogCard, Pagination, ImageMod, etc.)
- **partials/** — Page sections (Header, Footer, CallToAction, Testimonial, PostSidebar)
- **helpers/** — React interactive components (SearchModal, Announcement, DynamicIcon, Disqus)
- **shortcodes/** — React components auto-imported into MDX (Button, Accordion, Notice, Video, Youtube, Tabs/Tab)

### Styling

Tailwind CSS v4 with `@tailwindcss/vite` plugin. Styles in `src/styles/`:

- `main.css` — Entry point; imports Tailwind, plugins (`@tailwindcss/forms`, `@tailwindcss/typography`, `tailwind-bootstrap-grid`), and layer files
- `generated-theme.css` — Auto-generated from theme.json (do not hand-edit)
- Dark mode uses the `.dark` class variant

### Routing

Astro file-based routing in `src/pages/`:

- `[regular].astro` — Catches pages from the `pages` collection (privacy-policy, elements)
- `blog/[single].astro` — Individual blog posts
- `blog/page/[slug].astro` — Paginated blog listing
- `categories/[category].astro`, `tags/[tag].astro` — Taxonomy pages
- `authors/[single].astro` — Author detail pages

### Utilities

`src/lib/utils/` contains text conversion (`markdownify`, `slugify`, `plainify`), date formatting, reading time calculation, similar post matching, and taxonomy filtering.
