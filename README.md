# ReviewGate Action

Pre-merge quality gate for AI-generated code, as a GitHub Action. Reviews the PR diff in parallel across security / logic / perf / AI-smell dimensions, validates findings with a counter-evidence judge, posts a summary comment, and fails the check on high-confidence issues — while an unfinished review degrades to WARN instead of faking a PASS.

Engine: [dengmengmian/ReviewGate](https://github.com/dengmengmian/ReviewGate) (MIT, Rust). Docs: [reviewgate site](https://dengmengmian.github.io/ReviewGate/) · [benchmarks](https://dengmengmian.github.io/ReviewGate/benchmarks.html).

## Usage

Create `.github/workflows/reviewgate.yml`:

```yaml
name: ReviewGate
on:
  pull_request:

permissions:
  contents: read
  pull-requests: write   # needed for the PR summary comment

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          fetch-depth: 0   # full history, needed to diff base...head

      - uses: dengmengmian/reviewgate-action@v0
        env:
          REVIEWGATE_API_KEY: ${{ secrets.REVIEWGATE_API_KEY }}
        with:
          dimensions: all
          fail-on: block
          comment: "true"
```

Put your LLM key in repository **Secrets** as `REVIEWGATE_API_KEY`, and commit a `reviewgate.toml` (provider endpoint + model) to the repo root — see the [quick start](https://github.com/dengmengmian/ReviewGate#readme).

## Inputs

| Input | Default | Description |
|---|---|---|
| `dimensions` | `all` | `all` or comma-separated `security,perf,logic,style,ai_smell` |
| `fail-on` | `block` | Which verdict fails CI: `block` / `warn` / `never` |
| `comment` | `true` | Post a summary comment on the PR |
| `timeout` | `300` | Wall-clock seconds per dimension; a timed-out dimension is skipped and reported, never silently passed |
| `config` | `reviewgate.toml` | Path to the config file |
| `version` | `latest` | ReviewGate release to install |
| `intent` | `off` | `auto` = use PR title+body as acceptance criteria; `off`; or a path to an intent doc |

## Exit semantics

`0` clean · `1` gate blocked (or WARN with `fail-on: warn`) · `2` tool failure. Incomplete reviews (timeout / oversized diff) degrade to WARN — this action never reports an unfinished review as a clean pass.
