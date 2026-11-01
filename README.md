<div align="center">

<img src="assets/releasewave-logo.svg" alt="ReleaseWave — three flowing blue and cyan waves" width="144" />

# ReleaseWave

### Your code changed. Let your release notes catch up.

**An AI-powered changelog generator that understands real Git diffs — not just commit messages.**

Turn your changes into developer changelogs, user-facing release notes, and launch announcements from one command.

[![PyPI](https://img.shields.io/pypi/v/releasewave?style=flat-square&label=PyPI&color=2563eb)](https://pypi.org/project/releasewave/)
[![Python](https://img.shields.io/badge/Python-3.10%2B-0ea5e9?style=flat-square&logo=python&logoColor=white)](pyproject.toml)
[![License: MIT](https://img.shields.io/badge/License-MIT-334155?style=flat-square)](LICENSE)
[![GitHub Action](https://img.shields.io/badge/GitHub-Action-2563eb?style=flat-square&logo=githubactions&logoColor=white)](action.yml)

**[Quick start](#quick-start) · [Features](#why-releasewave) · [CLI](#command-line-reference) · [GitHub Action](#github-action) · [Configuration](#configuration)**

</div>

---

## From code changes to a clear release story

Commit messages often describe *what someone typed*, not *what changed for your users*. ReleaseWave reads the commits **and the actual code diffs** between two Git references, analyzes them with an LLM, and writes the kind of update each audience needs.

<div align="center">

```text
Git commits + diffs  →  Context-aware analysis  →  Release-ready writing
                                                   ├─ Developer changelog
                                                   ├─ User release notes
                                                   └─ Launch announcement
```

</div>

### Three outputs. One workflow.

| Audience | Generated file | What it contains |
|:--|:--|:--|
| **Developers** | `CHANGELOG-developer.md` | Technical changes, fixes, and relevant implementation details |
| **Users** | `RELEASE-NOTES.md` | Clear, readable descriptions of what's new and improved |
| **Community** | `TWEET.txt` | A concise, shareable release announcement |

See real output samples: **[Developer changelog](examples/CHANGELOG-developer.md)** · **[Release notes](examples/RELEASE-NOTES.md)** · **[Announcement](examples/TWEET.txt)**

## Why ReleaseWave?

| | |
|:--|:--|
| **Diff-first understanding**<br>Analyzes the underlying file changes instead of relying on perfectly formatted commits. | **Smart context handling**<br>Breaks large diffs into manageable chunks for model analysis. |
| **Write once, reach everyone**<br>Generate technical, user-friendly, and announcement-ready versions in a single run. | **Model flexibility**<br>Choose your LLM through LiteLLM, including supported hosted or local providers. |
| **CLI + GitHub Actions**<br>Use it when you need it locally or include it in a release workflow. | **Monorepo-aware**<br>Detects common package layouts and can produce output for individual packages. |

---

## Quick start

**Requirements:** Python 3.10+, Git, an existing Git repository, and access to an LLM provider (or a locally configured model).

### 1. Install

```bash
python -m pip install releasewave
```

The installed commands are `rwave` and `releasewave`.

### 2. Configure your model credentials

The default model is `gemini/gemini-2.5-flash`. Set its provider key in your shell:

```bash
# macOS / Linux
export GEMINI_API_KEY="your-api-key"
```

```powershell
# Windows PowerShell
$env:GEMINI_API_KEY = "your-api-key"
```

You can also use `RWAVE_API_KEY` as a model-independent override, or configure another provider through `--model`. **Never commit API keys to your repository.**

### 3. Generate your release notes

Run from the repository whose changes you want to summarize:

```bash
rwave generate v1.0.0 HEAD
```

Replace `v1.0.0` with an existing starting tag, branch, or commit SHA in **your** repository. `HEAD` refers to the latest checked-out commit.

ReleaseWave writes its three audience-specific files to the current directory by default. If this is your first release, use your initial commit SHA as the starting ref:

```bash
git rev-list --max-parents=0 HEAD
# Copy the returned SHA, then:
rwave generate <initial-commit-sha> HEAD
```

> **Tip:** Your working tree must have commits to compare. ReleaseWave needs *two* Git references; running `rwave generate` with no arguments does not generate a release.

---

## Command-line reference

```bash
# Create all three standard outputs
rwave generate v1.0.0 HEAD

# Generate only selected audiences
rwave generate v1.0.0 HEAD --audiences developer,user

# Write to another directory and export structured analysis
rwave generate v1.0.0 HEAD --output ./release-artifacts --json

# Choose a supported model
rwave generate v1.0.0 HEAD --model gpt-4o-mini

# Prepend the developer changelog to CHANGELOG.md
rwave generate v1.0.0 HEAD --update-changelog

# Generate a starter .rwave.yml in your current directory
rwave init

# Show model suggestions
rwave models
```

| Option | Purpose |
|:--|:--|
| `--model, -m` | Select a model using a LiteLLM-compatible identifier |
| `--audiences, -a` | Comma-separated `developer`, `user`, or `tweet` |
| `--output, -o` | Directory for generated files |
| `--format, -f` | Output format: `markdown`, `json`, or `text` |
| `--repo, -r` | Path to the target Git repository |
| `--update-changelog` | Prepend generated developer notes to `CHANGELOG.md` |
| `--json` | Export full analysis to `releasewave-output.json` |
| `--quiet, -q` | Suppress generated text on standard output |

Run `rwave generate --help` for the full command reference.

---

## GitHub Action

ReleaseWave is also packaged as a [composite GitHub Action](action.yml). For an example you can trigger manually, add `.github/workflows/release-notes.yml` to your repository:

```yaml
name: Generate release notes

on:
  workflow_dispatch:

jobs:
  release-notes:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout full Git history
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate release notes
        uses: sizwinz/releasewave@v1.0.0
        with:
          from_ref: v1.0.0
          to_ref: HEAD
          model: gemini/gemini-2.5-flash
          audiences: developer,user,tweet
          output_dir: .
          api_key: ${{ secrets.GEMINI_API_KEY }}
```

Before running, add **`GEMINI_API_KEY`** to your repository's **Settings → Secrets and variables → Actions** and replace `from_ref` with a reference that exists in the repository being checked out. `v1.0.0` is an existing ReleaseWave Action tag; the `from_ref` shown above is just a Git-history example and must be adapted to your project.

For a release-triggered integration with publishing steps, see the [repository's workflow example](.github/workflows/release-notes.yml).

---

## Configuration

Create a `.rwave.yml` manually, or run `rwave init` to generate one. All sections below are optional.

```yaml
# .rwave.yml
llm:
  model: gemini/gemini-2.5-flash
  temperature: 0.3
  token_budget: 120000

output:
  directory: ./release-artifacts
  audiences:
    - developer
    - user
    - tweet
  format: markdown
  update_changelog: false

monorepo:
  enabled: true

filters:
  exclude_patterns:
    - "*.lock"
    - "dist/*"
    - "node_modules/*"
```

Configuration precedence is **CLI options → environment variables → `.rwave.yml` → defaults**. Use `RWAVE_MODEL` to override the configured model or `RWAVE_API_KEY` to pass a universal model key.

Browse the complete [example configuration](.rwave.yml.example) for additional options, including custom instructions, diff filtering, and monorepo directories.

---

## How it works

1. **Extract:** `git_ops.py` resolves the supplied refs, collects commits, and reads the corresponding file diffs.
2. **Prepare:** `chunker.py` organizes large changes into model-sized context.
3. **Analyze:** `llm.py` and `prompts.py` interpret the changes and produce audience-specific content.
4. **Deliver:** `output.py` writes Markdown or text artifacts, with optional JSON analysis export and changelog updates.

ReleaseWave can fall back to commit-message analysis if diff-based LLM analysis fails. As with any AI-generated release notes, **review the output before publishing**, especially for breaking changes, security fixes, and customer-facing claims.

> **Privacy note:** With a hosted LLM, analyzed code and commit information may be sent to your chosen model provider. Review the diff filters and your provider's data-handling policy before running on sensitive repositories.

---

## Development & contributing

Contributions, bug reports, documentation improvements, and new provider support are welcome.

```bash
git clone https://github.com/sizwinz/releasewave.git
cd releasewave

python -m pip install -e ".[dev]"
python -m pytest
ruff check .
```

To propose a change, create a feature branch, run the relevant tests, and open a [pull request](https://github.com/sizwinz/releasewave/pulls). Found a problem? [Open an issue](https://github.com/sizwinz/releasewave/issues).

---

<div align="center">

**Built for developers who would rather ship code than write changelogs.**

[PyPI](https://pypi.org/project/releasewave/) · [Examples](examples/) · [Issues](https://github.com/sizwinz/releasewave/issues) · [MIT License](LICENSE)

If ReleaseWave helps your release workflow, consider giving the repository a ⭐.

</div>