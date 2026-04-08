# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a CollectionBuilder-CSV project showcasing example digital collection sites. CollectionBuilder is a static site generator built on Jekyll that creates digital collection websites from metadata CSV files. This specific instance hosts the CollectionBuilder Examples site at https://collectionbuilder.github.io/cb-examples/.

## Build Commands

### Development
```bash
# First time setup - install dependencies
bundle install

# Start development server
bundle exec jekyll s
```

Development server runs at `http://127.0.0.1:4000/cb-examples/`. In development mode, Jekyll skips analytics and some meta tags to speed up build time.

### Production Build
```bash
# Build for production (includes analytics and full meta tags)
JEKYLL_ENV=production bundle exec jekyll build

# Or use the rake task shortcut
rake deploy
```

Production builds output to `_site/` and take significantly longer due to additional processing.

### Rake Tasks

Available rake tasks in `rakelib/`:

- `rake deploy` - Production build with proper environment variable
- `rake generate_derivatives[thumb_size,small_size,density,missing,compress_originals]` - Generate image derivatives (requires ImageMagick and Ghostscript)
  - Defaults: thumbs=300x300, small=800x800, density=300, missing=true, compress_originals=false
  - Example: `rake generate_derivatives[100x100,300x300,70]`
- `rake download_by_csv` - Download objects based on CSV
- `rake rename_by_csv` - Rename objects based on CSV
- `rake rename_lowercase` - Rename files to lowercase
- `rake resize_images` - Resize images in objects directory

## Architecture

### Page Generation System

CollectionBuilder uses a custom Jekyll plugin (`_plugins/cb_page_gen.rb`) to automatically generate individual item pages from metadata CSV files.

**Key configuration** in `_config.yml`:
- `metadata: cb-examples` - points to `_data/cb-examples.csv` (do NOT include .csv extension)
- Optional `page_gen` array for advanced configuration

**How it works:**
1. Reads metadata from `_data/{metadata}.csv`
2. Filters rows without `objectid` or with `parentid` by default
3. Generates a page for each record at `/items/{objectid}.html`
4. Assigns layout based on `display_template` field or defaults to `item/item`
5. Adds navigation (`next_item`, `previous_item`) and index data to each page

### Template Structure

Pages consist of three parts:

1. **Stubs** in `pages/` - Markdown files with frontmatter, content, and feature includes
2. **Layouts** in `_layouts/` - HTML templates that wrap content
   - Item layouts in `_layouts/item/` correspond to `display_template` values
   - Supported display templates: `image`, `pdf`, `video`, `audio`, `panorama`, `record`, `item`, `multiple`, `compound_object`
3. **Includes** in `_includes/` - Modular HTML/Liquid components
   - `_includes/cb/` - CollectionBuilder core components
   - `_includes/feature/` - Feature components for content pages
   - `_includes/head/` - Head elements
   - JavaScript loaded via `_includes/foot.html`

### Metadata

**Required fields:**
- `objectid` - unique identifier (lowercase, no spaces, use _ or -, no /)
- `title` - object title

**Display fields:**
- `object_location` - URL or relative path to full quality object
- `image_small` - Small image representation
- `image_thumb` - Thumbnail representation
- `image_alt_text` - Alt text for images
- `format` - MIME type (used for icon fallbacks)
- `display_template` - Layout to use for item page
- `parentid` - For compound objects, matches parent's objectid

Use relative paths starting with `/` for objects in the repository (e.g., `/objects/example.jpg`). Do not include `baseurl`.

### Data Exports

Generated data files in `/assets/data/`:
- `metadata.csv/json` - Exportable metadata (fields configured in `config-metadata.csv`)
- `geodata.json` - Geographic data for mapping
- `facets.json` - Unique value counts (fields configured in theme)
- `subjects.json/csv` - Subject facets
- `locations.json/csv` - Location facets
- `timelinejs.json` - TimelineJS format
- `datapackage.json` - Frictionless Data package spec

Data exports are controlled by config CSV files in `_data/` (e.g., `config-metadata.csv`, `config-browse.csv`).

### Configuration Files

- `_config.yml` - Main Jekyll and site configuration
  - `url` and `baseurl` - Set production URLs
  - `metadata` - Name of CSV file (without .csv)
  - `title`, `tagline`, `description` - Site metadata
- `_data/config-*.csv` - Configure features (browse, metadata display, navigation, theme colors, etc.)
- `Gemfile` - Ruby dependencies

## Code Standards

### Links
- External links with `target="_blank"` must include `rel="noopener"` for security
- Links styled as buttons should NOT have `role="button"` unless they trigger interactivity (modals, etc.)
- For interactive triggers, prefer `<button>` over `<a>` elements

### File References
Use markdown link syntax for clickable references:
- Files: `[filename.ts](src/filename.ts)`
- Lines: `[filename.ts:42](src/filename.ts#L42)`
- Folders: `[src/utils/](src/utils/)`

## Testing

No automated test suite is included. Manual testing involves:
1. Building the site in development mode
2. Checking page generation output for errors/warnings
3. Verifying visualization pages (browse, map, timeline, subjects, locations)
4. Testing item page navigation and display templates
5. Production build to verify analytics and meta tag inclusion
