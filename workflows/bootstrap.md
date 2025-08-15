---
description: 
---

```yaml
name: bootstrap
description: "Mid-project bootstrap: import brief, seed Product/Active Context, sweep recent git."
inputs:
  since:
    type: string
    default: "14.days"
    description: "Git history window (e.g., 7.days, 2.weeks, 1.month)"
  install_deps:
    type: boolean
    default: false
    description: "If true, run dependency installation via env reference"
  dry_run:
    type: boolean
    default: false
    description: "If true, show planned actions without mutating ConPort"

steps:
  - id: env
    action: include
    file: .windsurf/workflows/conport/.env.md

  # Initialize (handles projectBrief in .windsurf/conport or repo root)
  - id: initialize
    action: include
    file: .windsurf/workflows/conport/initialize.md
    with:
      dry_run: "{{ inputs.dry_run }}"
      install_deps: "{{ inputs.install_deps }}"

  # Load to verify contexts
  - id: load
    action: include
    file: .windsurf/workflows/conport/load.md

  # Sweep recent git into ConPort (progress entries + active patch)
  - id: sync
    action: include
    file: .windsurf/workflows/conport/sync.md
    with:
      since: "{{ inputs.since }}"
      dry_run: "{{ inputs.dry_run }}"

outputs:
  success:
    status: ok
    message: "ConPort bootstrap complete (since={{ inputs.since }}, install_deps={{ inputs.install_deps }})"
```