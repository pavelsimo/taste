# TASTE.md Output Template

Use this exact template structure when writing `TASTE.md`. Be prescriptive: write "do X because Y", not "they did X". Keep repo-specific observations in the repo-specific notes section; everything else should be cross-repo synthesis.

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

---

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
