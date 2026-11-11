<p align="center"><img src="assets/icon.svg" width="96" alt="JianShanAgent"></p>

# instance-yys

> JianShanAgent · 实例 · 阴阳师 wiki 的 QQ 机器人这个部署
>
> A JianShanAgent deployment: the QQ bot serving the Onmyoji wiki.

| | |
|---|---|
| 层 | 实例 |
| id | — |
| 包 | 不是一个要发布的包（`pyproject.toml` 只用来搭开发环境） |
| 依赖 | 核心 `jianshanagent`、`site-huiji`、`site-huiji-yys`；版本见 `components.lock` |
| 登记 | 无 |

## 这是什么

把核心和插件拼成一个真的在跑的机器人：服务阴阳师 wiki（<https://yys.huijiwiki.com>），部署在一台内网服务器上。
这个仓库里只有三样东西：这个部署和默认值不一样的配置、用哪几个仓库的哪个提交、运维备忘。

## 在整套里的位置

JianShanAgent 由几个仓库拼成（并排放在开发机的同一个目录下）：

| 仓库 | 层 | 管什么 |
|---|---|---|
| [`agent-core`](https://github.com/JianShanAgent/agent-core) | 核心 | 机器人本体：QQ 网关、七个功能插件（查询、知识库、查证、对话、记忆、群聊记录、安全兜底）、管理后台、部署工具 |
| [`site-huiji`](https://github.com/JianShanAgent/site-huiji) | 站点插件 · 平台 | 灰机 wiki 平台在 MediaWiki 之上多出来的东西 |
| [`site-huiji-yys`](https://github.com/JianShanAgent/site-huiji-yys) | 站点插件 · 站 | 阴阳师 wiki 特有的东西（建在灰机平台上） |
| **[`instance-yys`](https://github.com/JianShanAgent/instance-yys)（这里）** | 实例 | 阴阳师 wiki 机器人这个部署：配置、用哪个版本、运维备忘 |
| [`template-site-platform`](https://github.com/JianShanAgent/template-site-platform) | 模板 · 站点插件 · 平台 | 接一个新的 wiki 平台时，从它开始 |
| [`template-site-wiki`](https://github.com/JianShanAgent/template-site-wiki) | 模板 · 站点插件 · 站 | 接一个新的 wiki 时，从它开始 |
| [`template-instance`](https://github.com/JianShanAgent/template-instance) | 模板 · 实例 | 新建一个部署时，从它开始 |

## 目录

```
config/
  core.wiki.yaml          服务哪个 wiki、用哪个站点插件（site: yys）
  secrets.example.yaml    凭据的模板；真的 secrets.yaml 不入库
  local/                  开发机的调试覆写，不入库
components.lock           这个部署用哪几个仓库的哪个提交
release                   发布入口（转给核心的 deploy/scripts/release.sh）；发布到哪台服务器也写在里面
pyproject.toml            开发环境：核心和插件都用旁边目录里的源码
docs/运维备忘.md           现在的状态、服务器、用到的外部服务、协作约定
data/                     运行数据（知识库文件等），不入库
```

## 安装和启用

开发机上几个仓库并排放：

```
JianShanAgent/
  agent-core/  site-huiji/  site-huiji-yys/  instance-yys/
```

发布（细节见核心的 [部署与运维](https://github.com/JianShanAgent/agent-core/blob/main/docs/部署与运维.md)）：

```bash
./release pin               # 把旁边各个仓库现在的提交记进 components.lock
git commit -am "…"          # 发布号 = 这个提交的短号
./release push              # 打包上传、构建镜像（不影响正在运行的服务）
./release activate <发布号>  # 切换；起不来自动切回
./release rollback          # 切回上一个
```

发布到哪台服务器写在 `release` 的 `SSH_HOST` 里（ssh 认得的名字）。服务器上：

| 是什么 | 在哪 |
|---|---|
| 发布、`.env`、凭据 | `/opt/jianshanagent/`（`releases/`、`shared/`、compose 入口 `ctl`） |
| 数据 | `/home/jianshanagent/`（postgres、backups、bot、redis，以及日志插件的 loki、grafana、alloy） |
| 管理后台 | 服务器内网地址的 13001 端口 |
| Grafana（部署插件 `logs`） | 服务器内网地址的 13000 端口，用户 `admin`，密码在服务器的 `shared/.env` |
| 带的部署插件 | `logs`（`shared/.env` 的 `JIANSHANAGENT_DEPLOY_PLUGINS`） |

## 配置

这里只写和默认值不一样的地方（默认值在核心和各插件的包里）。现在只有 `core.wiki.yaml`。
要改别的，新建和那份配置同名的文件，只写要改的项，例如 `config/plugin.chat.yaml`。
凭据：`cp config/secrets.example.yaml config/secrets.yaml` 后填写；传到服务器用 `./release secrets`。

## 开发和测试

```bash
uv sync                                        # 核心 + 两个插件，可编辑安装
JIANSHANAGENT_DRY_RUN=1 uv run jianshanagent     # 不连 QQ 试跑（同一个机器人不能两处同时在线）
uv run python -m jianshanagent.plugins.knowledge run --offline   # 用本地缓存重跑知识库预处理
```

各个仓库的测试在各自的目录里跑。

## 相关文档

- [运维备忘](docs/运维备忘.md)：现在的状态、服务器、用到的外部服务、对 wiki 的规矩、协作约定
- [插件规范](https://github.com/JianShanAgent/agent-core/blob/main/docs/插件规范.md)
- 核心的 [部署与运维](https://github.com/JianShanAgent/agent-core/blob/main/docs/部署与运维.md)
- 许可：[MIT](LICENSE)
