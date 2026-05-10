# taste

A skill for AI agents that extracts engineering conventions from a corpus of repositories.

Give it a list of repos. It clones them, samples key files, and produces a `TASTE.md` — a structured analysis of the shared patterns, best practices, style guidelines, and product philosophy across the set.

```
/taste steipete/sonoscli openclaw/wacli steipete/oracle
```

→ Produces `TASTE.md` in your current directory.

Add `--html` to also get a polished `TASTE.html` styled with the [thariqs design system](https://thariqs.github.io/html-effectiveness/):

```
/taste --html steipete/sonoscli openclaw/wacli steipete/oracle
```

→ Produces both `TASTE.md` and `TASTE.html`.

Add `--slides` (requires `--html`) to also get a fullscreen slide deck `TASTE-SLIDES.html`:

```
/taste --html --slides steipete/sonoscli openclaw/wacli steipete/oracle
```

→ Produces `TASTE.md`, `TASTE.html`, and `TASTE-SLIDES.html`.

Add `--output <dir>` to write all outputs to a specific directory instead of `./`:

```
/taste --html --output ~/reports steipete/sonoscli openclaw/wacli steipete/oracle
```

→ Writes `TASTE.md` (and `TASTE.html` if `--html`) into `~/reports/`.

---

## What It Produces

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

---

## Install

### Claude Code

```bash
mkdir -p ~/.claude/skills/taste
curl -fsSL https://raw.githubusercontent.com/pavelsimo/taste/main/SKILL.md \
  -o ~/.claude/skills/taste/SKILL.md
```

Then invoke in any session:

```
/taste owner/repo1 owner/repo2 owner/repo3
```

### Cursor

Place `SKILL.md` in your project's `.cursor/rules/taste.mdc` or install it as a personal rule. Invoke with the `/taste` command in Cursor's agent panel.

### GitHub Copilot

Copy the contents of `SKILL.md` into `.github/copilot-instructions.md` in a project where you want taste analysis available, or follow Copilot's skill installation documentation.

### OpenAI Codex / other agents

Any agent that supports `SKILL.md` or `AGENTS.md` can use this skill. Drop `SKILL.md` into the appropriate skills directory for your agent.

---

## Usage

### GitHub repos (shorthand)

```
/taste steipete/sonoscli openclaw/wacli steipete/oracle
```

### Full GitHub URLs

```
/taste https://github.com/steipete/sonoscli https://github.com/openclaw/wacli
```

### Local paths

```
/taste ~/Projects/myapp1 ~/Projects/myapp2
```

### Mixed

```
/taste openclaw/wacli ~/Projects/my-local-tool https://github.com/some/other-repo
```

---

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

---

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

---

## License

MIT
