# HTML and Slides Generation Reference

## Step 7 — Generate TASTE.html (only if `--html` was passed)

Use the thariqs HTML effectiveness corpus as the design reference:

- Index: `https://thariqs.github.io/html-effectiveness/`
- Source repo: `https://github.com/thariqs/html-effectiveness`

The index contains 20 self-contained `.html` examples. Use this catalog to choose
the right visual and interaction patterns before generating HTML:

| File | Pattern | Use it for |
|------|---------|------------|
| `01-exploration-code-approaches.html` | Side-by-side code approaches | Comparing multiple implementation options with trade-offs visible inline |
| `02-exploration-visual-designs.html` | Visual design directions | Showing layout, palette, or visual alternatives as rendered options |
| `03-code-review-pr.html` | Annotated pull request | Presenting diffs with severity tags, jump links, and reviewer notes |
| `04-code-understanding.html` | Module map | Explaining unfamiliar code with boxes, arrows, hot paths, and entry points |
| `05-design-system.html` | Living design system | Rendering tokens, type scale, spacing, and component primitives as a browsable sheet |
| `06-component-variants.html` | Component variants | Reviewing every size, state, and intent of a UI component in one place |
| `07-prototype-animation.html` | Animation sandbox | Tuning motion, duration, easing, and interaction feel before implementation |
| `08-prototype-interaction.html` | Clickable flow | Testing a small multi-screen interaction with enough fidelity to feel the workflow |
| `09-slide-deck.html` | Arrow-key slide deck | Turning a memo or thread into a lightweight browser-native presentation |
| `10-svg-illustrations.html` | SVG figure sheet | Creating editable inline diagrams or illustrations for posts and docs |
| `11-status-report.html` | Weekly status report | Formatting shipped/slipped work, timelines, and small charts for fast scanning |
| `12-incident-report.html` | Incident timeline | Presenting post-mortems with chronological events, logs, and follow-up checklists |
| `13-flowchart-diagram.html` | Annotated flowchart | Showing a pipeline or process with clickable steps, timings, and failure paths |
| `14-research-feature-explainer.html` | Feature explainer | Explaining a repo feature with TL;DR, collapsible paths, tabs, callouts, and FAQ |
| `15-research-concept-explainer.html` | Concept explainer | Teaching an abstract concept with interactive visuals, comparison tables, and glossary |
| `16-implementation-plan.html` | Implementation plan | Laying out milestones, data flow, inline mockups, risky code, and risk tables |
| `17-pr-writeup.html` | PR writeup | Giving reviewers motivation, before/after context, file tour, and focus areas |
| `18-editor-triage-board.html` | Ticket triage board | Building a temporary drag-and-drop board that exports the resulting order |
| `19-editor-feature-flags.html` | Feature flag editor | Editing grouped toggles with dependency warnings and copyable config diff |
| `20-editor-prompt-tuner.html` | Prompt tuner | Editing prompt templates while sample inputs re-render live beside them |

For `TASTE.html`, use `TASTE.md` as the source of truth, but do **not** make
the HTML a bland Markdown-to-HTML report. It should feel like an editorial
analysis artifact: scannable, visual, and opinionated, with components chosen
to explain the corpus rather than merely restating sections.

Start with these two references for baseline typography, layout, tables,
collapsible sections, tabs, callouts, and report structure:

```bash
curl -fsSL "https://raw.githubusercontent.com/thariqs/html-effectiveness/main/11-status-report.html" \
  -o /tmp/taste-ref-report.html
curl -fsSL "https://raw.githubusercontent.com/thariqs/html-effectiveness/main/14-research-feature-explainer.html" \
  -o /tmp/taste-ref-explainer.html
curl -fsSL "https://raw.githubusercontent.com/thariqs/html-effectiveness/main/13-flowchart-diagram.html" \
  -o /tmp/taste-ref-flowchart.html
curl -fsSL "https://raw.githubusercontent.com/thariqs/html-effectiveness/main/04-code-understanding.html" \
  -o /tmp/taste-ref-module-map.html
```

Read both files. Extract the `<style>` block from each. Merge and adapt them
instead of copying mechanically. If the corpus would benefit from a more
specific visual treatment, fetch 1-3 additional catalog examples and adapt
their components:

- Use `04-code-understanding.html` for module maps, architecture boxes, arrows,
  entry points, and hot paths.
- Use `13-flowchart-diagram.html` for pipeline, lifecycle, request flow,
  release flow, or data-sync diagrams.
- Use `15-research-concept-explainer.html` for teaching an abstract product or
  engineering philosophy with glossary, comparisons, or interactive visuals.
- Use `16-implementation-plan.html` for milestone layouts, risk tables, and
  data-flow diagrams.
- Use `17-pr-writeup.html` for file tours, before/after framing, and reviewer
  focus areas.
- Use `18-editor-triage-board.html` or `19-editor-feature-flags.html` only
  when an interactive board/editor genuinely explains the corpus.

Then write `./TASTE.html` as a single self-contained file.

**Design standard:**

- The page must have a distinct point of view and visual hierarchy. A reader
  should understand the corpus taste in 30 seconds from the header, hero/lead,
  top metrics, and first visual component.
- Use the report structure to organize content, but vary presentation by
  section. Avoid repeating the same card/list/detail pattern for every section.
- Include at least 5 distinct component types, and at least 3 must be more
  expressive than plain prose, a table, or bullet cards.
- Prefer visual explanation over decorative layout. Every component must teach
  something about the corpus: prevalence, tradeoff, workflow, architecture,
  maturity, risk, or transferable practice.
- It is acceptable to include a tasteful theme toggle, sticky horizontal nav,
  progress/step components, pattern grids, timeline strips, comparison panels,
  repo cards, flow diagrams, module maps, tabs, accordions, callouts, and
  checklist panels.
- Do not add animation-heavy gimmicks, fake dashboards, or decorative charts
  that are not backed by observed repo data.

**HTML structure:**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Engineering Taste: <corpus name></title>
  <style>
    /* adapted CSS from selected reference files */
  </style>
</head>
<body>
  <header> <!-- editorial title, lead, metadata, top insight --> </header>
  <nav> <!-- sticky nav generated from major sections --> </nav>
  <main>
    <!-- sections with varied components -->
  </main>
  <script>/* tab-switching JS from reference explainer */</script>
</body>
</html>
```

**Component selection guide** — choose components based on the corpus, not by
rigid section mapping:

| Need in the analysis | Prefer these HTML components |
|----------------------|------------------------------|
| Corpus scope and coverage | compact table, repo cards, language badges, small metric strip |
| High-level read | strong hero/lead, TL;DR callout, "core thesis" panel |
| Recurring patterns | pattern grid, ranked cards, accordions for detail, evidence chips, flow diagrams for process patterns |
| Architecture or data flow | module map, flowchart, pipeline diagram, swimlane |
| CLI/API surface | command matrix, tabs for human vs JSON output, before/after examples |
| Tradeoffs | side-by-side comparison cards, decision table, pros/cons columns |
| Testing/release discipline | timeline, checklist grid, maturity ladder, CI pipeline strip |
| Concrete code evidence | code tabs, annotated snippets, file-tour panels, before/after examples |
| Product taste | manifesto panel, principle cards, quote-style insight blocks |
| Best practices | numbered steps, checklist grid, "copy this" cards |
| Anti-patterns | danger callout list, red-accent cards, "avoid" matrix |
| Repo-specific notes | repo note cards, file-tour layout, grouped carryover panel |
| Practical playbook | stepper, read-only checkbox panels, collapsible implementation phases |
| Project structure / file organization | module map (04-code-understanding.html), labeled directory tree in `<pre><code>`, directory-purpose table |
| README structure | `<pre class="dir-tree">` section-tree with presence counts as `.comment` annotations (same span classes as project structure trees) |
| Self-documented guidelines | callout panel with clay left border, author-voice pull quote, two-column layout (agent rules left, documented conventions right), repo-presence indicators |
| MCP integration | tabbed panel (CLI mode tab + MCP server tab), client config JSON code block, tool registration snippet |

Always include these baseline mappings:

- Document title: `<h1>` with an "Engineering Taste" eyebrow.
- Metadata: compact pills for repos, date, lens, languages, and notable counts.
- Scope: a real table or repo-card grid with unavailable repos clearly marked.
- Inline `code`: `<code>` with mono font.
- Fenced code blocks: `<pre><code>` or a single-tab `.tabs` panel.
- `**bold**`: `<strong>`.
- Tool hyperlinks: render Markdown tool links as `<a href="..." target="_blank" rel="noopener noreferrer">tool-name</a>`. In CI pipeline strips, wrap each stage label in an `<a>` if a tool URL is known. In comparison tables, linkify tool names on first occurrence per section.
- Repo note names: in `.repo-note-name` (or equivalent heading element for repo-specific notes), wrap the repo name in `<a href="<repo-url>" target="_blank" rel="noopener">name</a>`. Use the same URL as the scope table or repo cards.

**CI pipeline strip rendering rules:**
- Each `.pipeline-stage` must have `min-width: 120px` and `flex: 1 1 120px`.
- Stage detail text (sub-label, version, steps) must have `word-break: break-word; overflow-wrap: anywhere; font-size: 0.7rem`.
- Shorten stage labels to ≤15 characters (e.g., "actions/checkout@v4" → "checkout@v4").
- If a pipeline has more than 8 stages, split into two rows using a `flex-wrap: wrap` container.

**Anti-pattern list rendering rules:**
- Render anti-patterns as a plain `<ul>` with `list-style: none; padding: 0`.
- Each `<li>` must be `display: block` (never flex or grid). Structure: one `<strong>` lead phrase followed by flowing prose text. Inline `<code>` elements must remain inline — do not wrap them in block-level elements.
- The `<ul>` itself and each `<li>` must NOT have `display: flex`, `display: grid`, or `float`.
- Add a left border accent: `border-left: 3px solid var(--rust); padding-left: 1rem; margin-bottom: 1rem`.

For Strong-signal patterns (3+ repos) that describe a process or decision chain, generate an SVG
or CSS+HTML flow diagram modeled on `13-flowchart-diagram.html`. Priority targets — generate a
diagram for any of these that appear in the corpus:

- **CLI command dispatch / main loop**: argument parsing → subcommand routing → handler execution → exit code
- **Error handling chain**: error created → wrapped with context → written to stderr → exit code set
- **Config resolution order**: CLI flag → env var → config file → compiled default
- **Release / CI pipeline**: branch → lint → test → build → sign → publish (more detailed than the pipeline strip)

Each diagram must reflect what the corpus actually does, not a generic template. Annotate nodes
with representative file paths or function names from the sampled repos. Use the same color
palette as the rest of the page.

**Practical Playbook rendering rules:**
- Render each checklist group as a milestone entry using a vertical timeline, NOT a 2-column grid. Three-column layout: `160px label | 28px dot+line | 1fr content`.
- CSS: `.playbook-milestones { display: flex; flex-direction: column; gap: 0; }` · `.playbook-milestone { display: grid; grid-template-columns: 160px 28px 1fr; gap: 0 18px; position: relative; }` · `.milestone-label { text-align: right; font-family: var(--mono); font-size: 0.72rem; font-weight: 700; text-transform: uppercase; letter-spacing: .05em; color: var(--gray-500); padding-top: 6px; }` · `.milestone-dot-col { display: flex; flex-direction: column; align-items: center; }` · `.milestone-dot { width: 14px; height: 14px; border-radius: 50%; background: var(--bg); border: 3px solid var(--clay); margin-top: 4px; flex-shrink: 0; }` · `.milestone-line { width: 2px; flex: 1; background: var(--gray-300); margin: 4px 0; min-height: 24px; }` · `.playbook-milestone:last-child .milestone-line { display: none; }` · `.milestone-body { padding-bottom: 2rem; font-size: 0.83rem; }`.
- Each checklist item: `<div class="checklist-item">` with `<input type="checkbox" disabled>` (read-only visual state).
- Do NOT use `grid-template-columns: 1fr 1fr` for the playbook container — the side-by-side layout makes groups hard to scan.

**Project structure tree rendering rules:**
- Wrap the directory tree in a `<pre class="dir-tree">` element (not a plain `<div>`) so `white-space: pre` applies automatically and literal indentation is preserved.
- Use ASCII tree connector characters: `├──` for non-last siblings, `└──` for the last sibling at a level, `│   ` (pipe + 3 spaces) as a continuation indent for items whose parent has more siblings below.
- Color spans: `.dir` (clay/accent, bold) for directory names, `.file` (foreground) for file names, `.comment` (gray italic) for `# ...` annotations. Tree connector characters are plain text with no extra span.
- Ensure the `.dir-tree` CSS rule includes `white-space: pre; overflow-x: auto;` — without `white-space: pre`, all indentation collapses and the tree becomes an unreadable inline flow.
- Keep each comment annotation to ≤ 35 characters so lines do not trigger horizontal scroll at typical column widths (~480px per panel).

**SVG flow diagram rendering rules (must follow exactly):**
- Minimum node dimensions: `width="240" height="52"` for single-line labels; `width="240" height="68"` for two-line labels.
- ViewBox minimum width: 340px for a single-column diagram; 620px for a side-by-side or branching layout.
- Font sizes: primary label 12px, secondary sub-label 10px. Never use font sizes smaller than 10px.
- Text layout: use two separate `<text>` elements (primary + sub), NOT `<tspan>`. Position primary at `y = node_top + 22`; secondary at `y = node_top + 38`.
- Text anchoring: `text-anchor="middle"` and `x = node_center_x` for both text elements.
- Label truncation: if a label would exceed 28 characters, shorten it (e.g., "subcommand.RunE(cmd, args)" → "subcmd.RunE(cmd, args)").
- Arrow clearance: leave at least 24px vertical gap between the bottom of one node and the top of the next; arrow labels must not overlap node boundaries.
- Always define an SVG `<defs>` arrowhead marker and reference it via `marker-end`.

**README structure tree rendering rules:**
- Render as a `<pre class="dir-tree">` block using the same ASCII connector characters and span classes as project structure trees (`.dir`, `.file`, `.comment`).
- Section headings (e.g., "## Installation") use `.dir` spans (bold, warm tan). `###` sub-sections use `.file` spans (normal weight). Presence counts (e.g., `[9/11 repos]`) use `.comment` spans (gray italic).
- The tree shows the canonical cross-repo section order, not any single repo's README. Omit sections that appear in fewer than 2 repos.

**Self-Documented Guidelines rendering rules:**
- Render as a callout panel with a clay (`var(--clay)`) left border and a light cream or slightly-tinted background. Use a distinct `ℹ` or author-quote icon to signal "these are the author's own words."
- Use a two-column layout inside the panel: left column for agent rules, right column for documented conventions. On narrow screens, stack vertically.
- Above the columns, add a compact row of repo-presence indicators (checkmark badges or pill tags) showing which repos contain these files.
- Any direct quotes from CLAUDE.md / AGENTS.md should be rendered in italic inside a `<blockquote>` or quote card.

**Navigation** - generate anchors from major sections. A horizontal sticky nav
usually works better for editorial reports; a sidebar is acceptable for dense
research explainers. Slugify heading text for anchor IDs (lowercase, spaces to
hyphens, strip special chars).

**Self-contained requirements:**
- All CSS in one `<style>` block — no external stylesheets
- System fonts only (`ui-serif Georgia`, `system-ui`, `ui-monospace`) — no Google Fonts
- Include only small inline JavaScript needed by chosen components, such as the
  tab-switching script from the explainer or a theme toggle. No external JS.
- Checklist `<input>` elements are `disabled` (read-only visual state, not interactive)
- File must open correctly from disk (`file://` URLs) — no server required
- Add an inline data-URL favicon so local browser previews do not request
  `favicon.ico`.
- Validate with a browser when practical. If using Playwright, a temporary
  local server is acceptable when `file://` is blocked; stop the server after
  verification.
- **Code block style and syntax highlighting:** Code blocks must always use a dark slate background regardless of page theme. Set `pre { background: #141413; border: none; border-radius: 10px; padding: 16px 20px; overflow-x: auto; }` and `pre code { color: #E8E6DE; font-size: 12.5px; background: none; padding: 0; }`. Include a self-contained inline `<script>` highlighter targeting all `<pre><code>` blocks. Token classes and colors (earthy warm palette from 01-exploration-code-approaches.html): `.kw { color: #D97757; font-weight:600 }` (keywords — clay) · `.str { color: #788C5D }` (strings — olive) · `.cm { color: #87867F; font-style:italic }` (comments) · `.num { color: #2aacb8 }` (numbers — teal) · `.fn { color: #C9B98A }` (identifiers — warm tan). The highlighter must escape HTML before applying spans, auto-detect language from `class="language-X"` or content heuristics, and mark processed blocks with `data-highlighted="true"`. These colors apply in both light and dark page themes — code blocks are always dark.

Write the result to `./TASTE.html`.

---

## Step 8 — Generate TASTE-SLIDES.html (only if `--slides` was passed)

Fetch the slide deck reference:

```bash
curl -fsSL "https://raw.githubusercontent.com/thariqs/html-effectiveness/main/09-slide-deck.html" \
  -o /tmp/taste-ref-slides.html
```

Read it. Extract its `<style>` block (slide sizing, scroll-snap, counter, title slide, invert slide) and JavaScript (keyboard nav + IntersectionObserver counter). Adapt both for the corpus content.

**Slide structure (8–12 slides):**

1. **Title** — corpus name, date, repo count + languages, one-line thesis from High-Level Read.
2. **Scope** — compact repo grid with language badges; unavailable repos shown dimmed.
3. **Top Patterns** — ranked list of all Strong patterns with evidence counts.
4. **Pattern deep-dive** — one slide per notable Strong pattern (code snippet + implication callout); typically 2–3 slides.
5. **Project Structure** — the canonical dir-tree (`<pre class="dir-tree">` with ASCII connectors) for the primary language. **Copy the tree verbatim from TASTE.md's "Project Structure" section.** Never compress multi-level paths onto a single line (e.g., `cmd/<name>/main.go` on one line is wrong — each directory must be its own indented level: `cmd/` → `<name>/` → `main.go`). Never flatten a nested directory by inlining its parent (e.g., `.github/workflows/` is wrong — render `.github/` with `workflows/` indented beneath it).
6. **Tooling & CI** — CI pipeline stages (text list or mini strip) + formatter/linter highlights.
7. **Best Practices** — top 6–7 actionable items as short bullets.
8. **Anti-patterns** — top 4–6 items with red accent.
9. **Playbook Quickstart** — "Starting a New \<language\> CLI" condensed to 6–8 items.
10. **Close** — "Patterns synthesized across N repos." with corpus repo links.

**Rendering rules:**
- Each `<section class="slide" id="sN">` fills `100vw × 100vh`. `body { scroll-snap-type: y mandatory; overflow-x: hidden; }`.
- Fixed `#counter` at bottom-right: `"{current} / {total}"` in monospace 12px.
- Keyboard: ArrowRight/Down/Space → next; ArrowLeft/Up → previous. Smooth `scrollIntoView`.
- Code on slides: dark slate background (`#141413`), same token colors as TASTE.html.
- Dir-trees on slides: use a `<div class="dir-tree">` or `<pre class="dir-tree">` element. **The `.dir-tree` CSS rule MUST include `white-space: pre; overflow-x: auto;`** — without `white-space: pre`, all newlines and indentation collapse into a single unreadable inline flow. Use `.hl` / `.dim` spans for highlighting.
- Content must fit one viewport — no internal scroll. Split long content across two slides if needed.
- Use same CSS variable palette (`--clay`, `--olive`, `--rust`, `--gray-*`) as TASTE.html.
- Write the result to `./TASTE-SLIDES.html`.
