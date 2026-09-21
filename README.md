Document loading & source
- Fetches a default Markdown file (your_md_file.md) on load; falls back to a manual file picker if fetch fails (e.g., local file:// browsing)
- "Open a Markdown file" collapsible picker with drag-free file input, filename display, and a loading spinner while rendering
- Displays the current source filename in the header

Markdown rendering
- Renders Markdown via marked.js with GFM support
- YAML-like front matter (--- block) parsed into a dedicated key/value table at the top of the document
- Custom renderer intercepts fenced ```mermaid code blocks and renders them as diagrams instead of code

Table of contents / navigation
- Auto-generated sidebar TOC from h2/h3/h4 headings, with unique slugified anchor IDs
- Active-section highlighting synced to scroll position
- Reading-progress bar tied to scroll depth
- Sticky sidebar; collapsible mobile drawer with a "Contents" toggle button

Search
- Global document search box that hides non-matching top-level content blocks and TOC links
- "No matching sections" empty-state message

Section filter
- "Filter" modal with a hierarchical checkbox tree (h2 → h3 → h4)
- Cascading selection: checking a parent selects children; che ancestors
- "Select all" checkbox with indeterminate state
- Save/Cancel; closes on overlay click, close button, or Escape
- Hides filtered-out sections from both content and TOC

Tables
- Wraps tables in a horizontally scrollable container
- Auto-pagination for tables with more than 10 rows: per-table search box, rows-per-page selector (10/25/50/100), Prev/Next pager with row-range info
- Zebra striping that respects pagination/filtering

Mermaid diagrams
- Renders diagrams with a dark "screen" theme (swapped to a ling)
- Pan (drag) and zoom (scroll wheel, +/− buttons) per diagram, with a reset-view button
- Scrollable, bordered diagram frame with floating control overlay

Printing
- Print button (desktop + mobile) that opens all collapsed <details> elements, resets diagram zoom, swaps to print-friendly diagram theme, then triggers
  window.print()
- Dedicated print stylesheet: hides chrome (topbar, sidebar, filters, draft note), forces page breaks at h2, unhides all paginated table rows, adjusts colors/contrast for print
- Restores UI state (afterprint event) — recollapses details,

Responsive / accessibility
- Mobile breakpoint (≤840px): collapsible sidebar drawer, condensed toolbar, hidden filter/print buttons replaced by sidebar action buttons
- Reduced-motion media query disables animations/transitions
- Visible focus outlines, screen-reader-only labels for search inputs

Misc UI
- Summary card area (currently commented out, unused placehold
- "Draft without direction" badge in the header denoting reader-view status
