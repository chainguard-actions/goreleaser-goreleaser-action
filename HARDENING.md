<!-- markdownlint-disable -->

# Hardening Report: goreleaser--goreleaser-action/v7.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **goreleaser--goreleaser-action/v7.2.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` steps in release-major-tag.yml directly interpolate `workflow_dispatch` user-controlled inputs into shell commands via `${{ github.event.inputs.major_version }}` and `${{ github.event.inputs.target }}`. This is a sub-rule (a) violation: the expressions are expanded by the Actions template engine before the shell ever sees them, allowing an attacker with workflow_dispatch access to inject arbitrary shell commands.

Offending lines:
- `run: git tag -f ${{ github.event.inputs.major_version }} ${{ github.event.inputs.target }}`
- `run: git push origin ${{ github.event.inputs.major_version }} --force`

Fix: move the inputs into `env:` variables and reference them as quoted shell variables, e.g.:
```yaml
env:
  MAJOR_VERSION: ${{ github.event.inputs.major_version }}
  TARGET: ${{ github.event.inputs.target }}
run: |
  git tag -f "$MAJOR_VERSION" "$TARGET"
  git push origin "$MAJOR_VERSION" --force
```

Locations:

- `.github/workflows/release-major-tag.yml:36`
- `.github/workflows/release-major-tag.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/.github/workflows/release-major-tag.yml. The 'Move' step now uses env vars MAJOR_VERSION and TARGET (both quoted in the shell command) instead of directly interpolating ${{ github.event.inputs.major_version }} and ${{ github.event.inputs.target }}. The 'Push' step now uses env var MAJOR_VERSION (quoted) instead of directly interpolating ${{ github.event.inputs.major_version }}. The run-name: and step name: fields retain the ${{ }} expressions as they are display strings, not shell commands.

