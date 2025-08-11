# OpenIM Server 故障排查指南

## 常见问题与解决方案

### 1. 服务启动问题

#### 问题：服务启动失败
**症状**：
- 服务进程无法启动
- 日志显示端口被占用
- 配置文件读取失败

**排查步骤**：
```bash
# 1. 检查端口占用
netstat -tlnp | grep 10002
lsof -i :10002

# 2. 检查配置文件
cat config/config.yaml
yaml-lint config/config.yaml

# 3. 检查日志
tail -f logs/api.log
journalctl -u openim-api -f

# 4. 检查进程状态
ps aux | grep openim
systemctl status openim-api
```

**解决方案**：
```bash
# 杀死占用端口的进程
kill -9 $(lsof -t -i:10002)

# 修复配置文件权限
chmod 644 config/config.yaml
chown openim:openim config/config.yaml

# 重启服务
systemctl restart openim-api
```

#### 问题：依赖服务连接失败
**症状**：
- MongoDB 连接超时
- Redis 连接被拒绝
- etcd 服务不可用

**排查步骤**：
```bash
# 检查 MongoDB 状态
systemctl status mongod
mongo --eval "db.runCommand('ping')"

# 检查 Redis 状态
systemctl status redis-server
redis-cli ping

# 检查 etcd 状态
systemctl status etcd
etcdctl endpoint health
```

**解决方案**：
```bash
# 启动依赖服务
systemctl start mongod redis-server etcd

# 检查防火墙设置
ufw status
iptables -L

# 修改配置文件中的连接地址
vim config/config.yaml
```

### 2. 数据库问题

#### 问题：MongoDB 连接池耗尽
**症状**：
- 大量连接超时错误
- 服务响应缓慢
- 日志显示连接池满

**排查步骤**：
```bash
# 查看 MongoDB 连接状态
mongo --eval "db.serverStatus().connections"

# 检查慢查询
mongo --eval "db.setProfilingLevel(2, {slowms: 100})"
mongo --eval "db.system.profile.find().sort({ts: -1}).limit(5)"

# 查看当前连接
mongo --eval "db.currentOp()"
```

**解决方案**：
```yaml
# 调整连接池配置
mongo:
  maxPoolSize: 200
  maxRetry: 5
  timeout: 30
```

```javascript
// 添加数据库索引
db.users.createIndex({"userID": 1})
db.messages.createIndex({"conversationID": 1, "sendTime": -1})
db.groups.createIndex({"groupID": 1})
```

#### 问题：Redis 内存不足
**症状**：
- Redis 写入失败
- 缓存命中率下降
- OOM 错误

**排查步骤**：
```bash
# 检查 Redis 内存使用
redis-cli info memory

# 查看键空间信息
redis-cli info keyspace

# 检查大键
redis-cli --bigkeys
```

**解决方案**：
```bash
# 设置内存限制和淘汰策略
redis-cli config set maxmemory 2gb
redis-cli config set maxmemory-policy allkeys-lru

# 清理过期键
redis-cli --scan --pattern "*" | xargs redis-cli del
```

### 3. 消息传输问题

#### 问题：消息发送失败
**症状**：
- 消息无法发送
- 消息丢失
- 发送超时

**排查步骤**：
```bash
# 检查消息网关状态
curl -X GET http://localhost:10001/health

# 查看 Kafka 状态
kafka-topics.sh --bootstrap-server localhost:9092 --list
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# 检查消息队列积压
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group msgToRedis
```

**解决方案**：
```bash
# 重启消息相关服务
systemctl restart openim-msggateway
systemctl restart openim-msgtransfer

# 清理 Kafka 积压消息
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --reset-offsets --to-latest --group msgToRedis --all-topics --execute
```

#### 问题：WebSocket 连接断开
**症状**：
- 客户端频繁重连
- 消息推送延迟
- 连接不稳定

**排查步骤**：
```bash
# 检查 WebSocket 连接数
netstat -an | grep :10001 | wc -l

# 查看连接状态
ss -tuln | grep :10001

# 检查负载均衡配置
nginx -t
systemctl status nginx
```

**解决方案**：
```nginx
# Nginx WebSocket 代理配置
upstream websocket {
    server 127.0.0.1:10001;
    server 127.0.0.1:10002;
}

server {
    location /ws {
        proxy_pass http://websocket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400;
    }
}
```

### 4. 性能问题

#### 问题：API 响应缓慢
**症状**：
- 接口响应时间长
- 并发处理能力差
- CPU 使用率高

**排查步骤**：
```bash
# 检查系统资源
top
htop
iostat -x 1

# 分析 API 性能
curl -w "@curl-format.txt" -o /dev/null -s "http://localhost:10002/api/v3/user/get_users_info"

# 使用 pprof 分析
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

**curl-format.txt**：
```
     time_namelookup:  %{time_namelookup}\n
        time_connect:  %{time_connect}\n
     time_appconnect:  %{time_appconnect}\n
    time_pretransfer:  %{time_pretransfer}\n
       time_redirect:  %{time_redirect}\n
  time_starttransfer:  %{time_starttransfer}\n
                     ----------\n
          time_total:  %{time_total}\n
```

**解决方案**：
```yaml
# 优化配置
api:
  timeout: 30
  maxConnections: 1000

# 启用连接池
mongo:
  maxPoolSize: 100
  minPoolSize: 10

redis:
  poolSize: 50
  minIdleConns: 10
```

#### 问题：内存泄漏
**症状**：
- 内存使用持续增长
- 系统变慢
- OOM Killer 触发

**排查步骤**：
```bash
# 监控内存使用
watch -n 1 'ps aux --sort=-%mem | head -10'

# 使用 pprof 分析内存
go tool pprof http://localhost:6060/debug/pprof/heap

# 检查 goroutine 泄漏
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

**解决方案**：
```go
// 正确关闭资源
defer func() {
    if conn != nil {
        conn.Close()
    }
}()

// 使用 context 控制超时
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
```

### 5. 网络问题

#### 问题：服务间通信失败
**症状**：
- RPC 调用超时
- 服务发现失败
- 负载均衡异常

**排查步骤**：
```bash
# 检查服务注册
etcdctl get --prefix /openim/

# 测试服务连通性
telnet 127.0.0.1 10110
nc -zv 127.0.0.1 10110

# 检查 DNS 解析
nslookup openim-api
dig openim-api
```

**解决方案**：
```bash
# 重新注册服务
systemctl restart openim-rpc-user

# 检查防火墙规则
ufw allow 10110/tcp
iptables -A INPUT -p tcp --dport 10110 -j ACCEPT

# 更新 hosts 文件
echo "127.0.0.1 openim-api" >> /etc/hosts
```

### 6. 认证授权问题

#### 问题：Token 验证失败
**症状**：
- 用户无法登录
- Token 过期错误
- 权限验证失败

**排查步骤**：
```bash
# 检查 Token 配置
grep -A 5 "tokenPolicy" config/config.yaml

# 验证 JWT Token
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." | base64 -d

# 检查时间同步
timedatectl status
ntpq -p
```

**解决方案**：
```yaml
# 调整 Token 配置
tokenPolicy:
  expire: 168  # 7天，单位小时

secret: "your-new-secret-key"
```

```bash
# 同步系统时间
ntpdate -s time.nist.gov
systemctl restart ntp
```

## 日志分析

### 1. 日志级别说明

| 级别 | 数值 | 说明 |
|------|------|------|
| PANIC | 0 | 系统崩溃 |
| FATAL | 1 | 致命错误 |
| ERROR | 2 | 错误信息 |
| WARN | 3 | 警告信息 |
| INFO | 4 | 一般信息 |
| DEBUG | 5 | 调试信息 |
| TRACE | 6 | 跟踪信息 |

### 2. 常见错误日志

#### 数据库连接错误
```
ERROR: failed to connect to MongoDB: connection timeout
ERROR: Redis connection refused
ERROR: etcd client connection failed
```

#### 消息处理错误
```
ERROR: failed to send message: timeout
ERROR: message validation failed
ERROR: conversation not found
```

#### 认证错误
```
ERROR: invalid token
ERROR: token expired
ERROR: permission denied
```

### 3. 日志分析脚本

```bash
#!/bin/bash
# log-analyzer.sh

LOG_FILE="logs/api.log"
TIME_RANGE="1h"

echo "=== 错误统计 ==="
grep -i error "$LOG_FILE" | tail -100 | awk '{print $4}' | sort | uniq -c | sort -nr

echo "=== 最近错误 ==="
grep -i error "$LOG_FILE" | tail -10

echo "=== 性能统计 ==="
grep "response_time" "$LOG_FILE" | awk '{print $6}' | awk -F'=' '{sum+=$2; count++} END {print "平均响应时间:", sum/count "ms"}'

echo "=== 访问统计 ==="
grep "GET\|POST" "$LOG_FILE" | awk '{print $7}' | sort | uniq -c | sort -nr | head -10
```

## 监控与告警

### 1. 系统监控指标

```bash
# CPU 使用率
top -bn1 | grep "Cpu(s)" | awk '{print $2}' | awk -F'%' '{print $1}'

# 内存使用率
free | grep Mem | awk '{printf "%.2f%%\n", $3/$2 * 100.0}'

# 磁盘使用率
df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{print $5 " " $1}' | while read output; do
  usage=$(echo $output | awk '{print $1}' | cut -d'%' -f1)
  partition=$(echo $output | awk '{print $2}')
  if [ $usage -ge 80 ]; then
    echo "警告: 磁盘分区 $partition 使用率达到 $usage%"
  fi
done
```

### 2. 应用监控指标

```bash
# 检查服务状态
check_service() {
    local service=$1
    local port=$2
    
    if nc -z localhost $port; then
        echo "$service: 运行正常"
    else
        echo "$service: 服务异常"
        systemctl status $service
    fi
}

check_service "openim-api" 10002
check_service "openim-rpc-user" 10110
check_service "openim-msggateway" 10001
```

### 3. 告警脚本

```bash
#!/bin/bash
# alert.sh

WEBHOOK_URL="https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"

send_alert() {
    local message=$1
    local level=$2
    
    curl -X POST -H 'Content-type: application/json' \
        --data "{\"text\":\"[$level] OpenIM Alert: $message\"}" \
        $WEBHOOK_URL
}

# 检查服务状态
if ! nc -z localhost 10002; then
    send_alert "API 服务不可用" "CRITICAL"
fi

# 检查磁盘空间
disk_usage=$(df / | tail -1 | awk '{print $5}' | cut -d'%' -f1)
if [ $disk_usage -gt 80 ]; then
    send_alert "磁盘使用率过高: $disk_usage%" "WARNING"
fi

# 检查内存使用
mem_usage=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100.0}')
if [ $mem_usage -gt 85 ]; then
    send_alert "内存使用率过高: $mem_usage%" "WARNING"
fi
```

## 性能调优

### 1. 系统级优化

```bash
# 调整文件描述符限制
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf

# 调整内核参数
cat >> /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_max_tw_buckets = 5000
EOF

sysctl -p
```

### 2. 数据库优化

```javascript
// MongoDB 索引优化
db.users.createIndex({"userID": 1}, {background: true})
db.messages.createIndex({"conversationID": 1, "sendTime": -1}, {background: true})
db.messages.createIndex({"sendTime": -1}, {background: true})
db.groups.createIndex({"groupID": 1}, {background: true})
db.group_members.createIndex({"groupID": 1, "userID": 1}, {background: true})

// 设置 TTL 索引自动清理过期数据
db.messages.createIndex({"createTime": 1}, {expireAfterSeconds: 7776000}) // 90天
```

```bash
# Redis 优化配置
redis-cli config set maxmemory-policy allkeys-lru
redis-cli config set timeout 300
redis-cli config set tcp-keepalive 60
```

### 3. 应用级优化

```yaml
# 连接池优化
mongo:
  maxPoolSize: 100
  minPoolSize: 10
  maxIdleTimeMS: 300000

redis:
  poolSize: 50
  minIdleConns: 10
  idleTimeout: 300

# 缓存优化
cache:
  msgCacheTimeout: 3600
  userCacheTimeout: 7200
  groupCacheTimeout: 3600
```

## 备份与恢复

### 1. 数据备份

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backup/openim"
DATE=$(date +%Y%m%d_%H%M%S)

# MongoDB 备份
mongodump --uri="mongodb://localhost:27017/openim_v3" --out="$BACKUP_DIR/mongo_$DATE"

# Redis 备份
redis-cli --rdb "$BACKUP_DIR/redis_$DATE.rdb"

# 配置文件备份
tar -czf "$BACKUP_DIR/config_$DATE.tar.gz" config/

# 清理旧备份（保留7天）
find $BACKUP_DIR -type f -mtime +7 -delete

echo "备份完成: $DATE"
```

### 2. 数据恢复

```bash
#!/bin/bash
# restore.sh

BACKUP_DIR="/backup/openim"
RESTORE_DATE=$1

if [ -z "$RESTORE_DATE" ]; then
    echo "用法: $0 <备份日期>"
    exit 1
fi

# 恢复 MongoDB
mongorestore --uri="mongodb://localhost:27017/openim_v3" --drop "$BACKUP_DIR/mongo_$RESTORE_DATE/openim_v3"

# 恢复 Redis
redis-cli flushall
redis-cli --rdb "$BACKUP_DIR/redis_$RESTORE_DATE.rdb"

# 恢复配置文件
tar -xzf "$BACKUP_DIR/config_$RESTORE_DATE.tar.gz" -C /

echo "恢复完成: $RESTORE_DATE"
```

这份故障排查指南涵盖了 OpenIM Server 运行过程中可能遇到的各种问题，提供了详细的排查步骤和解决方案，帮助运维人员快速定位和解决问题。