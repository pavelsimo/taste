---
name: taste
version: 1.1.0
description: >
  Extract engineering conventions and best practices from a corpus of repositories.
  Clones repos to a local staging area, samples key files, and synthesizes a TASTE.md
  document with patterns, style guidelines, anti-patterns, and a practical playbook.
  Pass --html to also generate a polished TASTE.html using the thariqs design system.
---

# Taste — Engineering Convention Extractor

Analyze a set of repositories and produce a `TASTE.md` document that captures the shared engineering conventions, architectural decisions, and product philosophy across the corpus.

## When to Use

- You have 2–20 repositories from a shared author, org, or domain and want to extract a transferable style guide
- You want to understand what makes a codebase family cohesive before contributing or building something similar
- You want to bootstrap a new project by inheriting conventions from repos you admire

Pass `--html` to produce both `TASTE.md` and a polished `TASTE.html`.
Pass `--slides` (requires `--html`) to also produce `TASTE-SLIDES.html` — a fullscreen slide deck summarizing the corpus.

Do not invoke for a single file, a single tiny repo, or repos with no shared context.

## Protocol

### Step 1 — Intake

Parse all arguments. Recognized flags:

- `--html` → generate `TASTE.html` in addition to `TASTE.md`
- `--slides` → also generate `TASTE-SLIDES.html` (requires `--html`)
- `--output <dir>` → write outputs to `<dir>` instead of `./`

For each non-flag argument:

- Starts with `https://` or `git@` → treat as full git URL
- Matches `owner/repo` (no slashes except the one separator) → expand to `https://github.com/owner/repo`
- Starts with `/`, `./`, `~/`, or is an existing path → treat as local path
- Otherwise → try as `https://github.com/<argument>`

Create a staging directory:

```bash
TASTE_STAGE="${HOME}/.taste/corpus/$(date +%Y%m%d-%H%M%S)"
mkdir -p "${TASTE_STAGE}"
echo "Staging in ${TASTE_STAGE}"
```

### Step 2 — Clone

For each git URL:

```bash
REPO_NAME="$(basename "${URL%.git}")"
DEST="${TASTE_STAGE}/${REPO_NAME}"

if [ -d "${DEST}/.git" ]; then
  echo "[cached] ${REPO_NAME}"
else
  echo "[clone]  ${REPO_NAME}"
  git clone --depth 1 --single-branch --quiet "${URL}" "${DEST}" \
    || echo "[unavailable] ${REPO_NAME}: clone failed"
fi
```

For local paths: use the path directly without cloning. Mark as `[local]` in the scope table.

If a clone fails: note `[unavailable]` in the scope table, skip that repo, continue.

### Step 3 — Inventory Each Repo

For each available repo, collect:

**A. Language composition** — count source files by extension, excluding generated/dependency dirs:

```bash
find "${REPO}" -type f \
  \( -name "*.go" -o -name "*.ts" -o -name "*.tsx" -o -name "*.js" \
     -o -name "*.py" -o -name "*.rs" -o -name "*.swift" -o -name "*.rb" \
     -o -name "*.java" -o -name "*.kt" -o -name "*.cs" -o -name "*.cpp" \
     -o -name "*.c" -o -name "*.zig" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" \
  -not -path "*/vendor/*" -not -path "*/dist/*" \
  -not -path "*/build/*" -not -path "*/__pycache__/*" \
  2>/dev/null | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -5
```

**B. Framework signals** — search dependency manifests:

```bash
[ -f "${REPO}/go.mod" ] && \
  grep -iE "cobra|kong|bubbletea|urfave/cli|spf13" "${REPO}/go.mod"
[ -f "${REPO}/package.json" ] && \
  grep -oE '"(commander|yargs|meow|inquirer|clack|chalk|vitest|jest|mocha)[^"]*"' "${REPO}/package.json"
for f in "${REPO}/requirements.txt" "${REPO}/pyproject.toml" "${REPO}/setup.py"; do
  [ -f "${f}" ] && grep -iE "click|typer|argparse|fire|rich|pytest|ruff|black" "${f}"
done
[ -f "${REPO}/Cargo.toml" ] && \
  grep -iE "clap|structopt|tokio|serde|anyhow|thiserror" "${REPO}/Cargo.toml"
```

Also scan for MCP (Model Context Protocol) SDK dependencies:

```bash
# Go MCP SDKs
[ -f "${REPO}/go.mod" ] && \
  grep -iE "mcp-go|modelcontextprotocol|go-sdk" "${REPO}/go.mod"
# TypeScript / Node MCP
[ -f "${REPO}/package.json" ] && \
  grep -oE '"(@modelcontextprotocol/sdk|@anthropic-ai/mcp)[^"]*"' "${REPO}/package.json"
# Python MCP
for f in "${REPO}/requirements.txt" "${REPO}/pyproject.toml"; do
  [ -f "${f}" ] && grep -iE "^mcp[>=\[]|mcp-server" "${f}"
done
# MCP server entry point files
find "${REPO}" -maxdepth 3 \
  \( -name "mcp*.go" -o -name "mcp*.ts" -o -name "server.go" -o -name "mcp_server*" \) \
  -not -path "*/.git/*" 2>/dev/null | head -5
# MCP client config examples documented in README
grep -l "mcpServers\|mcp_servers\|\"command\":" "${REPO}/README.md" 2>/dev/null
```

**C. Key top-level files present** — check for each: `README.md`, `CHANGELOG.md`, `LICENSE`, `Makefile`, `.goreleaser.yml`, `.goreleaser.yaml`, `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `tsconfig.json`, `vitest.config.ts`, `jest.config.*`, `.golangci.yml`, `.github/`, `AGENTS.md`, `CLAUDE.md`, `CODEX.md`, `docs/`, `Dockerfile`, `install.sh`, `Formula/`, `mkdocs.yml`, `docusaurus.config.js`, `docusaurus.config.ts`, `astro.config.mjs`, `_config.yml`, `hugo.toml`, `hugo.yaml`, `conf.py`, `typedoc.json`, `.readthedocs.yaml`

**D. Test file count**:

```bash
find "${REPO}" \
  \( -name "*_test.go" -o -name "*.test.ts" -o -name "*.spec.ts" \
     -o -name "*.test.js" -o -name "*.spec.js" -o -name "*_test.py" \
     -o -name "test_*.py" -o -name "*_spec.rb" \) \
  -not -path "*/node_modules/*" -not -path "*/.git/*" \
  2>/dev/null | wc -l
```

**E. Badge inventory** — extract full badge markdown from README.md to capture both
the image URL and surrounding link/style metadata:

```bash
# Linked badges: [![alt](img-url)](link-url)
grep -oE '\[!\[[^]]*\]\([^)]+\)\]\([^)]+\)' "${REPO}/README.md" 2>/dev/null | head -30

# Unlinked badge images: ![alt](img-url)
grep -oE '!\[[^]]*\]\(https://[^)]+\)' "${REPO}/README.md" 2>/dev/null \
  | grep -viE '\[!\[' | head -10

# Banner check — first 10 lines, any image (badge or not)
head -10 "${REPO}/README.md" 2>/dev/null \
  | grep -oE '!\[[^]]*\]\([^)]+\)'
```

For each badge markdown found, record:
1. The full markdown expression (preserves link target, alt text, and raw URL with all query parameters)
2. The image URL domain — classify by type:
   - **CI/Build** — `github.com/*/actions` or `*/workflows/*` badge
   - **Coverage** — `codecov.io`, `coveralls.io`
   - **Docs** — `pkg.go.dev`, `godoc.org`
   - **Quality** — `goreportcard.com`, `snyk.io`, `deps.rs`
   - **Version/Release** — `shields.io/github/v/release`, `shields.io/npm/v`
   - **License** — `shields.io/github/license`
   - **Social** — stars, forks, downloads
   - **AI/LLM Docs** — `deepwiki.com`, or shields.io badges whose link target points to `deepwiki.com`
3. From the raw query string, extract and note:
   - `style=` value if present (`flat-square`, `flat`, `for-the-badge`, or absent)
   - `color=` or `labelColor=` value if present (hex or named)
   - `logo=` value if present (the Simple Icons slug, e.g., `npm`, `swift`, `node.js`, `homebrew`, `apple`)
   - `logoColor=` value if present
4. Whether the badge is linked (wrapped in `[...](url)`) and what the link target domain is
5. Banner image: if the banner-check grep returns an image whose URL does not contain a known badge
   domain (`shields.io`, `codecov.io`, `badgen.net`, `github.com/*/workflows`, `deepwiki.com`),
   treat it as a project banner — record its relative path and alt text separately

Record badges in row order as they appear in README.md — order is itself a convention signal.

**F. README section headings** — capture the structural outline of README.md:

```bash
grep -E '^#{1,3} ' "${REPO}/README.md" 2>/dev/null | head -40
```

Record heading level, text, and order. Normalize for cross-repo comparison
(e.g., "Installation", "Install", "Getting Started" → `install` slot).

### Step 4 — Sample Files by Priority

For each repo, read files in this order. Stop after ~50 files per repo (token budget). Read up to 150 lines per implementation file; read config/CI files in full if under 200 lines.

**P1 — CLI entry points and wiring** (most signal-dense):
- `cmd/*/main.go`, `main.go`, `cmd/root.go`, `internal/cli/root.go`
- `src/index.ts`, `src/cli.ts`, `bin/*.ts`, `bin/*.js`
- `main.py`, `__main__.py`, `src/main.rs`, `Sources/*/main.swift`
- First file in `cmd/`, `commands/`, or `internal/cli/` after root
- MCP server entry points: `cmd/*/mcp*.go`, `internal/mcp/*.go`, `src/mcp*.ts`, `src/server.ts`, any file matching `*mcp*server*`

**P2 — Config and architecture**:
- `internal/config/*.go`, `src/config*.ts`, `config/*.py`
- Any file matching `*config*` or `*settings*` in `internal/` or `src/`
- Schema/migration files: `*schema*`, `*migration*`, `prisma/schema.prisma`

**P3 — Tests** (3 representative files):
- Prefer files named `root_test`, `cli_test`, `main_test`, `cmd_test`
- One unit test, one integration/CLI test, one config test if distinct

**P4 — Tooling and CI** (read in full):
- `.github/workflows/ci.yml` (or `ci.yaml`)
- `.github/workflows/release.yml` (or `release.yaml`)
- Any `.github/workflows/` file whose name contains `docs`, `pages`, `deploy`, or `preview` — these reveal documentation CI patterns
- Doc-site config files if present (detected in Step 3C): `mkdocs.yml`, `docusaurus.config.js`, `docusaurus.config.ts`, `astro.config.mjs`, `_config.yml`, `hugo.toml`, `hugo.yaml`, `conf.py`, `typedoc.json`, `.readthedocs.yaml`
- `Makefile` (first 80 lines)
- `package.json` (scripts and devDependencies sections)
- `vitest.config.ts`, `jest.config.*`, `pytest.ini`, `pyproject.toml`
- `.golangci.yml`, `.eslintrc.*`, `oxlint.json`, `biome.json`, `ruff.toml`
- `.goreleaser.yml` or `.goreleaser.yaml`

**P5 — Docs** (first 100 lines each):
- `README.md`
- `AGENTS.md`, `CLAUDE.md`, `CODEX.md` (full, they are usually short)
- List `docs/` and read 2–3 representative files

**P6 — Output helpers** (if present):
- Files in `internal/out/`, `internal/output/`, `pkg/format/`, `src/output*`, `src/format*`

**P7 — Code evidence snippets** (required, 6-12 short examples total):
- Import ordering and grouping from representative implementation files.
- Function/class docstring style from public APIs or framework extension points.
- Typing style: annotations, generics, overloads, Protocols/interfaces, typed config, or explicit untyped choices.
- Variable declaration and naming style: constants, module globals, local names, env/config names.
- Error handling style: wrapping, custom exceptions, stderr/stdout behavior, warning policy, validation failures.
- Test style: fixtures, parametrization/table tests, snapshot tests, assertion helpers, naming conventions.
- CI/linter examples: exact formatter/linter/type-check/test commands from config or workflows.

For each snippet, capture the repo, file path, and why the example is worth copying or avoiding. Keep snippets short: usually 5-20 lines, never more than 30 lines unless the surrounding context is essential.

### Step 5 — Detect Signals

For each signal family, note which repos show the pattern. Then classify:

- **Strong** (3+ repos): top-level recurring pattern, gets its own `### N.` section
- **Moderate** (2 repos): mention in Style Guidelines or Code Style section
- **Single** (1 repo): mention only in Repo-Specific Notes, unless it is exceptionally novel

**Family 1 — CLI surface**
- Global flags: `--json`, `--format`, `--dry-run`, `--read-only`, `--quiet`, `--debug`, `--verbose`
- Output format control (human-readable vs. machine-readable vs. streaming NDJSON)
- Completions support (which shells; any LLM-facing completions target)
- Diagnostic commands: `doctor`, `status`, `health`, `check`
- Version injection method (ldflags, build tags, `package.json` version field)
- Help text style (first-line sentence, examples in long description, flag capitalization)
- Error output: stderr vs stdout, lowercase convention, exit codes

**Family 2 — Architecture and data**
- Persistence: SQLite, flat files, JSONL, in-memory, hosted DB — which and why
- Schema migration strategy: `PRAGMA user_version`, Flyway, Alembic, custom
- Full-text search: FTS5, pg_trgm, Tantivy, none
- Local-first vs live-only vs hybrid
- Config resolution order: flag > env > config file > default
- Config file format: TOML, YAML, JSON/JSONC, INI
- Config path convention: XDG, `~/.config/<app>/`, `~/.<app>/`
- Secret storage: OS keyring, env var, dotfile — which default
- Multi-backend / multi-transport abstraction patterns

**Family 3 — Testing discipline**
- Test framework and version
- Coverage threshold: value + whether CI-enforced
- Test distribution: unit vs integration vs e2e
- Race condition detection (`-race` flag)
- Test naming style: behavior-oriented vs implementation-oriented
- Notable: testing the `--json` output schema specifically

**Family 4 — Tooling and shipping**
- Build tooling: Makefile, `package.json` scripts, Taskfile, just
- Linter and formatter choice
- Release automation: goreleaser, npm publish, semantic-release, GitHub Releases
- Distribution channels: Homebrew, npm, brew, go install, pre-built binaries
- **CI pipeline shape** — for each CI workflow file sampled, record:
  - Gate order: the sequence of jobs/steps (e.g., lint → test → build → security → release) — state the canonical order if 3+ repos agree
  - Multi-OS matrix: whether the workflow tests on `ubuntu-latest`, `macos-latest`, `windows-latest`, or subsets; note which OS combinations are used
  - Version matrix: Go, Node, or Python versions tested simultaneously (e.g., `matrix: go: [1.22, 1.23]`)
  - Cache usage: `actions/cache` or `actions/setup-*` built-in caching for Go modules, npm, pip, or Cargo
  - Security scanning: CodeQL (`github/codeql-action`), Trivy (`aquasecurity/trivy-action`), gosec, Semgrep, zizmor — which scanners run, on which trigger (every push, PRs only, scheduled scan)
  - Dependency automation: Dependabot (`dependabot.yml`) or Renovate (`renovate.json`) — which ecosystems are managed (npm, gomod, github-actions), automerge policy for patch/minor updates
  - Release triggers: push of a version tag `v*`, manual `workflow_dispatch`, or merge to main/master
  - Changelog/release notes generation: git-cliff, conventional-changelog, release-please, goreleaser changelog — note the required commit-message convention (Conventional Commits, etc.)
- Architecture targets: universal binary, arm64+amd64, cross-platform claims
- **Documentation CI** — detect whether any workflow deploys a documentation website:
  - **Doc-site tool**: MkDocs (`mkdocs build` / `mkdocs gh-deploy`), Docusaurus (`npm run build` + deploy step), VitePress, Astro, Jekyll (`jekyll build`), Hugo (`hugo` command), Sphinx (`sphinx-build` or `make html`), TypeDoc (`typedoc`), godoc (via `gomarkdoc` or similar static export)
  - **Deployment target**: GitHub Pages (`actions/configure-pages` + `actions/deploy-pages`, or legacy `peaceiris/actions-gh-pages`), Netlify deploy action, Vercel deploy action, ReadTheDocs webhook trigger
  - **Trigger**: push to main/master, release tag, PR preview deployment (ephemeral deploy per PR), manual `workflow_dispatch`
  - **Workflow file**: note the file name (e.g., `docs.yml`, `pages.yml`, `deploy-docs.yml`) and whether it is a standalone workflow or a job inside `release.yml`
  Classify across repos: Strong (3+), Moderate (2), Single (Repo-Specific Notes).
- **Installation mechanics** — for each distribution channel, record the exact user-facing install command and the underlying mechanism:
  - **Homebrew**: goreleaser `brews:` block → `brew install <tap>/<formula>`. Note the tap name, formula template, and any `caveats`.
  - **npm / npx**: `package.json` `bin` field + `name` → `npm install -g <pkg>` or `npx <pkg>`. Note scoped vs. unscoped and `engines.node` constraint.
  - **go install**: module path in `go.mod` → `go install <module>@latest`. Check if README documents this as the primary install path.
  - **Pre-built binaries**: goreleaser `archives:` + GitHub Release assets → direct download. Note architecture matrix and whether checksums/signatures are provided.
  - **Docker**: `Dockerfile` present → `docker pull` / `docker run`. Note multi-stage vs. scratch image.
  - **Shell one-liner**: `install.sh` or README `curl | sh` pattern.
  Classify across repos: Strong (3+), Moderate (2), Single (Repo-Specific Notes).

**Family 5 — Documentation**
- README structure: positioning → install → quickstart → command reference
- Dedicated docs site or `docs/` directory
- Example density: inline vs dedicated `examples/`
- Troubleshooting documentation
- Architecture or spec docs
- Cross-platform caveats documented explicitly
- **README section structure** — using the headings captured in Step 3F, detect:
  - Which `##`/`###` sections appear in 3+ repos (canonical), 2 repos (common), or 1 repo (unique)
  - The canonical section order and naming preferences ("Installation" vs. "Install" vs. "Getting Started"; "Usage" vs. "Quick Start")
  - Presence of "Examples" / "Demo" sections and their placement
  - Tail sections: "Contributing", "License", "Acknowledgements"
  - Depth preference: flat `##` sections vs. nested `###` sub-sections
  - Synthesize a representative README structure tree showing all canonical sections with presence counts (e.g., `[9/11 repos]`)
- **Badge patterns** — using the badge metadata captured in Step 3E, detect:
  - Which badge types appear in 3+ repos (Strong): CI, coverage, version, docs, quality, license, AI/LLM docs
  - Badge provider consistency: all shields.io? GitHub-generated CI badges? badgen.net? Mixed?
  - Whether coverage badges show a live threshold value or only pass/fail state
  - Canonical badge ordering if consistent across repos
  - **Visual style**: does the corpus use a consistent `style=` parameter — `flat-square`, `flat`, `for-the-badge`, or unset (shields.io default)? State it as a convention if 3+ repos agree
  - **Color palette**: are custom hex colors (e.g., `?color=ffd60a`) or named colors used? Is there a consistent palette across badge rows?
  - **Logo usage**: which shields.io Simple Icons logos appear (`logo=npm`, `logo=swift`, `logo=node.js`, `logo=homebrew`, `logo=apple`, etc.)? Is `logoColor=white` (or another value) applied consistently?
  - **Banner image**: is there a project banner (`![Name](assets/banner.png)` or similar) placed before the badge row? Note the relative path and naming convention if present
  - **Badge linking**: are badges wrapped in `[![badge](img-url)](link-url)` syntax (linked) or bare `![badge](img-url)` (unlinked)? Which types are consistently linked and to where?
  - **AI/LLM documentation badges**: DeepWiki badges (`deepwiki.com`) or similar AI-generated docs service badges — note their presence as a signal of AI-first documentation practice

**Family 6 — Code philosophy**
- Entry point size: thin vs fat main
- Error handling: wrap with context, discard, panic on programmer error
- Naming conventions: noun-verb commands, naming after what something IS vs DOES
- Global mutable state usage
- `init()` / module-level side effects
- Abstraction level: minimal vs aggressive
- Comment density and when comments appear

**Family 7 — Concrete code conventions**
- Import order and grouping: stdlib/third-party/local, absolute vs relative, multiline style, generated import tooling.
- Docstring style: public API examples, behavior-oriented test docstrings, parameter docs, "why" comments.
- Typing posture: strictness, annotations on public APIs, overloads/generics, runtime typing compatibility, ignored type errors.
- Variable and constant style: uppercase settings, env var names, sentinel/default objects, module-level constants.
- Error handling idioms: custom exception classes, message wording, `from None`, warning escalation, traceback controls.
- Test idioms: pytest fixtures, unittest helpers, parametrization, snapshots, warning assertions, CLI runner patterns.
- Tooling snippets: exact linter, formatter, type-check, docs, test, and publish commands.

**Family 8 — Project Structure**
- Top-level directory conventions: `cmd/`, `internal/`, `pkg/`, `src/`, `lib/`, `core/`, `app/`
- Entry point placement: where does `main()` / the CLI root live relative to package boundaries?
- Command handler organization: flat `commands/` vs. nested `cmd/<name>/` vs. co-located handler files
- Test file placement: co-located (`_test.go` beside source) vs. separate `tests/` tree
- Config and settings placement: root-level vs. `config/` vs. `internal/config/`
- Asset and resource embedding: `embed.FS`, `static/`, or external references
- Module/package boundary discipline: what goes in `internal/` (unexported) vs. top-level packages
- Shared utilities: `pkg/`, `lib/`, `internal/util/` — naming and scope conventions
- Monorepo layout: workspace configuration, per-tool directories, shared packages

**Family 9 — AI Collaboration and Self-Documentation**

For each repo containing `CLAUDE.md`, `AGENTS.md`, or `CODEX.md`, extract:
- **Presence and scope** — short project overview or long guidelines doc? AI-agent-only or also human-developer conventions?
- **Documented conventions** — coding style rules, naming conventions, or architectural decisions written down explicitly. These are the highest-confidence signals in the corpus because they are deliberate, not inferred.
- **Agent rules** — explicit do/don't instructions for AI tools: banned operations, required checks before committing, test commands to run, files to never edit.
- **Workflow guidance** — build commands, test commands, PR process, environment setup steps documented for agents or new contributors.
- **Cross-repo consistency** — are the files copy-pasted across repos (shared template), repo-specific, or absent? Consistency here signals a mature, opinionated workflow.

Classify:
- **Strong** (3+ repos share ≥ 2 documented conventions): top-level pattern in Recurring Patterns
- **Moderate** (2 repos): mention in Code Style or Style Guidelines
- **Single**: Repo-Specific Notes only, unless the content is exceptionally rich

**Family 10 — MCP Integration**

For each repo that includes an MCP SDK or MCP server implementation (detected in Step 3B), extract:
- **SDK used** — which MCP SDK and version (mark3labs/mcp-go, @modelcontextprotocol/sdk, Python mcp, etc.)
- **Transport** — stdio (most common for CLI tools), HTTP/SSE, or WebSocket
- **Tool registration pattern** — how MCP tools are defined: `mcp.NewTool(name, schema, handler)`, decorator-based, or manifest file
- **Resources and prompts** — whether the server exposes MCP resources, prompts, or only tools
- **Client configuration** — how users add the server to Claude Desktop or another MCP host (JSON snippet: `"mcpServers": { ... }`), what args/env vars are required
- **Authentication** — API key via env var, OAuth, or none
- **Dual-mode** — whether the tool runs as both a regular CLI and an MCP server (common pattern: `--mcp` flag or `mcp serve` subcommand)

Classify:
- **Strong** (3+ repos): prescriptive pattern in Recurring Patterns
- **Moderate** (2 repos): Style Guidelines / Tooling section
- **Single**: Repo-Specific Notes unless the implementation is exceptionally novel

### Step 6 — Write TASTE.md

Write `./TASTE.md` in the **current working directory** (not the staging directory). Follow the output template exactly. Be prescriptive: write "do X because Y", not "they did X". Keep repo-specific observations in the repo-specific notes section; everything else should be cross-repo synthesis.

The report must include concrete evidence. Do not only describe style; show it.
Include at least 6 short code/config snippets across at least 2 repos, and
prefer all available repos when the corpus is 3+ repos. Every snippet must have
a short caption explaining the convention it demonstrates. Good snippet topics:
imports, docstrings, typing, constants/config variables, error handling, tests,
and CI/linter commands. If a corpus has no usable snippet for a category, say
that explicitly and explain what replaced it.

When mentioning specific tools by name — linters (ruff, black, flake8, golangci-lint, oxlint, biome),
formatters (isort, gofmt, prettier), test frameworks (pytest, vitest, jest, inline-snapshot),
release tools (goreleaser, semantic-release), and CI utilities (alls-green, zizmor, act, CodSpeed) —
look up and include the canonical GitHub or documentation URL as a Markdown hyperlink on first
mention in the Tooling section. Format: `[tool-name](https://github.com/owner/repo)`.
Prefer the source repository URL over a package registry page (PyPI, npm, pkg.go.dev).
Subsequent mentions in the same section may use plain text.

### Step 7 — Generate TASTE.html (only if `--html` was passed)

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

### Step 8 — Generate TASTE-SLIDES.html (only if `--slides` was passed)

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

## Output Template

```markdown
# Engineering Taste: <corpus description — e.g., "steipete / openclaw CLI Corpus">

**Repos:** <comma-separated list>
**Analyzed:** <YYYY-MM-DD>
**Lens:** <auto-detected — e.g., "CLI tools, local-first architecture, TypeScript/Go">

---

## Scope

| Repo | Language | Framework | Domain |
|------|----------|-----------|--------|
| ...  | ...      | ...       | ...    |

<Note any unavailable repos or unusual observations here.>

---

## High-Level Read

<2–3 paragraphs. What kind of developer(s) built these? What is the dominant taste?
What is the single most important cross-cutting observation? Be specific — name the repos,
name the patterns. Do not summarize; synthesize.>

---

## Recurring Patterns

<One ### section per strong/moderate signal, ordered by prevalence and importance.
Each section: name the pattern, describe it, list which repos show it, state the implication.>

### 1. <Pattern Name>

<Description. Evidence. Implication.>

### 2. <Pattern Name>

...

---

## Style Guidelines

### Command Design
### Flag Design
### Help and Discoverability
### Defaults
### Error Handling
### Output Design
### Documentation

<Include a README structure tree showing the canonical section order across the corpus.
Use an ASCII tree with presence annotations. Example format:>

```
README.md
├── # <Title + tagline>          [N/M repos]
├── ## Badges                    [N/M repos]
├── ## Features / Overview       [N/M repos]
├── ## Installation              [N/M repos]
│   ├── ### Homebrew
│   └── ### go install
├── ## Usage / Quick Start       [N/M repos]
├── ## Commands / Reference      [N/M repos]
├── ## Configuration             [N/M repos]
├── ## Examples                  [N/M repos]
├── ## Contributing              [N/M repos]
└── ## License                   [N/M repos]
```

<Note canonical section names (e.g., "Installation" preferred over "Install"), depth
preference (flat vs. nested), and any unexpectedly absent or unique sections.>

<Badge style — document all of the following if the corpus shows consistent patterns:>

- **Badge types and order**: which types appear consistently (CI, coverage, version, docs, quality, license, AI/LLM docs) and in what canonical sequence
- **Provider**: shields.io, GitHub-generated, badgen.net, or mixed — state it as a convention if 3+ repos agree
- **Visual style**: the `style=` parameter used (e.g., `flat-square`) — if consistent, prescribe it; if mixed, note the split
- **Color palette**: any shared `color=` or `labelColor=` hex/named values; note whether the palette is consistent across the badge row
- **Logo conventions**: which `logo=` values appear and whether `logoColor=` is standardized (e.g., always `white`)
- **Linking practice**: whether badges are consistently linked to their target service (CI run, coverage report, package registry, docs site)
- **Banner image**: whether a project banner precedes the badge row — if so, note the file naming convention (e.g., `assets/banner.png`, `docs/img/banner.png`)
- **AI/LLM badges**: presence of DeepWiki or similar AI-generated documentation badges — treat as a signal of AI-first documentation practice

<If 3+ repos share the same badge row style, include a representative example using one repo's actual markdown. Show the full linked-badge syntax including query parameters, so readers can copy the pattern directly. Example format:>

```markdown
[![CI](https://img.shields.io/github/actions/workflow/status/owner/repo/ci.yml?style=flat-square&logo=github&logoColor=white)](https://github.com/owner/repo/actions)
[![npm](https://img.shields.io/npm/v/pkg?style=flat-square&logo=npm&logoColor=white)](https://npmjs.com/package/pkg)
[![License](https://img.shields.io/github/license/owner/repo?style=flat-square)](LICENSE)
```

---

## Project Structure

<One paragraph synthesizing the canonical directory layout observed across the corpus.
Then: a directory-purpose table OR an ASCII tree showing the layout a new project
in this style should follow. Be prescriptive — describe what belongs where and why,
not just what was observed. Cover: entry point, command handlers, config, tests,
output helpers, assets, and internal vs. exported packages.>

---

## Code Style

### <Primary Language>
### <Secondary Language, if applicable>
### General Principles

---

## Code Evidence

<6-12 short snippets from the sampled repos. Each item should include a repo/file label,
a fenced code block, and 1-3 sentences explaining the convention. Cover import order,
docstrings, typing, variable/constants/config style, error handling, tests, and CI/linting
where the corpus provides evidence. Keep excerpts short and relevant.>

---

## Tooling and Shipping Practices

### Build
### Testing
### Release
### CI

<Describe the CI pipeline shape across the corpus. Cover each of the following if the corpus provides evidence:>

- **Gate order**: the canonical job/step sequence (e.g., lint → test → security → build → release) — state it prescriptively if 3+ repos agree
- **OS and version matrix**: which OS combinations are tested (`ubuntu-latest`, `macos-latest`, `windows-latest`) and which language version matrices are used
- **Caching**: whether `actions/cache` or setup-action built-in caching is used; which ecosystems (Go modules, npm, pip, Cargo) and the cache key strategy
- **Security scanning**: which scanners run (CodeQL, Trivy, gosec, Semgrep, zizmor) and at which trigger (every push, PRs only, scheduled scan)
- **Dependency automation**: Dependabot or Renovate — which ecosystems are managed and whether automerge is enabled for patch/minor updates
- **Release triggers**: what event fires the release job — version tag `v*`, manual dispatch, or merge to main
- **Changelog generation**: which tool produces release notes (git-cliff, release-please, conventional-changelog, goreleaser) and the required commit-message convention it enforces
- **Documentation deployment**: if any workflow deploys a docs site — the tool used (MkDocs, Docusaurus, VitePress, Hugo, Sphinx, TypeDoc), the deployment target (GitHub Pages, Netlify, Vercel, ReadTheDocs), and the trigger (push to main, release tag, PR preview)

<If the corpus shows a canonical CI structure, state it prescriptively: "All repos run lint and test on every push, security scanning on PRs only, and release on v* tags." Concrete is more useful than hedged.>

### Distribution

### Installation

<For each channel used by 2+ repos: state the exact user-facing install command and the
mechanism behind it. Example: "goreleaser publishes a Homebrew tap formula automatically
on every v* tag — users run `brew install <tap>/<tool>`." For pre-built binaries, note the
architecture matrix and whether checksums or signatures are provided.>

---

## MCP Integration

<Only include this section if 1+ repos implement an MCP server.>

<Describe which repos expose MCP servers, the SDK and transport used (stdio, HTTP/SSE),
and how tools/resources are registered. Include the client configuration pattern if
consistent across repos.>

**Tool registration pattern:**
<Code snippet or description of how the author registers MCP tools.>

**Client configuration:**
<The JSON snippet users add to Claude Desktop or other MCP hosts, with env var and arg
conventions.>

**Dual-mode pattern (if applicable):**
<How the tool switches between CLI mode and MCP server mode — flag, subcommand, or env var.>

---

## Product Taste

<2–4 paragraphs on philosophy — what does this developer optimize for, what do they deprioritize,
what values show up repeatedly even when they are not the easiest path.>

---

## Best Practices Worth Copying

<Numbered list, 8–15 items. Each is an actionable prescription: "Do X because Y."
Lead with the thing to do, not with the observation.>

1. **<Imperative instruction.>** <One sentence of justification.>
2. ...

---

## Anti-Patterns This Style Avoids

<Bulleted list. Each describes what the corpus consistently does NOT do and why it matters.
Use negative framing: "No X." or "Never Y.">

---

## Repo-Specific Notes

<One paragraph per repo. Only what is worth studying in that repo specifically — not a summary.
What makes it the best or most distinctive example of a particular pattern?>

**<repo-name>:** ...

---

## Self-Documented Guidelines

<Only include this section if 2+ repos have CLAUDE.md, AGENTS.md, or CODEX.md.>

<Note which repos have these files, whether they share a common structure or template,
and which conventions are explicitly written down across multiple repos.>

**Shared agent rules:**
<Bulleted list of instructions that appear in 2+ files — e.g., "always run tests before
committing", "never edit vendored files", "use gofumpt before pushing".>

**Documented conventions:**
<Coding style, naming, or architectural decisions explicitly stated in these files.>

**Workflow guidance:**
<Build, test, or release steps the author explicitly documented for agents or new contributors.>

*Note: conventions here are direct quotes or close paraphrases from the author's own
documentation — treat them as the highest-confidence signals in this corpus.*

---

## Practical Playbook

### Starting a New <Project Type>

<Numbered checklist derived from corpus patterns.>

### <Domain-Specific Checklist — e.g., Output Design, Local Persistence, Safety, Release>

<Checkbox list: - [ ] item>

---

*Analysis synthesizes recurring patterns across <N> repositories.
Generated by the [taste skill](https://github.com/pavelsimo/taste).*
```

## Edge Cases

| Situation | Action |
|-----------|--------|
| Clone fails (auth, 404, timeout) | Mark `[unavailable — reason]` in scope table; continue with remaining repos |
| Local path given | Use directly; mark `[local]` in scope table; run same inventory and sampling steps |
| Repo already cloned in staging | Use existing checkout; do not `git pull` (determinism) |
| Empty or archived repo (<3 source files) | Mark `[empty or archived]` in scope table; one-line note in repo-specific notes; skip analysis |
| Single repo | Run full protocol; prefix every recommendation with a single-repo caveat |
| Mono-repo with multiple tools | Count as one scope entry; sample each tool's entry point separately within the 50-file budget |
| Non-English README | Proceed; use code as primary signal source; note language in scope table |
| Very large repo (>5000 source files) | Stick to the P1–P6 priority list strictly; do not expand sampling |
