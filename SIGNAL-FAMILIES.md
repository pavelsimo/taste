# Signal Families — Detection Reference

For each family, note which repos show the pattern. Classify:

- **Strong** (3+ repos): top-level recurring pattern, gets its own `### N.` section
- **Moderate** (2 repos): mention in Style Guidelines or Code Style section
- **Single** (1 repo): mention only in Repo-Specific Notes, unless it is exceptionally novel

## Family 1 — CLI surface

- Global flags: `--json`, `--format`, `--dry-run`, `--read-only`, `--quiet`, `--debug`, `--verbose`
- Output format control (human-readable vs. machine-readable vs. streaming NDJSON)
- Completions support (which shells; any LLM-facing completions target)
- Diagnostic commands: `doctor`, `status`, `health`, `check`
- Version injection method (ldflags, build tags, `package.json` version field)
- Help text style (first-line sentence, examples in long description, flag capitalization)
- Error output: stderr vs stdout, lowercase convention, exit codes

## Family 2 — Architecture and data

- Persistence: SQLite, flat files, JSONL, in-memory, hosted DB — which and why
- Schema migration strategy: `PRAGMA user_version`, Flyway, Alembic, custom
- Full-text search: FTS5, pg_trgm, Tantivy, none
- Local-first vs live-only vs hybrid
- Config resolution order: flag > env > config file > default
- Config file format: TOML, YAML, JSON/JSONC, INI
- Config path convention: XDG, `~/.config/<app>/`, `~/.<app>/`
- Secret storage: OS keyring, env var, dotfile — which default
- Multi-backend / multi-transport abstraction patterns

## Family 3 — Testing discipline

- Test framework and version
- Coverage threshold: value + whether CI-enforced
- Test distribution: unit vs integration vs e2e
- Race condition detection (`-race` flag)
- Test naming style: behavior-oriented vs implementation-oriented
- Notable: testing the `--json` output schema specifically

## Family 4 — Tooling and shipping

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

## Family 5 — Documentation

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

## Family 6 — Code philosophy

- Entry point size: thin vs fat main
- Error handling: wrap with context, discard, panic on programmer error
- Naming conventions: noun-verb commands, naming after what something IS vs DOES
- Global mutable state usage
- `init()` / module-level side effects
- Abstraction level: minimal vs aggressive
- Comment density and when comments appear

## Family 7 — Concrete code conventions

- Import order and grouping: stdlib/third-party/local, absolute vs relative, multiline style, generated import tooling.
- Docstring style: public API examples, behavior-oriented test docstrings, parameter docs, "why" comments.
- Typing posture: strictness, annotations on public APIs, overloads/generics, runtime typing compatibility, ignored type errors.
- Variable and constant style: uppercase settings, env var names, sentinel/default objects, module-level constants.
- Error handling idioms: custom exception classes, message wording, `from None`, warning escalation, traceback controls.
- Test idioms: pytest fixtures, unittest helpers, parametrization, snapshots, warning assertions, CLI runner patterns.
- Tooling snippets: exact linter, formatter, type-check, docs, test, and publish commands.

## Family 8 — Project Structure

- Top-level directory conventions: `cmd/`, `internal/`, `pkg/`, `src/`, `lib/`, `core/`, `app/`
- Entry point placement: where does `main()` / the CLI root live relative to package boundaries?
- Command handler organization: flat `commands/` vs. nested `cmd/<name>/` vs. co-located handler files
- Test file placement: co-located (`_test.go` beside source) vs. separate `tests/` tree
- Config and settings placement: root-level vs. `config/` vs. `internal/config/`
- Asset and resource embedding: `embed.FS`, `static/`, or external references
- Module/package boundary discipline: what goes in `internal/` (unexported) vs. top-level packages
- Shared utilities: `pkg/`, `lib/`, `internal/util/` — naming and scope conventions
- Monorepo layout: workspace configuration, per-tool directories, shared packages

## Family 9 — AI Collaboration and Self-Documentation

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

## Family 10 — MCP Integration

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
