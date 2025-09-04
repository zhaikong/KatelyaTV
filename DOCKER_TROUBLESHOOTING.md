# Docker + Kvrocks 部署故障排除指南

## 🐛 常见问题及解决方案

### 问题一：`failed to read dockerfile: open Dockerfile: no such file or directory`

**症状**：

```bash
docker compose -f docker-compose.kvrocks.yml up -d
# 报错：failed to read dockerfile: open Dockerfile: no such file or directory
```

**原因**：

- 使用了旧版本的 `docker-compose.kvrocks.yml` 文件
- 旧版本使用 `build: .` 需要完整源代码和 Dockerfile
- 但部署文档只让下载配置文件，没有下载源代码

**解决方案**：

#### 方案一：使用预构建镜像（推荐）

```bash
# 重新下载最新版本的配置文件
curl -O https://raw.githubusercontent.com/katelya77/KatelyaTV/main/docker-compose.kvrocks.yml

# 启动服务
docker compose -f docker-compose.kvrocks.yml up -d
```

最新版本使用 `image: ghcr.io/katelya77/katelyatv:latest`，无需本地构建。

#### 方案二：本地构建

如果想要从源代码构建：

```bash
# 克隆完整源代码
git clone https://github.com/katelya77/KatelyaTV.git
cd KatelyaTV

# 使用本地构建版本
docker compose -f docker-compose.kvrocks.local.yml up -d
```

### 问题二：Kvrocks 连接失败

**症状**：

```bash
# 应用日志显示连接 Kvrocks 失败
Error: connect ECONNREFUSED
```

**解决方案**：

1. 检查 `.env` 文件中的 `KVROCKS_URL` 是否正确：

   ```bash
   KVROCKS_URL=redis://kvrocks:6666
   ```

2. 确保 Kvrocks 服务正常运行：

   ```bash
   docker compose -f docker-compose.kvrocks.yml ps
   ```

3. 测试 Kvrocks 连接：
   ```bash
   docker compose -f docker-compose.kvrocks.yml exec kvrocks redis-cli -h localhost -p 6666 ping
   ```

### 问题三：环境变量配置错误

**常见错误**：

- `NEXTAUTH_SECRET` 未设置
- `KVROCKS_PASSWORD` 不匹配

**解决方案**：
检查 `.env` 文件，确保所有必要变量都已正确配置：

```bash
# 必须配置的变量
NEXT_PUBLIC_STORAGE_TYPE=kvrocks
KVROCKS_URL=redis://kvrocks:6666
KVROCKS_PASSWORD=your_secure_password_here
NEXTAUTH_SECRET=your_nextauth_secret_here
NEXTAUTH_URL=http://localhost:3000
```

## 🔧 调试命令

```bash
# 查看服务状态
docker compose -f docker-compose.kvrocks.yml ps

# 查看应用日志
docker compose -f docker-compose.kvrocks.yml logs -f katelyatv

# 查看 Kvrocks 日志
docker compose -f docker-compose.kvrocks.yml logs -f kvrocks

# 进入容器调试
docker compose -f docker-compose.kvrocks.yml exec katelyatv sh

# 重建服务
docker compose -f docker-compose.kvrocks.yml up -d --force-recreate
```

## 📞 获取帮助

如果以上方案都无法解决问题，请：

1. 提供完整的错误日志
2. 说明使用的配置文件版本
3. 提供系统环境信息（操作系统、Docker 版本等）

---

**文档版本**：v0.6.0-katelya  
**更新日期**：2025 年 9 月 3 日
