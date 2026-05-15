# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development

No build system. Open `index.html` directly in a browser (double-click in Finder, or `open index.html`). All code lives in a single file.

For local development with Claude-in-Chrome, use `serve.py` instead of the plain Python server — it disables HTTP caching so Chrome always fetches the latest files:
```bash
python3 serve.py
```

**Whenever you replace a media file** (same filename, new content), append or bump a version query string in the `projects` array. Without this, Chrome will serve the old cached file even across page reloads:
```js
media: ["media/foo.png?v=20260512"]
```
This applies to all media types including SVG. The `?v=` string makes Chrome treat it as a new URL it has never seen, bypassing both disk cache and in-memory cache. SVGs are especially prone to this because they are loaded lazily (only when a modal opens) and Chrome's in-memory cache for them survives page reloads within the same browser session.

## Architecture

Everything is in `index.html`: CSS (top `<style>`), HTML structure, and a `<script>` block at the bottom. No external JS dependencies.

### Page sections (in order)
- **Nav** — sticky top bar
- **Hero** — full-screen landing with scroll hint
- **Stage** (`#showcase`) — sticky scroll-jacked animation cycling through 3 demo videos; controlled by the `stageCopy` array and `updateStage()` scroll handler
- **Experience** (`#experience`) — static HTML cards with glow overlay effect
- **Projects** (`#work`) — dynamically rendered from the `projects` JS array via `renderProjects()`
- **About** (`#about`) — headshot + bio

### Adding / editing projects

Edit the `projects` array in the `<script>` block. Each object supports:

| Field | Notes |
|---|---|
| `title`, `label`, `desc`, `stack`, `year`, `role`, `concepts`, `body` | Text content |
| `feat: true` | Makes the card span full width (auto-applied to first card when total count is odd) |
| `repo` | GitHub path (`"user/repo"`); omit or `""` to hide the link |
| `media` | Array of file paths. Supports `.mp4`, `.mov`, `.webm`, `.png`, `.jpg`, `.svg` |
| `layout` | Custom gallery layout class. Currently only `"layout-top-bar"` is defined |

If `media` is absent, the modal opens with no gallery (just text). If `video` is used instead of `media` (legacy), it still works.

### Modal gallery layouts (CSS classes on `.m-gallery`)

- **`single`** — one item, full width. The tile auto-sizes to the media's natural aspect ratio (`height: auto; object-fit: initial; padding: 0`). Do not add `min-height` or fixed `aspect-ratio` here — that would force a tile shape that doesn't match the content, producing black bars or cropping. Each video/SVG defines its own tile height.
- **`collage count-N`** — 2-column grid, first item spans all rows; `object-fit: cover` (photos fill tiles edge-to-edge)
- **`count-2`** — two equal columns, 300px row height; `object-fit: cover`
- **`layout-top-bar`** — custom 4-tile layout: wide top bar + left tall + 2 right stacked. Row heights live in `grid-template-rows` on `.m-gallery.layout-top-bar`. Uses `object-fit: cover` throughout; adjust `object-position` per child via nth-child selectors to control crop anchor.

### Theming

CSS custom properties on `:root` (light) and `[data-theme="dark"]`. Theme is persisted to `localStorage` and defaults to dark. The global spotlight effect and hero spotlight are dark-mode only.

### Media files

All media lives in `media/`. When adding new project videos or images, drop the file there and reference it in the `projects` array. `.MOV` files need two `<source>` tags (quicktime + fallback) when used in the stage section, but the modal renderer handles this automatically based on file extension.
