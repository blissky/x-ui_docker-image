# x-ui Docker Image

本仓库用于自动构建并发布 [MHSanaei/3x-ui](https://github.com/MHSanaei/3x-ui) 项目的 Docker 镜像，发布地址为：

```text
ghcr.io/blissky/x-ui
```

镜像由 GitHub Actions 使用上游 `3x-ui` 仓库正式 Release 对应 tag 中的 Dockerfile 直接构建，不在本仓库复制上游源码。推送到本仓库 `main`、推送 Git tag、定时检查或手动运行 workflow 时会检查上游最新正式版；只有对应版本镜像尚未发布时才会执行构建。

## 快速部署

确保已安装 Docker Engine 和 Docker Compose v2，然后在本仓库目录执行：

```bash
docker compose pull
docker compose up -d
```

面板默认通过以下地址访问：

```text
http://<服务器地址>:2053
```

Compose 使用宿主机网络模式（`network_mode: host`）。因此容器不会进行端口映射，面板和 Xray 入站会直接监听宿主机端口。需要在 3x-ui 面板中配置的每个入站端口都必须确保未被宿主机其他程序占用，并在云服务器安全组及防火墙中放行。

如果 `x-ui` package 尚未设置为公开，先登录 GitHub Container Registry：

```bash
docker login ghcr.io
```

## 持久化目录

Compose 会将以下目录挂载到容器中，建议在升级或重建容器时保留这些目录：

| 主机目录 | 容器目录 | 用途 |
| --- | --- | --- |
| `./db` | `/etc/x-ui/` | SQLite 数据库及面板配置 |
| `./cert` | `/root/cert/` | TLS 证书 |
| `./acme` | `/root/.acme.sh/` | acme.sh 证书续期状态 |

首次部署前可以手动创建目录：

```bash
mkdir -p db cert acme
```

## 配置说明

当前 Compose 配置默认使用 SQLite，面板直接监听宿主机的以下端口：

```text
2053
```

镜像默认启用 Fail2ban，因此 Compose 为容器添加了 `NET_ADMIN` 和 `NET_RAW` capability。由于容器使用宿主机网络模式，Fail2ban 可能直接修改宿主机的 iptables 规则。若不需要 Fail2ban，可在 `docker-compose.yml` 中将 `XUI_ENABLE_FAIL2BAN` 设置为 `"false"`，并按需移除对应 capability。

`network_mode: host` 主要适用于 Linux Docker 主机。在 Docker Desktop（Windows/macOS）上，宿主机网络模式的行为与原生 Linux 不同，不建议依赖该模式暴露服务。

PostgreSQL 服务默认以注释形式保留在 `docker-compose.yml` 中，不会被加载或启动。如需使用 PostgreSQL，请同时取消以下内容的注释：

- `x-ui` 服务中的 `XUI_DB_TYPE` 和 `XUI_DB_DSN`；
- 文件末尾的 `postgres` 服务。

然后执行：

```bash
docker compose --profile postgres up -d
```

## 镜像标签

- `latest`：指向最新上游正式版；
- 上游版本 tag：例如 `v3.7.0`；
- 不带 `v` 的版本号：例如 `3.7.0`。

workflow 每 6 小时检查一次上游 `releases/latest`。生产环境建议使用具体版本标签，例如 `v3.7.0`，而不是长期依赖 `latest`。

## 上游项目与许可证

本仓库的镜像内容来自 [MHSanaei/3x-ui](https://github.com/MHSanaei/3x-ui)，上游项目遵循 [GNU General Public License v3.0](https://github.com/MHSanaei/3x-ui/blob/main/LICENSE)。本仓库新增的 workflow、Compose 配置和文档同样以 GPL-3.0 发布，完整许可证文本见 [LICENSE](./LICENSE)。

如需了解 3x-ui 的完整功能、环境变量和使用限制，请以上游项目的[官方文档](https://github.com/MHSanaei/3x-ui)为准。
