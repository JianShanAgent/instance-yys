<p align="center"><img src="assets/icon.svg" width="88" alt="JianShanAgent"></p>

<h1 align="center">instance-yys</h1>

<p align="center"><a href="https://github.com/JianShanAgent/agent-core">见山</a>在<a href="https://yys.huijiwiki.com">阴阳师 wiki</a> 的那一个部署。</p>

<p align="center">
  <img src="https://img.shields.io/badge/实例-阴阳师%20wiki-2f6b5a?style=flat-square&labelColor=1f3b34" alt="实例：阴阳师 wiki">
  <img src="https://img.shields.io/github/license/JianShanAgent/instance-yys?label=License&color=2f6b5a&style=flat-square&labelColor=1f3b34" alt="License">
</p>

<sub>A JianShanAgent deployment: the QQ bot serving the Onmyoji wiki. Documentation is in Chinese.</sub>

## 这个仓库是干什么的

见山的代码分散在几个仓库里：[核心](https://github.com/JianShanAgent/agent-core)、[灰机平台的插件](https://github.com/JianShanAgent/site-huiji)、[阴阳师 wiki 的插件](https://github.com/JianShanAgent/site-huiji-yys)。它们都不知道自己要部署到哪、服务哪个 wiki。把它们拼成一个真的在 QQ 上回话的机器人，是这个仓库的事。

所以这里几乎没有代码，只有三样东西：

| | |
|---|---|
| [`config/`](config) | 这个部署和默认值不一样的地方。现在只有一个文件：服务哪个 wiki、用哪个站点插件 |
| [`components.lock`](components.lock) | 用哪几个仓库的哪个提交。线上跑的是什么，以它为准 |
| [`docs/运维备忘.md`](docs/运维备忘.md) | 这个部署现在是什么样、用了哪些外面的服务、对 wiki 要守的规矩 |

想给你自己的 wiki 部署一个，不用复制这个仓库，用模板 [`template-instance`](https://github.com/JianShanAgent/template-instance)；这里可以当一个填好了的例子看。

## 怎么发布

开发机上几个仓库并排放：

```text
JianShanAgent/
  agent-core/  site-huiji/  site-huiji-yys/  instance-yys/
```

改完代码、各自提交之后，在这个目录里：

```bash
./release pin               # 把旁边各个仓库现在的提交记进 components.lock
git commit -am "…"          # 发布号就是这个提交的短号
./release push              # 打包上传、构建镜像，不影响正在跑的服务
./release activate <发布号>  # 切换；起不来会自动切回上一个
./release rollback          # 反悔
```

只改了 `config/` 时不用 `pin`，提交了直接 `push`。发布到哪台服务器写在 [`release`](release) 的 `SSH_HOST` 里。回滚、备份恢复、知识库怎么更新，见核心的[部署与运维](https://github.com/JianShanAgent/agent-core/blob/main/docs/部署与运维.md)。

## 改配置

这里只写和默认值不一样的地方——默认值在核心和各个插件的包里。要改哪份配置，就在 `config/` 下建一个和它同名的文件，只写要改的那几项：

```yaml
# config/plugin.chat.yaml
entry_timeout_seconds: 45
```

凭据不入库：`cp config/secrets.example.yaml config/secrets.yaml` 后填写，用 `./release secrets` 传到服务器。

## 在本机试

```bash
uv sync                                        # 核心和两个插件，用旁边目录里的源码
JIANSHANAGENT_DRY_RUN=1 uv run jianshanagent     # 不连 QQ
uv run python -m jianshanagent.plugins.knowledge run --offline   # 用本地缓存重跑知识库预处理
```

同一个机器人不能两处同时在线，不然一条消息会被回复两次——线上的在跑时，本机一定要带 `JIANSHANAGENT_DRY_RUN=1`。

全部仓库的目录在[组织主页](https://github.com/JianShanAgent)。

## 许可

[MIT](LICENSE)
