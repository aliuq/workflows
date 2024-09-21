# Workflows

## Usage

### 同步上游仓库

```yaml
name: "Upstream Sync"
on:
  schedule:
    - cron: "0 */4 * * *" # 每 4 小时运行一次
  workflow_dispatch:

jobs:
  call-workflow-sync:
    uses: aliuq/workflows/.github/workflows/sync.yml@master
    with:
      ref: master
      target_sync_branch: master
      upstream_sync_branch: main
      upstream_sync_repo: aliuq/shs
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```
