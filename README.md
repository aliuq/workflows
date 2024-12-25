# GitHub Workflows

可复用的 GitHub 工作流集合。

## Workflows

### sync-with-notify

用于同步上游仓库更新并通过 Telegram 发送通知的工作流。

#### 功能特点

- 自动同步上游仓库的更新
- 支持私有仓库同步
- 可配置同步分支
- 支持 Telegram 通知
- 提供同步状态输出

#### 使用方法

```yaml
name: Sync from upstream

on:
  schedule:
    - cron: "0 */8 * * *"  # 每8小时运行一次
  workflow_dispatch:        # 支持手动触发
    inputs:
      test_mode:
        description: "测试模式"
        type: string
        default: "true"



jobs:
  call-workflow-sync:
    uses: aliuq/workflows/.github/workflows/sync-with-notify.yml@master
    permissions:
      contents: write
    with:
      upstream_repo: owner/repo  # 上游仓库
      test_mode: ${{ inputs.test_mode }}
    secrets:
      TOKEN: ${{ secrets.GITHUB_TOKEN }}
      BOT_TOKEN: ${{ secrets.BOT_TOKEN }}     # Telegram Bot Token
      CHAT_ID: ${{ secrets.CHAT_ID }}         # Telegram Chat ID
      REPLY_TO_MESSAGE_ID: ${{ secrets.REPLY_TO_MESSAGE_ID }}  # Telegram Topic ID
```

#### 输入参数

| 参数名 | 说明 | 必填 | 默认值 |
|--------|------|------|--------|
| upstream_repo | 上游仓库地址 | 是 | - |
| upstream_branch | 上游分支 | 否 | master |
| target_branch | 目标分支 | 否 | master |
| test_mode | 测试模式 | 否 | false |
| persist-credentials | 私有仓库需要设置为 true | 否 | false |
| shallow_since | Git 浅克隆时间范围 | 否 | 1 month ago |

#### 密钥配置

| 密钥名 | 说明 | 必填 |
|--------|------|------|
| TOKEN | GitHub Token | 是 |
| UPSTREAM_REPO_SECRET | 私有上游仓库的访问令牌 | 否 |
| BOT_TOKEN | Telegram Bot Token | 否 |
| CHAT_ID | Telegram Chat ID | 否 |
| REPLY_TO_MESSAGE_ID | Telegram 回复消息 ID | 否 |

#### 输出

| 名称 | 说明 | 类型 |
|------|------|------|
| has_new_commits | 是否有新提交 | boolean |

#### 使用输出示例

```yaml
jobs:
  call-workflow-sync:
    uses: aliuq/workflows/.github/workflows/sync-with-notify.yml@master
    # ...配置省略...

  post-sync:
    needs: call-workflow-sync
    if: needs.call-workflow-sync.outputs.has_new_commits == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: 处理新提交
        run: echo "检测到新提交!"
```

### sync

用于同步上游仓库代码的基础工作流。

#### 功能特点

- 自动同步上游仓库更新
- 支持定时和手动触发
- 支持私有仓库同步
- 可配置同步分支

#### 输入参数

| 参数名 | 说明 | 必填 | 默认值 |
|--------|------|------|--------|
| persist-credentials | 私有仓库需要设置为 true | 否 | false |
| target_branch | 目标分支 | 否 | master |
| upstream_branch | 上游分支 | 否 | master |
| upstream_repo | 上游仓库地址 | 是 | - |
| host_domain | Git 托管域名 | 否 | github.com |
| shallow_since | Git 浅克隆时间范围 | 否 | 1 month ago |
| test_mode | 测试模式 | 否 | false |
| git_config_user | Git 提交用户名 | 否 | GH Action - Upstream Sync |
| git_config_email | Git 提交邮箱 | 否 | <action@github.com> |

#### 密钥配置

| 密钥名 | 说明 | 必填 |
|--------|------|------|
| token | GitHub Token | 是 |
| upstream_repo_secret | 私有上游仓库的访问令牌 | 否 |

#### 输出

| 名称 | 说明 | 类型 |
|------|------|------|
| has_new_commits | 是否有新提交 | boolean |

#### 使用示例

```yaml
jobs:
  call-workflow-sync:
    uses: aliuq/workflows/.github/workflows/sync.yml@master
    with:
      upstream_repo: owner/repo
      target_branch: main
      shallow_since: "2 months ago"
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}

  post-sync:
    needs: call-workflow-sync
    if: needs.call-workflow-sync.outputs.has_new_commits == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "有新的更新!"
```

### build-image-notify

用于构建和发布 Docker 镜像并发送通知的工作流。

#### 功能特点

- 支持多平台镜像构建
- 自动标记版本号
- 支持 Docker Hub 和 GitHub Container Registry
- 构建完成后发送 Telegram 通知
- 支持自定义镜像标签和构建参数

#### 输入参数

| 参数名 | 说明 | 必填 | 默认值 |
|--------|------|------|--------|
| images | 要构建的 Docker 镜像列表 | 是 | - |
| tags | 镜像标签规则 | 是 | - |
| labels | Docker 镜像标签 | 否 | - |
| context | Docker 构建上下文 | 否 | . |
| file | Dockerfile 路径 | 否 | Dockerfile |
| push | 是否推送镜像 | 否 | false |
| platforms | 构建平台列表 | 否 | linux/amd64,linux/arm64 |

#### 密钥配置

| 密钥名 | 说明 | 必填 |
|--------|------|------|
| DOCKERHUB_USERNAME | Docker Hub 用户名 | 是 |
| DOCKERHUB_TOKEN | Docker Hub 访问令牌 | 是 |
| TOKEN | GitHub Token | 是 |
| BOT_TOKEN | Telegram Bot Token | 否 |
| CHAT_ID | Telegram Chat ID | 否 |
| REPLY_TO_MESSAGE_ID | Telegram 回复消息 ID | 否 |

#### 使用示例

```yaml
jobs:
  call-workflow-build:
    uses: aliuq/workflows/.github/workflows/build-image-notify.yml@master
    permissions:
      contents: read
      packages: write
    with:
      images: |
        username/image
        ghcr.io/username/image
      tags: |
        type=semver,pattern={{version}}
        type=semver,pattern={{major}}.{{minor}}
        type=semver,pattern={{major}}
        type=raw,value=latest
        type=sha,enable=true,priority=100,prefix=,suffix=,format=short

      push: false
      platforms: linux/amd64,linux/arm64
    secrets:
      TOKEN: ${{ secrets.GITHUB_TOKEN }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
      BOT_TOKEN: ${{ secrets.BOT_TOKEN }}
      CHAT_ID: ${{ secrets.CHAT_ID }}
      REPLY_TO_MESSAGE_ID: ${{ secrets.REPLY_TO_MESSAGE_ID }}
```

### sync-with-notify + build-image-notify

用于同步上游仓库的代码到自己的仓库，并构建 Docker 镜像，发送通知

> 修改 UserName/Repo 为自己的上游仓库
> 修改 app/name 和 ghcr.io/username/name 为自己的镜像名称
> 测试后修改 push 为 true，即可自动推送镜像

```yaml
name: Sync from upstream

on:
  schedule:
    - cron: "0 */8 * * *" # 每 8 小时运行一次
  push:
    branches:
      - master
    paths:
      - src/** # 仅当 src 目录发生变化时运行
  workflow_dispatch:
    inputs:
      test_mode:
        description: "Fork Sync Test Mode"
        type: string
        default: "true"

jobs:
  call-workflow-sync:
    uses: aliuq/workflows/.github/workflows/sync-with-notify.yml@master
    permissions:
      contents: write
      packages: write
    with:
      upstream_repo: UserName/Repo
      # upstream_branch: main
      # target_branch: main

      # shallow_since: "2 years ago"
      test_mode: ${{ inputs.test_mode }}
    secrets:
      TOKEN: ${{ secrets.GITHUB_TOKEN }}
      BOT_TOKEN: ${{ secrets.BOT_TOKEN }}
      CHAT_ID: ${{ secrets.CHAT_ID }}
      REPLY_TO_MESSAGE_ID: ${{ secrets.REPLY_TO_MESSAGE_ID }}

  call-workflow-build:
    needs: call-workflow-sync
    if: needs.call-workflow-sync.outputs.has_new_commits == 'true'
    uses: aliuq/workflows/.github/workflows/build-image-notify.yml@master
    permissions:
      contents: read
      packages: write
    with:
      images: |
        app/name
        ghcr.io/username/name
      tags: |
        type=semver,pattern={{version}}
        type=semver,pattern={{major}}.{{minor}}
        type=semver,pattern={{major}}
        type=raw,value=latest
        type=sha,enable=true,priority=100,prefix=,suffix=,format=short
      push: false
    secrets:
      TOKEN: ${{ secrets.GITHUB_TOKEN }}
      BOT_TOKEN: ${{ secrets.BOT_TOKEN }}
      CHAT_ID: ${{ secrets.CHAT_ID }}
      REPLY_TO_MESSAGE_ID: ${{ secrets.REPLY_TO_MESSAGE_ID }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```
