# run-if-changed

Detect changed files and expose named path-filter results as GitHub Actions
outputs. Filters support glob patterns, exclusions, change statuses, and inline or
file-based YAML configuration.

## Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: read
    steps:
      - uses: actions/checkout@v6

      - id: changes
        uses: rselbach/actions/run-if-changed@v1
        with:
          filters: |
            backend:
              - 'backend/**'
            frontend:
              - 'frontend/**'
              - '!frontend/**/*.md'

      - if: steps.changes.outputs.backend == 'true'
        run: ./scripts/test-backend
```

For pull-request events, the action uses the GitHub API when `token` is set. For
other events, or when `token` is empty, it compares Git refs with the local
`git` checkout and fetches additional history when needed.

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `filters` | yes | | Inline YAML filters or the path to a YAML filter file. |
| `token` | no | `${{ github.token }}` | Token used to list pull-request files through the GitHub API. Set to an empty string to use `git`. |
| `working-directory` | no | repository root | Directory containing the checkout to inspect. |
| `ref` | no | event ref | Git ref whose changes should be detected. Ignored for pull-request events. |
| `base` | no | default branch | Git ref to compare against. Use `HEAD` to inspect staged and unstaged local changes. |
| `list-files` | no | `none` | File-list output format: `none`, `csv`, `json`, `shell`, or `escape`. |
| `initial-fetch-depth` | no | `100` | Initial history depth fetched while finding a merge base. |
| `predicate-quantifier` | no | `some` | Pattern behavior: `some`, `every`, or `some-with-excludes`. |

## Filter syntax

A basic filter is a list of glob patterns:

```yaml
frontend:
  - 'web/**'
  - '!web/**/*.md'
```

Use `predicate-quantifier: some-with-excludes` when exclusions must take
precedence over every inclusion. Filters can also select change statuses:

```yaml
configuration:
  - added|modified:
      - '**/*.yaml'
      - '**/*.json'
removed:
  - deleted: '**/*'
```

Supported statuses are `added`, `copied`, `deleted`, `modified`, `renamed`, and
`unmerged`.

## Outputs

For every filter named `<filter>`, the action sets:

- `<filter>`: `true` when at least one file matched, otherwise `false`.
- `<filter>_count`: number of matching files.
- `<filter>_files`: matching paths in the selected `list-files` format; omitted
  when `list-files` is `none`.

The `changes` output is a JSON array containing every matching filter name.
Avoid naming a filter `changes`, because that would conflict with this output.
