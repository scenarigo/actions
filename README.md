# Scenarigo GitHub Actions

Composite actions for the scenarigo users.

- `install/`: Ensure Go (stable if missing) and install scenarigo CLI (skip when already present with the requested version; `latest` also skips reinstall if present).
- `run/`: Install dependencies (via `install`) and execute `scenarigo run`.
- `build-plugin/`: Install dependencies (via `install`), build scenarigo plugins, and expose their paths as outputs.

## Action: scenarigo/actions/run@v1

Installs scenarigo, optionally builds plugins, and runs scenarios.

```yaml
name: CI
on:
  push:
  pull_request:
jobs:
  scenarigo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run scenarigo scenarios
        uses: scenarigo/actions/run@v1
        with:
          scenarigo-version: "latest" # e.g. v0.23.0
          working-directory: "."
          run-args: "" # e.g. --config scenarigo.yaml --vars vars.yaml
          build-plugins: "true"      # set false to skip plugin build
          build-args: ""             # e.g. --config plugin.yaml
          list-args: ""              # optional flags for plugin list
```

Inputs:
- `scenarigo-version`: scenarigo CLI version (default: `latest`).
- `working-directory`: Directory containing scenarigo configuration and scenarios (default: `.`).
- `run-args`: Extra args for `scenarigo run` (default: empty).
- `build-plugins`: Whether to build plugins before running (default: `true`).
- `build-args`: Extra args for `scenarigo plugin build` (default: empty).
- `list-args`: Extra args for `scenarigo plugin list` (default: empty).
- Behavior: Uses the `install` action. If `go` is already available, setup-go is skipped; otherwise installs stable. `scenarigo` installs only when not already present or when version differs (for `latest`, skips reinstall if present). If the `go` version/platform reported by `go version` does not match the tail of `scenarigo version`, scenarigo is reinstalled. Plugin build reuses the same `scenarigo` install.

Outputs:
- `plugin-paths`: Newline-separated plugin paths from `scenarigo plugin list` (empty if plugin build is skipped).

## Action: scenarigo/actions/build-plugin@v1

Build scenarigo plugins and expose plugin paths via outputs.

```yaml
name: Build Plugins
on:
  workflow_dispatch:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build plugins
        id: build
        uses: scenarigo/actions/build-plugin@v1
        with:
          scenarigo-version: "latest" # e.g. v0.15.0
          working-directory: "./plugins"
          build-args: "" # e.g. --config plugin.yaml
          list-args: ""  # optional flags for list
      - name: Show plugin paths
        run: |
          echo "Built plugin paths:"
          echo "${{ steps.build.outputs.paths }}"
```

Inputs:
- `scenarigo-version`: scenarigo CLI version (default: `latest`).
- `working-directory`: Directory containing plugin sources/config (default: `.`).
- `build-args`: Extra args for `scenarigo plugin build` (default: empty).
- `list-args`: Extra args for `scenarigo plugin list` (default: empty).
- Behavior: Uses the `install` action. If `go` is already available, setup-go is skipped; otherwise installs stable. `scenarigo` installs only when not already present or when version differs (for `latest`, skips reinstall if present). If the `go` version/platform reported by `go version` does not match the tail of `scenarigo version`, scenarigo is reinstalled.

Outputs:
- `paths`: Newline-separated plugin paths returned by `scenarigo plugin list`.

## Notes

- Checkout the target repository before invoking these actions (`actions/checkout@v4`).
- Provide any required config/vars files via the corresponding args. Add lint/build steps separately in your workflow as needed.
- A runnable example lives in `examples/get-repo`; CI exercises it via the `run` action.

## Action: scenarigo/actions/install@v1

Ensure Go and scenarigo CLI are available.

Inputs:
- `scenarigo-version`: scenarigo CLI version (default: `latest`; skips reinstall if already present when `latest`).

Behavior:
- Skips setup-go when `go` is already available; otherwise installs stable Go.
- Installs scenarigo only when absent, version differs, or when the Go version/platform from `go version` differs from the tail of `scenarigo version` (adds install path to `GITHUB_PATH`).
