# Workflows

## Usage

### 同步上游仓库

[sync.yml](./.github/workflows/sync.yml) 用于同步上游仓库的代码到自己的仓库

1. [actions/checkout@v4](https://github.com/actions/checkout)
2. [aormsby/Fork-Sync-With-Upstream-action](https://github.com/aormsby/Fork-Sync-With-Upstream-action)

```yaml
name: "Upstream Sync"
on:
  schedule:
    - cron: "0 */4 * * *" # 每 4 小时运行一次
  workflow_dispatch:
    inputs:
      sync_test_mode:
        description: "Fork Sync Test Mode"
        type: boolean
        default: false

jobs:
  call-workflow-sync:
    uses: aliuq/workflows/.github/workflows/sync.yml@master
    with:
      ref: master
      target_sync_branch: master
      upstream_sync_branch: main
      upstream_sync_repo: aliuq/shs
      sync_test_mode: ${{ inputs.sync_test_mode }}
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```
