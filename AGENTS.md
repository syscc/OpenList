# AGENTS.md

本仓库是 `syscc/OpenList` 个人 fork，目标是跟随官方 OpenList，同时维护自用 beta 镜像。后续代理处理本仓库时请遵守以下规则。

## 本地构建限制

- 测试和排查阶段禁止在本地将源码编译为二进制程序。
- 不要在本地运行 `go build`、`bash build.sh`、`docker build`、发布打包、镜像构建等会生成编译产物的命令。
- 本地只做源码检查、YAML/文本校验、Git/GitHub 状态检查。
- 构建验证以 GitHub Actions 为准。

## Remote 用法

- `origin` 指向官方 `OpenListTeam/OpenList`，用于读取官方状态，不要向它 push。
- `fork` 指向 `syscc/OpenList`，需要推送时只推这里。
- `xrgzs` 只作为参考 remote，不要向 `xrgzs/OpenList` 创建 PR 或推送。

## 分支策略

- `main`：跟随官方主线，但保留本 fork 必需的管理文件和 workflow。不要把未合并 PR 的功能代码长期放在 `main`。
- `beta`：自动生成的自用运行分支，内容为 `main` 加上当前仍未合并的自用 PR。
- `feat/*`、`fix/*`：给官方提交 PR 的功能分支，应基于官方 `OpenListTeam/OpenList:main` 创建，避免带入本 fork 的 README/workflow/管理提交。

当前期望状态：

```text
main = 官方 beta 成功构建过的基线 + fork 管理文件
beta = main + syscc 在官方仓库中 open、非 draft、head 来自 syscc/OpenList 的 PR
```

## Beta PR 规则

- 默认情况下，`Sync beta branch` 会自动收集 `OpenListTeam/OpenList` 中 author 为 `syscc`、状态为 open、非 draft、head repo 为 `syscc/OpenList` 的 PR，并叠加到 `beta`。
- `.github/beta-prs.txt` 是可选限制列表：
  - 文件里没有 PR 号时，自动使用所有符合条件的 open PR。
  - 文件里写了 PR 号时，只叠加这些 PR。
- 官方合并某个 PR 后，该 PR 不再是 open，下一次生成 `beta` 时会自动不再叠加。

## Workflow 说明

- `Sync upstream main`
  - 每小时检查官方 `OpenListTeam/OpenList` 最新成功的 `Beta Release (Docker)` push run。
  - 只把 `main` 同步到官方已经成功构建 beta Docker 的 SHA。
  - 同步成功后触发 `Sync beta branch`。

- `Sync beta branch`
  - 从 `main` 重新生成 `beta`。
  - 自动叠加符合规则的未合并 PR。
  - 写入 `.github/beta-state.txt` 记录 base 和 PR 状态。
  - 只有生成后的 `beta` tree 有变化才 push。

- `Beta Release (Docker)`
  - 只从 `beta` 分支构建 GHCR 镜像。
  - 只在 Go、Docker、build、public 等影响镜像内容的路径变化时触发。
  - 目标镜像为 `ghcr.io/syscc/openlist` 和 `ghcr.io/syscc/alist`。

- `Beta Release builds`
  - 只从 `beta` 分支构建 beta release 二进制包。
  - 同样带 paths 过滤。

- `Test Build`
  - 用于 PR 构建检查。

## 常用检查

完整检查时优先执行：

```bash
git status --short
git ls-remote --heads fork main beta
ruby -e 'require "yaml"; ARGV.each { |f| YAML.load_file(f); puts "ok #{f}" }' .github/workflows/*.yml
git diff --check refs/heads/main refs/heads/beta
gh workflow list --repo syscc/OpenList --all
gh run list --repo syscc/OpenList --limit 12
gh pr list --repo OpenListTeam/OpenList --author syscc --state open
```

必要时检查：

```bash
git diff --stat refs/heads/main..refs/heads/beta
cat .github/beta-state.txt
```

## 操作注意

- 不要把 `beta` 分支直接拿去给官方开 PR。
- 不要从 `beta` 开官方 PR。
- 不要因为 GitHub compare 页面出现 `xrgzs/OpenList/compare/main...syscc:beta` 就创建 PR；那只是比较页。
- `main` 的 force push 只在明确需要重写 fork 管理历史时执行，并且必须确认不会覆盖用户未保存工作。
- `beta` 是生成分支，可以由 workflow 或维护操作 force push。
- 本地未跟踪的 `.spec-workflow/` 不要提交，除非用户明确要求。
