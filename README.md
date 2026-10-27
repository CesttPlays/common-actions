# common-actions

Shared GitHub Actions for the "&lt;Thing&gt; Tracker: Local Zones" addon family
(`achivement-local-tracker`, `mount-local-tracker`, `transmog-local-tracker`).
Each addon repo keeps only thin callers. The callers own the triggers and
schedules, and the logic lives here.

| File | Kind | What it does |
|------|------|--------------|
| `.github/workflows/ci.yml` | reusable workflow | luacheck, headless smoke tests, every `tests/test_*.py` |
| `.github/workflows/release.yml` | reusable workflow | gate on CI/data change, bump TOC, tag, BigWigs packager → GitHub Release + CurseForge |
| `.github/workflows/refresh-data.yml` | reusable workflow | run `tools/<generator>`, open PR, CI → merge → release |
| `.github/workflows/refresh-locales.yml` | reusable workflow | pull CurseForge translations, open PR, CI → merge → release |
| `.github/workflows/sync-phrases.yml` | reusable workflow | push `Locales/enUS.lua` phrases to CurseForge |
| `ci-merge-release/action.yml` | composite action | dispatch `ci.yml` on a bot PR, wait, squash-merge, dispatch `release.yml` |

The addon folder is discovered at runtime (`*_Tracker_Local_Zones`). Secrets
(`CF_API_KEY` in the caller's `deploy` environment) come in through `secrets: inherit`.

## Callers

```yaml
# ci.yml (keep `name: CI`; release.yml's workflow_run matches on it)
jobs:
  ci:
    uses: CesttPlays/common-actions/.github/workflows/ci.yml@main

# release.yml (triggers: workflow_run on CI + workflow_dispatch inputs)
jobs:
  release:
    uses: CesttPlays/common-actions/.github/workflows/release.yml@main
    with:
      data_file: MountData.lua
      notes: "Updated mount data."
    secrets: inherit

# refresh-<thing>-data.yml
jobs:
  refresh:
    uses: CesttPlays/common-actions/.github/workflows/refresh-data.yml@main
    with:
      generator: generate_mount_zones.py
      data_file: MountData.lua
      branch: chore/refresh-mount-data
      title: "chore: refresh mount data (zones + global)"
      build: ${{ inputs.build }}

# refresh-locales.yml / sync-phrases.yml
jobs:
  refresh:
    uses: CesttPlays/common-actions/.github/workflows/refresh-locales.yml@main
    secrets: inherit
```

The caller must grant the permissions the called workflow declares
(`contents`, `pull-requests`, `actions: write` for the refresh flows).

This repo must stay **public** so the private sibling repos can call it.
A private copy would need *Settings → Actions → Access* opened to the owner's
repositories.
