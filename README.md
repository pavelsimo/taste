# taste skill

A skill for agents that extracts engineering conventions from a corpus of repositories. Works with Claude Code, Cursor, GitHub Copilot, and OpenAI Codex.

## Usage

```
/taste steipete/sonoscli openclaw/wacli steipete/oracle
```

Clones each repo, samples key files, and produces a `TASTE.md` — a structured analysis of shared patterns, best practices, style guidelines, and product philosophy.

Pass `--html` to also get a polished `TASTE.html` styled with the [thariqs design system](https://thariqs.github.io/html-effectiveness/):

```
/taste --html steipete/sonoscli openclaw/wacli steipete/oracle
```

Pass `--slides` (requires `--html`) to also get a fullscreen slide deck `TASTE-SLIDES.html`:

```
/taste --html --slides steipete/sonoscli openclaw/wacli steipete/oracle
```

Pass `--output <dir>` to write all outputs to a specific directory:

```
/taste --html --output ~/reports steipete/sonoscli openclaw/wacli steipete/oracle
```

Input accepts GitHub shorthands, full URLs, and local paths — mixed freely:

```
/taste openclaw/wacli ~/Projects/my-local-tool https://github.com/some/other-repo
```

## Output

`TASTE.md` follows a consistent structure:

| Section | What it contains |
|---------|-----------------|
| **Scope** | Table of repos analyzed, languages, frameworks, domains |
| **High-Level Read** | 2–3 paragraph synthesis of the dominant taste |
| **Recurring Patterns** | Cross-repo signals, ordered by prevalence |
| **Style Guidelines** | Command design, flag conventions, error handling, output |
| **Code Style** | Language-specific and general principles |
| **Tooling & Shipping** | Build, testing, release, CI, distribution |
| **Product Taste** | Philosophy — what the author optimizes for |
| **Best Practices** | 8–15 prescriptive "do X because Y" items |
| **Anti-Patterns** | What this style consistently avoids |
| **Repo-Specific Notes** | What makes each repo worth studying |
| **Practical Playbook** | Checklists for starting, outputting, releasing |

## Installation

```bash
npx skills@latest add pavelsimo/taste
```

## How It Works

1. **Clone** each repo with `git clone --depth 1` into `~/.taste/corpus/<timestamp>/`
2. **Inventory** language composition, frameworks, and key files per repo
3. **Sample** up to 50 prioritized files per repo: entry points → config → tests → CI → docs
4. **Detect signals** across ten families: CLI surface, architecture, testing, tooling, docs, code philosophy, concrete conventions, project structure, AI collaboration, MCP integration
5. **Synthesize** into a `TASTE.md` written to your current directory

Repos are cached in `~/.taste/corpus/` — re-running with the same URLs is fast. To force a fresh clone, delete that directory:

```bash
rm -rf ~/.taste/corpus/
```

## Signal Taxonomy

The skill looks for patterns across ten families:

- **CLI surface** — global flags, output formats, completions, `doctor` commands, version injection
- **Architecture & data** — persistence strategy, config resolution, secret storage, local-first patterns
- **Testing discipline** — framework, coverage thresholds, test distribution, race detection
- **Tooling & shipping** — build system, linter, release automation, CI pipeline shape, documentation deployment, distribution channels
- **Documentation** — README structure, badge style, docs site, example density, troubleshooting docs
- **Code philosophy** — entry point size, error handling style, naming conventions, abstraction level
- **Concrete code conventions** — import ordering, docstring style, typing posture, error idioms, test idioms
- **Project structure** — top-level directory conventions, entry point placement, module boundaries
- **AI collaboration** — presence of `CLAUDE.md`/`AGENTS.md`/`CODEX.md`, documented agent rules and conventions
- **MCP integration** — SDK used, transport, tool registration pattern, dual-mode CLI+server behavior

Signals appearing in 3+ repos become top-level recurring patterns. Signals in 2 repos appear in style guidelines. Single-repo observations go into repo-specific notes.

## Contributing

Open an issue or pull request. Keep commits atomic and follow the [commit conventions](https://github.com/pavelsimo/commit).

## License

MIT
