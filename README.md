# TokenTicks prompt cost check

A GitHub Action that checks the prompt files in a pull request and posts what
the change does to your monthly AI bill:

> ### TokenTicks — prompt cost change
>
> **+$128.40/month** from 1 prompt file (+214 tokens per call in total).
>
> | File | Model | Tokens | Δ tokens | Calls/day | Δ per month |
> |---|---|---:|---:|---:|---:|
> | `prompts/support.md` | Claude Sonnet 5 | 1,580 → 1,794 | +214 | 10,000 | +$128.40 |
>
> <sub>Input cost only, at list rates with no caching, over 30 days at the call volumes in `.tokenticks.json`. Compared against `origin/main`. Output tokens depend on the task, not the prompt edit, so they are not guessed.</sub>

It runs the [`tokenticks`](https://www.npmjs.com/package/tokenticks) CLI.
Prompts are read and counted on the runner and never uploaded; with a key, the
only network call is a licence check.

## Use it

```yaml
name: Prompt cost
on:
  pull_request:
    paths: ["prompts/**", "**/*.prompt.md", ".tokenticks.json"]

permissions:
  contents: read
  pull-requests: write

jobs:
  cost:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: shekath/tokenticks-action@v1
        with:
          key: ${{ secrets.TOKENTICKS_KEY }}
```

Get a key at [TokenTicks](https://shekath.github.io/Tokenlens/) → profile → CLI &
MCP keys, and add it as the repository secret `TOKENTICKS_KEY`.

## What each plan gets

| | Free (no key) | Pro | Team |
| --- | :---: | :---: | :---: |
| Token budgets per prompt file | ✓ | ✓ | ✓ |
| Waste (Token Trimmer) and cache-order findings | | ✓ | ✓ |
| Cost comment on the pull request | | | ✓ |

Without a Team key the comment step is skipped; nothing is posted.

## Inputs

| Input | Default | What it does |
| --- | --- | --- |
| `key` | | Licence key. Store it as a secret. |
| `lint` | `true` | Run `tokenticks lint` and annotate the changed files. |
| `fail-on-lint` | `true` | Fail the job on an error-level finding. |
| `comment` | `true` | Post or update one cost comment (Team). |
| `base` | the PR's base branch | Ref to compare against. |
| `config` | `./.tokenticks.json` | Config file: model, calls per day, budgets, which files are prompts. |
| `working-directory` | `.` | Where to run. |
| `version` | `0` | `tokenticks` version to run (any npm range). |
| `github-token` | `github.token` | Token used to post the comment. |

## Outputs

| Output | What it holds |
| --- | --- |
| `monthly-delta-usd` | The monthly cost change in USD, when the cost check ran. |
| `comment-file` | Path of the Markdown comment. |

Pull requests from forks do not receive secrets, so they run the free checks.

A shallow checkout is fine: the action fetches the base branch history it needs.# tokenticks-action
