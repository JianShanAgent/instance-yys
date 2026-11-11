<p align="center"><img src="assets/icon.svg" width="88" alt="JianShanAgent"></p>

<h1 align="center">instance-yys</h1>

<p align="center"><a href="https://github.com/JianShanAgent/agent-core">JianShanAgent（见山）</a>服务于<a href="https://yys.huijiwiki.com">灰机阴阳师 Wiki</a> 的线上生产部署实例。</p>

<p align="center">
  <img src="https://img.shields.io/badge/部署实例-阴阳师%20Wiki-2f6b5a?style=flat-square&labelColor=1f3b34" alt="部署实例：阴阳师 Wiki">
  <img src="https://img.shields.io/github/license/JianShanAgent/instance-yys?label=License&color=2f6b5a&style=flat-square&labelColor=1f3b34" alt="License">
</p>

<sub>Production deployment repository of JianShanAgent serving the Onmyoji Wiki (`yys.huijiwiki.com`).</sub>

## 这个仓库是什么

见山采用“核心框架 + 站点插件 + 部署实例”分离的架构：
- [`agent-core`](https://github.com/JianShanAgent/agent-core)：通用 MediaWiki QQ 机器人框架、Web 管理后台与部署工具；
- [`site-huiji`](https://github.com/JianShanAgent/site-huiji)：灰机 Wiki 平台的 `Data:` 命名空间插件；
- [`site-huiji-yys`](https://github.com/JianShanAgent/site-huiji-yys)：阴阳师 Wiki 的式神/御魂查询、公告剧情知识库与立绘图库插件。

本仓库（`instance-yys`）负责**把上面三个仓库组装成线上实际运行的阴阳师 QQ 机器人**。仓库里不包含业务代码，只存放三样东西：
1. [`config/`](config)：阴阳师线上实例相对默认配置的差异项（目前只需一个 [`config/core.wiki.yaml`](config/core.wiki.yaml) 指明灰机接口地址并启用 `site: yys`）；
2. [`components.lock`](components.lock)：锁定线上当前使用的 `agent-core`、`site-huiji`、`site-huiji-yys` 各自精确的 Git Commit；
3. [`docs/运维备忘.md`](docs/运维备忘.md)：线上知识库规模、服务器目录约定与外部服务说明。

> **想给你自己的 Wiki 搭建机器人？**
> 请不要直接复制本仓库，去使用开箱即用的实例模板 **[`template-instance`](https://github.com/JianShanAgent/template-instance)**（支持 Python `uv` 原生直跑与 `docker compose up -d` 一键部署）；本仓库可作为配置写法的真实参考。

## 本地开发联调

开发机上将四个仓库并排放在同一个父目录下：

```text
JianShanAgent/
├── agent-core/
├── site-huiji/
├── site-huiji-yys/
└── instance-yys/
```

在本仓库目录下即可直接用源码可编辑模式启动调试：

```bash
# 首次运行复制凭据模板并填入密钥（secrets.yaml 不入库）
cp config/secrets.example.yaml config/secrets.yaml

# 安装同级目录下的核心与两个站点插件源码
uv sync

# 不连接 QQ 网关在本地试跑（打开 http://127.0.0.1:18080 即可在网页后台对话调试，令牌见 data/admin/token）
JIANSHANAGENT_DRY_RUN=1 uv run jianshanagent

# 使用本地已拉取的缓存离线重跑知识库清洗切片（调规则时使用）
uv run python -m jianshanagent.plugins.knowledge run --offline
```

> **注意**：线上服务器运行 `qqbot` 期间，开发机本地启动请务必加上 `JIANSHANAGENT_DRY_RUN=1`，避免同一个 QQ 机器人两处同时在线导致一条消息被重复回复。

## 生产服务器发布与回滚

服务器 SSH 主机名配置在 [`release`](release) 脚本的 `SSH_HOST` 中（完整运维手册见 [`agent-core` 部署与运维](https://github.com/JianShanAgent/agent-core/blob/main/docs/部署与运维.md)）：

```bash
# 1. 如果修改了旁边 agent-core / site-huiji / site-huiji-yys 的代码并已提交，先锁定最新 Commit：
./release pin
git commit -am "更新组件版本"

# （如果只修改了本仓库的 config/，提交后无需 pin）

# 2. 一键打包上传当前提交、构建镜像并切换上线（健康检查不过会自动切回上一版，无需手输发布号）：
./release deploy

# 3. 其他常用运维操作：
./release rollback    # 一键回退到上一个可用版本
./release secrets     # 更新服务器上的 shared/secrets.yaml 凭据
./release list        # 查看服务器上的历史版本与切换记录
```

## 修改线上配置

- **日常热更新参数**：直接登录 Web 管理后台在网页上修改即可，改动保存在服务器数据盘上，重新发版不会被覆盖。
- **随版本走的配置覆盖**：在 `config/` 下新建与默认配置同名的 YAML 文件，只写需要修改的字段（例如新建 `config/plugin.chat.yaml` 写 `entry_timeout_seconds: 45`），提交后执行 `./release deploy`。

全部关联仓库见 [JianShanAgent 组织主页](https://github.com/JianShanAgent)。

## 开源许可

[MIT License](LICENSE)
