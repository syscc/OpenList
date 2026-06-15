# OpenList Personal Beta

一个基于 [OpenListTeam/OpenList](https://github.com/OpenListTeam/OpenList) 的个人实验性 Fork，用于跟进官方主分支、验证自用改动，并自动构建 beta Docker 镜像。

> [!WARNING]
>
> **高风险实验性分支 - 不建议生产使用**
>
> 此仓库是 OpenList 的非官方个人 Fork，可能包含尚未合并到官方仓库的实验代码：
>
> - 可能存在未充分验证的功能实现
> - 可能包含不稳定的实验代码
> - 可能存在 BUG、安全风险或兼容性问题
> - 使用第三方网盘或服务时，可能带来账号限制、封禁、数据丢失等风险
>
> **建议：** 生产环境优先使用 [官方 OpenList 稳定版本](https://github.com/OpenListTeam/OpenList)。
>
> **特别声明：** 此分支的所有代码、构建产物及产生的任何后果与 OpenListTeam 无关。合适的改动会通过 PR 提交到官方仓库，尽量减少与官方主分支的差异。

## 分支策略

本仓库的 `main` 分支用于自用 beta 构建：

- 定时同步官方 `OpenListTeam/OpenList:main`
- 可临时包含本人尚未合并到官方的 PR 或自用改动
- `main` 更新后自动构建并推送 GHCR beta 镜像

如果需要向官方提交 PR，应从官方 `OpenListTeam/OpenList:main` 创建干净分支，避免把自用构建配置或未合并实验改动带入官方 PR。

## Docker 镜像

此仓库仅提供 GHCR 上的 CI beta 镜像，不提供 Docker Hub 镜像。

用于兼容替换原版 AList 的镜像，路径仍使用 `/opt/alist`：

```bash
docker pull ghcr.io/syscc/alist:main
```

OpenList 镜像，路径使用 `/opt/openlist`：

```bash
docker pull ghcr.io/syscc/openlist:beta
```

常用标签：

```text
ghcr.io/syscc/openlist:beta
ghcr.io/syscc/openlist:main
ghcr.io/syscc/openlist:latest
ghcr.io/syscc/openlist:beta-ffmpeg
ghcr.io/syscc/openlist:beta-aria2
ghcr.io/syscc/openlist:beta-aio

ghcr.io/syscc/alist:beta
ghcr.io/syscc/alist:main
ghcr.io/syscc/alist:latest
ghcr.io/syscc/alist:beta-ffmpeg
ghcr.io/syscc/alist:beta-aria2
ghcr.io/syscc/alist:beta-aio
```

为了加快构建速度，Docker beta 镜像仅构建以下平台：

- `linux/amd64`
- `linux/arm64`

如需其他平台，请自行基于源码构建。

## Release

本仓库保留 beta release 构建流程，可生成常见系统架构的二进制压缩包。

注意：本仓库的 beta release 和 Docker 镜像均为实验构建，不代表官方发布版本。

## 反馈

如果问题只出现在本分支的实验功能中，请优先在本仓库内讨论。

如果问题在官方版本中也能复现，请使用官方版本复测后反馈至上游：

- [OpenListTeam/OpenList Issues](https://github.com/OpenListTeam/OpenList/issues)
- [OpenListTeam/OpenList Discussions](https://github.com/OpenListTeam/OpenList/discussions)

## AGPL 授权声明

本软件受 [GNU Affero General Public License v3.0 (AGPL-3.0)](https://www.gnu.org/licenses/agpl-3.0.html) 许可协议保护。您可以自由使用、修改和分发本软件，但必须遵守 AGPL-3.0 的相关条款，包括在分发和提供网络服务时公开源代码。

## 使用条款

- 本项目仅供合法用途，用户不得利用本项目从事任何违法活动。
- 用户应自行承担因使用本项目而产生的所有风险和责任。
- 本项目按“现状”提供，不对其可用性、准确性、兼容性或适用性作任何明示或暗示的保证。
- 所有基于 OpenList 的下游项目必须遵守 AGPL-3.0 协议，包括明确标注来源、保持开源属性并采用兼容的许可方式。

## 免责声明

1. 本项目为个人实验性分支，未经充分测试，可能存在安全漏洞、兼容性问题或数据丢失风险。
2. 本项目的所有代码、构建产物及产生的任何后果与 OpenListTeam 无关。
3. 对于因使用本项目而产生的任何损失或后果，维护者不承担责任。
