# pi-release-notes

Generate Markdown release notes from git history using
[`pi`](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent) and
an LLM provider such as DeepSeek.

The action is designed to be a single step inside an existing release workflow.
It writes a Markdown file that can be passed to `softprops/action-gh-release` as
`body_path`.

## Usage

```yaml
name: Release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - id: release_notes
        uses: rselbach/actions/pi-release-notes@v1
        env:
          DEEPSEEK_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
        with:
          project-name: nomadl
          project-description: Terminal UI for browsing Nomad services/jobs and tailing allocation logs.
          output-file: build/release-notes.md

      - name: Append installation notes
        run: cat .github/release-installation.md >> build/release-notes.md

      - uses: softprops/action-gh-release@v2
        with:
          name: ${{ github.ref_name }}
          body_path: ${{ steps.release_notes.outputs.body-path }}
          files: dist/*
```

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `project-name` | yes | | Project name used in the prompt. |
| `project-description` | yes | | Short project description used in the prompt. |
| `current-tag` | no | `GITHUB_REF_NAME` | Current release tag. |
| `previous-tag` | no | auto-detected | Previous release tag. |
| `tag-prefix` | no | `v` | Prefix used when auto-detecting the previous tag. |
| `tag-pattern` | no | | Git tag glob. Overrides `tag-prefix` when set. |
| `output-file` | no | `build/release-notes.md` | Generated Markdown output path. |
| `prompt-file` | no | bundled prompt | Custom prompt file path. |
| `additional-context` | no | | Extra maintainer context to include. |
| `additional-context-file` | no | | File containing extra maintainer context. |
| `provider` | no | `deepseek` | `pi` provider name. |
| `model` | no | `deepseek-v4-flash` | `pi` model name or provider/model pattern. |
| `thinking` | no | `off` | `pi` thinking level. |
| `pi-version` | no | `0.70.2` | `@mariozechner/pi-coding-agent` version. |
| `node-version` | no | `22` | Node.js version used to install and run `pi`. |

## Outputs

| Name | Description |
| --- | --- |
| `body-path` | Path to the generated Markdown release notes. |
| `current-tag` | Current release tag used for the notes. |
| `previous-tag` | Previous release tag used for the notes. |
| `range` | Git revision range used for the notes. |

## Provider Setup

For the default DeepSeek configuration, set a repository secret named
`DEEPSEEK_API_KEY` and pass it in the action environment.

```yaml
env:
  DEEPSEEK_API_KEY: ${{ secrets.DEEPSEEK_API_KEY }}
```

To use another provider, set `provider`, `model`, and the relevant provider API
key environment variable expected by `pi`.

## Notes

- The action runs `pi` with `--no-tools` and `--no-session`.
- Release-specific installation notes should be appended by the caller, because
  installation instructions are usually project-specific.
- Use `actions/checkout` with `fetch-depth: 0` when possible. The action tries
  to fetch tags itself, but a full checkout makes release range detection more
  reliable.
