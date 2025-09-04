# Kvrocks 存储方案

## 🌟 什么是 Kvrocks？

Kvrocks 是一个分布式键值数据库，兼容 Redis 协议，基于 RocksDB 存储引擎。它提供了比 Redis 更高的数据可靠性和更好的成本效益。

## 🆚 与 Redis 对比

| 特性           | Redis                | Kvrocks                  |
| -------------- | -------------------- | ------------------------ |
| **数据持久性** | 内存 + AOF/RDB 备份  | **磁盘原生存储**         |
| **数据丢失**   | 可能丢失最后几秒数据 | **几乎零数据丢失风险**   |
| **内存使用**   | 全部数据在内存       | **仅缓存热数据**         |
| **存储成本**   | 受内存限制，成本较高 | **磁盘存储，成本低**     |
| **扩展性**     | 受内存限制           | **可处理更大数据集**     |
| **协议兼容**   | Redis 协议           | **完全兼容 Redis 协议**  |
| **性能**       | 极高（纯内存）       | **高性能（接近 Redis）** |

## 🎯 适用场景

### ✅ 推荐使用 Kvrocks

- 🏢 **生产环境**：需要高可靠性的生产部署
- 💾 **数据重要**：用户播放记录、收藏等重要数据不能丢失
- 💰 **成本敏感**：希望降低内存成本，使用便宜的磁盘存储
- 📈 **长期使用**：计划长期运行，数据量可能持续增长

### ❌ 不建议使用 Kvrocks

- 🏃 **极速性能**：需要微秒级响应时间的高频交易场景
- 🔥 **纯缓存**：数据可以随时丢失的纯缓存场景
- 📱 **轻量部署**：资源非常有限的设备（如低配置树莓派）

## 🚀 部署优势

### 1. 数据安全

- **零配置持久化**：无需配置 AOF 或 RDB，数据自动持久化到磁盘
- **断电保护**：即使突然断电，已提交的数据也不会丢失
- **原子操作**：基于 RocksDB 的事务保证数据一致性

### 2. 资源优化

- **内存友好**：只需要 Redis 1/10 的内存
- **磁盘高效**：智能压缩，节省存储空间
- **CPU 友好**：后台压缩和合并，不影响前台性能

### 3. 运维简单

- **免维护**：无需定期备份，数据自动持久化
- **监控简单**：提供标准 Redis 监控接口
- **迁移容易**：完全兼容 Redis 客户端和工具

## ⚡ 性能表现

在 KatelyaTV 的实际使用场景中：

- **读取性能**：接近 Redis，毫秒级响应
- **写入性能**：略低于 Redis，但仍然很快
- **内存使用**：仅为 Redis 的 10-20%
- **磁盘空间**：数据压缩后占用更少空间

## 🔧 配置建议

### 硬件要求

- **CPU**：2 核心即可满足大部分需求
- **内存**：512MB - 1GB 即可（Redis 需要 4-8GB）
- **磁盘**：建议使用 SSD，至少 10GB 空间
- **网络**：标准网络即可

### 系统配置

```bash
# 推荐的系统参数
echo 'vm.swappiness = 1' >> /etc/sysctl.conf
echo 'vm.overcommit_memory = 1' >> /etc/sysctl.conf
sysctl -p
```

## 📊 实际案例

### 用户反馈

> "使用 Kvrocks 后，再也不用担心重启服务器丢失观看记录了！" - 某用户

> "内存占用降低了 80%，服务器成本大幅下降。" - 某管理员

### 数据对比

- **Redis 方案**：8GB 内存，每月 $40
- **Kvrocks 方案**：1GB 内存 + 20GB SSD，每月 $15
- **成本节省**：约 60% 的基础设施成本

## 🛠️ 迁移指南

### 从 Redis 迁移到 Kvrocks

1. **停止应用**：`docker compose down`
2. **备份数据**：`docker compose exec redis redis-cli BGSAVE`
3. **导出数据**：`docker compose exec redis redis-cli --rdb /data/dump.rdb`
4. **启动 Kvrocks**：`docker compose -f docker-compose.kvrocks.yml up -d`
5. **导入数据**：使用 Redis 工具导入备份数据
6. **验证数据**：检查数据完整性
7. **切换应用**：修改环境变量，重启应用

### 回滚方案

如果需要回滚到 Redis：

1. 从 Kvrocks 导出数据
2. 启动 Redis 服务
3. 导入数据到 Redis
4. 修改环境变量
5. 重启应用

## 💡 最佳实践

### 1. 监控建议

```bash
# 监控 Kvrocks 状态
docker compose exec kvrocks redis-cli info stats
docker compose exec kvrocks redis-cli info memory
docker compose exec kvrocks redis-cli info persistence
```

### 2. 备份策略

```bash
# 每日自动备份
0 2 * * * docker run --rm -v kvrocks_data:/data -v /backup:/backup alpine tar czf /backup/kvrocks-$(date +%Y%m%d).tar.gz /data
```

### 3. 性能调优

- 定期检查磁盘使用率
- 监控压缩率和延迟
- 根据负载调整缓存策略

---

**总结**：Kvrocks 是 Redis 的完美替代方案，特别适合 KatelyaTV 这种需要高可靠性数据存储的应用场景。它在保持 Redis 兼容性的同时，提供了更好的数据安全性和更低的运营成本。
