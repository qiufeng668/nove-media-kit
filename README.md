# Nove Media Kit

一个开箱即用的媒体服务端部署包，基于开源项目构建，提供 Docker 镜像与运行说明。

> 面向个人 NAS / 小服务器场景：直接读取已有媒体库（NFO、海报、演员头像原样复用），
> 无需重新刮削，Infuse、SenPlayer、VidHub 等 Emby 兼容客户端可直接连接。

---

## 特性

- **Emby 兼容**：客户端像连 Emby 一样连接，无需改造
- **只读模式**：不写媒体目录，不破坏原有元数据
- **直链播放**：`.strm` 走 302 跳转，服务器不中转视频流量
- **多平台**：amd64 / arm64 双架构镜像
- **轻量**：1 核 1G 即可运行，常驻内存约 90MB
- **内置 ffmpeg**：无需额外安装

---

## 快速开始

### 方式一：Docker Compose（推荐）

```yaml
services:
  nove:
    image: qiufeng668/nove-media-kit:latest
    container_name: nove
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Asia/Shanghai
      - NOVA_SERVER_ADDR=:8098
    volumes:
      - ./config:/config      # 数据库、日志、插件、封面，必须持久化
      - ./media:/media        # 媒体目录，可加 :ro 只读
      # - /CloudNAS:/CloudNAS:rslave   # CD2 挂载时用
      # - /var/lib/emby/metadata/people:/emby-people:ro  # 复用 Emby 演员头像
```

启动：

```bash
docker compose up -d
```

### 方式二：Docker Run

```bash
mkdir -p ./config ./media

docker run -d \
  --name nove \
  --restart unless-stopped \
  --network host \
  -e TZ=Asia/Shanghai \
  -e NOVA_SERVER_ADDR=:8098 \
  -v "$PWD/config:/config" \
  -v "$PWD/media:/media" \
  qiufeng668/nove-media-kit:latest
```

### 首次访问

浏览器打开 `http://服务器IP:8098`，使用日志中的 6 位初始化码创建管理员：

```bash
docker logs nove 2>&1 | grep -i "setup_code"
```

---

## 配置说明

| 环境变量 | 说明 | 默认 |
|---|---|---|
| `NOVA_SERVER_ADDR` | 监听地址 | `:8096` |
| `NOVA_PATHS_CONFIG` | 配置目录 | `/config` |
| `NOVA_MEMORY_LIMIT_MB` | 内存软上限（MB），不设则自动取容器限制的 70% | 自动 |
| `NOVA_LOG_LEVEL` | 日志级别：`info` / `debug` / `trace` | `info` |
| `NOVA_DB_CACHE_SIZE_MB` | SQLite 缓存，大库建议 32 以上 | SQLite 默认 |
| `TZ` | 时区 | `UTC` |

### 目录结构

```
/config
├── novaemby.db      数据库（用户、媒体库、播放记录）
├── logs/            日志文件
├── plugins/         插件
├── covers/          生成的封面
├── fonts/           字体
└── backups/         备份
```

**`/config` 一定要持久化**，否则重建容器会丢配置。

---

## 常用命令

```bash
# 查看日志
docker logs -f --tail 100 nove

# 重启（网页里也能重启，靠 restart policy 拉起）
docker restart nove

# 升级镜像
docker compose pull && docker compose up -d

# 备份
tar czf nove-backup-$(date +%F).tar.gz ./config
```

---

## 资源占用参考

| 场景 | 内存 | 说明 |
|---|---|---|
| 空载（无扫描） | ~90 MB | 最小占用 |
| 常规使用 | 150–300 MB | 有媒体库、少量播放 |
| 扫描中 | +100–200 MB | 扫描完成后回落 |

1 核 1G 的机器足够个人使用。**不建议在 1G 机器上同时跑扫描和转码。**

---

## 故障排查

**容器起不来**
```bash
docker logs nove
```
常见原因：`/config` 无写权限、端口被占用（同机有 Emby 时换 8098）。

**网页打不开**
```bash
curl http://127.0.0.1:8098/health
```
返回 `{"status":"ok"}` 说明服务正常，检查防火墙 / 安全组。

**内存被吃光**
设置 `NOVA_MEMORY_LIMIT_MB=600`，并避免同时开启多个扫描任务。

**忘记管理员密码**
```bash
docker exec -it nove novaemby --help
```
或在网页登录页使用「密码重置」流程。

---

## 镜像标签

| 标签 | 说明 |
|---|---|
| `latest` | 最新版 |
| `v2` | 固定版本 |

固定版本更稳妥，避免意外升级。

---

## 系统要求

- Linux（amd64 / arm64）
- Docker 20.10+
- 磁盘：镜像约 620 MB，配置目录视媒体库大小而定

---

## 授权说明

本仓库仅提供部署脚本与镜像分发，镜像基于上游项目构建。
使用前请自行确认符合上游项目的许可条款。

---

## 免责声明

本项目仅供个人学习和自建媒体服务器使用。
请勿用于商业分发或任何违反当地法律法规的用途。
使用者需自行承担因使用本软件产生的全部责任。
