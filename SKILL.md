---
name: taste
version: 1.1.0
description: Analyzes 2-20 repositories and extracts engineering conventions into a TASTE.md document with patterns, style guidelines, anti-patterns, and a practical playbook. Use when the user wants to understand shared conventions across a corpus of repos or bootstrap a new project from existing engineering taste.
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

For each signal family, record which repos show the pattern and classify:

- **Strong** (3+ repos) → top-level section in output
- **Moderate** (2 repos) → Style Guidelines mention
- **Single** (1 repo) → Repo-Specific Notes only

Analyze all 10 families: CLI surface, Architecture/data, Testing, Tooling/shipping,
Documentation, Code philosophy, Code conventions, Project structure, AI collaboration,
MCP integration.

See **[SIGNAL-FAMILIES.md](SIGNAL-FAMILIES.md)** for the complete signal definitions.

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

Fetches reference files from the thariqs HTML effectiveness corpus, selects appropriate visual
components, and writes a self-contained `TASTE.html` with at least 5 distinct component types.
The result should feel like an editorial analysis artifact — scannable, visual, and opinionated.

See **[HTML-GENERATION.md](HTML-GENERATION.md)** for the full design system reference,
component selection guide, and rendering rules.

### Step 8 — Generate TASTE-SLIDES.html (only if `--slides` was passed)

Produces an 8–12 slide browser-native deck using scroll-snap, covering: title, scope, top
patterns, pattern deep-dives, project structure, tooling/CI, best practices, anti-patterns,
playbook quickstart, and close.

See **[HTML-GENERATION.md](HTML-GENERATION.md)** (Slides section) for the full rendering rules.

## Output Template

See **[OUTPUT-TEMPLATE.md](OUTPUT-TEMPLATE.md)** for the exact TASTE.md output format.

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
