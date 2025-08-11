# OpenIM Server 配置指南

## 配置文件概述

OpenIM Server 使用 YAML 格式的配置文件，主要包括以下几个配置文件：

- `config.yaml` - 主配置文件
- `notification.yaml` - 通知配置文件
- `webhooks.yaml` - Webhook 配置文件

## 主配置文件 (config.yaml)

### 完整配置示例

```yaml
# 环境配置
env: production  # development, testing, production

# 服务发现配置
discovery:
  enable: etcd
  etcd:
    address: [127.0.0.1:2379]
    username: ""
    password: ""
    rootDirectory: openim

# API 服务配置
api:
  openImApiPort: [10002]
  listenIP: 0.0.0.0

# RPC 服务配置
rpcPort:
  openImUserPort: [10110]
  openImFriendPort: [10120]
  openImMessagePort: [10130]
  openImGroupPort: [10150]
  openImAuthPort: [10160]
  openImPushPort: [10170]
  openImConversationPort: [10180]
  openImThirdPort: [10190]

# RPC 注册名称
rpcRegisterName:
  openImUserName: User
  openImFriendName: Friend
  openImMsgName: Msg
  openImPushName: Push
  openImMessageGatewayName: MessageGateway
  openImGroupName: Group
  openImAuthName: Auth
  openImConversationName: Conversation
  openImThirdName: Third

# 数据库配置
mongo:
  uri: mongodb://127.0.0.1:27017/openim_v3?maxPoolSize=100
  address: [127.0.0.1:27017]
  database: openim_v3
  username: ""
  password: ""
  maxPoolSize: 100
  maxRetry: 5

# Redis 配置
redis:
  address: [127.0.0.1:6379]
  username: ""
  password: ""
  db: 0
  specialSeparator: ":"
  clusterMode: false

# Kafka 配置
kafka:
  username: ""
  password: ""
  addr: [127.0.0.1:9092]
  latestMsgToRedis:
    topic: "latestMsgToRedis"
  offlineMsgToMongo:
    topic: "offlineMsgToMongo"
  msgToPush:
    topic: "msgToPush"
  consumerGroupID:
    msgToRedis: redis
    msgToMongo: mongo
    msgToMySql: mysql
    msgToPush: push

# 日志配置
log:
  storageLocation: ../logs/
  rotationTime: 24
  remainRotationCount: 2
  remainLogLevel: 6
  isStdout: false
  isJson: false
  withStack: false

# 密钥配置
secret: openIM123

# Token 配置
tokenPolicy:
  expire: 90

# 消息验证配置
messageVerify:
  friendVerify: false

# iOS 推送配置
iosPush:
  pushSound: "xxx"
  badgeCount: true
  production: false

# 回调配置
callback:
  url: http://127.0.0.1:8080/callback
  beforeSendSingleMsg:
    enable: false
    timeout: 5
    failedContinue: true
  afterSendSingleMsg:
    enable: false
    timeout: 5
  beforeSendGroupMsg:
    enable: false
    timeout: 5
    failedContinue: true
  afterSendGroupMsg:
    enable: false
    timeout: 5
  msgModify:
    enable: false
    timeout: 5
    failedContinue: true
  userOnline:
    enable: false
    timeout: 5
  userOffline:
    enable: false
    timeout: 5
  userKickOff:
    enable: false
    timeout: 5
  offlinePush:
    enable: false
    timeout: 5
    failedContinue: true
  onlinePush:
    enable: false
    timeout: 5
    failedContinue: true
  superGroupOnlinePush:
    enable: false
    timeout: 5
    failedContinue: true
  beforeAddFriend:
    enable: false
    timeout: 5
    failedContinue: true
  beforeCreateGroup:
    enable: false
    timeout: 5
    failedContinue: true
  beforeMemberJoinGroup:
    enable: false
    timeout: 5
    failedContinue: true
  beforeSetGroupMemberInfo:
    enable: false
    timeout: 5
    failedContinue: true

# 对象存储配置
object:
  enable: minio
  minio:
    bucket: openim
    endpoint: http://127.0.0.1:9000
    accessKeyID: root
    secretAccessKey: openIM123
    sessionToken: ""
    signEndpoint: http://127.0.0.1:10005
    publicRead: false
  cos:
    bucketURL: https://temp-1252357374.cos.ap-chengdu.myqcloud.com
    secretID: ""
    secretKey: ""
    sessionToken: ""
    publicRead: true
  oss:
    endpoint: https://oss-cn-chengdu.aliyuncs.com
    bucket: ""
    bucketURL: https://openim-1306374445.oss-cn-chengdu.aliyuncs.com
    accessKeyID: ""
    accessKeySecret: ""
    sessionToken: ""
    publicRead: true
  kodo:
    endpoint: http://s3.cn-east-1.qiniucs.com
    bucket: ""
    bucketURL: ""
    accessKeyID: ""
    secretAccessKey: ""
    sessionToken: ""
    publicRead: false
  aws:
    endpoint: ""
    region: us-east-1
    bucket: ""
    accessKeyID: ""
    secretAccessKey: ""
    sessionToken: ""
    publicRead: false

# 推送配置
push:
  enable: getui
  geTui:
    pushUrl: "https://restapi.getui.com/v2/$appId"
    masterSecret: ""
    appKey: ""
    intent: ""
    channelID: ""
    channelName: ""
  fcm:
    serviceAccount: "x.json"
  jpns:
    appKey: ""
    masterSecret: ""
    pushUrl: ""
    pushIntent: ""

# 验证码配置
verificationCode:
  store: redis
  superCode: "666666"
  mail:
    title: ""
    senderMail: ""
    senderAuthorizationCode: ""
    smtpAddr: ""
    smtpPort: 587
  ali:
    endpoint: ""
    accessKeyId: ""
    accessKeySecret: ""
    signName: ""
    verificationCodeTemplateCode: ""
  tencent:
    endpoint: "sms.tencentcloudapi.com"
    region: "ap-nanjing"
    accessKeyId: ""
    accessKeySecret: ""
    appId: ""
    signName: ""
    verificationCodeTemplateId: ""
  testDepartMentID: 001

# 多端登录策略
multiLoginPolicy: 1

# 聊天记录清理配置
chatRecordsClearTime: 180

# 消息缓存超时配置
msgCacheTimeout: 86400

# 群成员缓存超时配置
groupMemberCacheTimeout: 86400

# 好友缓存超时配置
friendCacheTimeout: 86400

# 对话缓存超时配置
conversationCacheTimeout: 86400

# 批量消息拉取数量
batchMsgCount: 100
```

### 配置项详解

#### 1. 环境配置
```yaml
env: production  # 环境类型
# development: 开发环境
# testing: 测试环境  
# production: 生产环境
```

#### 2. 服务发现配置
```yaml
discovery:
  enable: etcd           # 服务发现类型
  etcd:
    address: [127.0.0.1:2379]  # etcd 地址列表
    username: ""               # etcd 用户名
    password: ""               # etcd 密码
    rootDirectory: openim      # 根目录
```

#### 3. 端口配置
```yaml
api:
  openImApiPort: [10002]  # API 服务端口
  listenIP: 0.0.0.0       # 监听 IP

rpcPort:
  openImUserPort: [10110]        # 用户服务端口
  openImFriendPort: [10120]      # 好友服务端口
  openImMessagePort: [10130]     # 消息服务端口
  openImGroupPort: [10150]       # 群组服务端口
  openImAuthPort: [10160]        # 认证服务端口
  openImPushPort: [10170]        # 推送服务端口
  openImConversationPort: [10180] # 会话服务端口
  openImThirdPort: [10190]       # 第三方服务端口
```

#### 4. 数据库配置
```yaml
mongo:
  uri: mongodb://127.0.0.1:27017/openim_v3?maxPoolSize=100
  address: [127.0.0.1:27017]  # MongoDB 地址
  database: openim_v3         # 数据库名称
  username: ""                # 用户名
  password: ""                # 密码
  maxPoolSize: 100           # 最大连接池大小
  maxRetry: 5                # 最大重试次数
```

#### 5. Redis 配置
```yaml
redis:
  address: [127.0.0.1:6379]  # Redis 地址
  username: ""               # Redis 用户名
  password: ""               # Redis 密码
  db: 0                      # 数据库编号
  specialSeparator: ":"      # 特殊分隔符
  clusterMode: false         # 是否集群模式
```

#### 6. Kafka 配置
```yaml
kafka:
  username: ""               # Kafka 用户名
  password: ""               # Kafka 密码
  addr: [127.0.0.1:9092]    # Kafka 地址
  latestMsgToRedis:
    topic: "latestMsgToRedis"  # 最新消息到 Redis 主题
  offlineMsgToMongo:
    topic: "offlineMsgToMongo" # 离线消息到 MongoDB 主题
  msgToPush:
    topic: "msgToPush"         # 消息推送主题
  consumerGroupID:
    msgToRedis: redis          # 消费者组 ID
    msgToMongo: mongo
    msgToMySql: mysql
    msgToPush: push
```

#### 7. 日志配置
```yaml
log:
  storageLocation: ../logs/     # 日志存储位置
  rotationTime: 24             # 日志轮转时间(小时)
  remainRotationCount: 2       # 保留轮转文件数量
  remainLogLevel: 6            # 日志级别
  isStdout: false             # 是否输出到标准输出
  isJson: false               # 是否 JSON 格式
  withStack: false            # 是否包含堆栈信息
```

#### 8. 对象存储配置
```yaml
object:
  enable: minio              # 启用的存储类型
  minio:
    bucket: openim           # 存储桶名称
    endpoint: http://127.0.0.1:9000  # MinIO 端点
    accessKeyID: root        # 访问密钥 ID
    secretAccessKey: openIM123  # 访问密钥
    sessionToken: ""         # 会话令牌
    signEndpoint: http://127.0.0.1:10005  # 签名端点
    publicRead: false        # 是否公开读取
```

## 通知配置文件 (notification.yaml)

### 完整配置示例

```yaml
# 群组通知配置
groupCreated:
  conversation:
    reliabilityLevel: 1    # 可靠性级别
    unreadCount: true      # 是否计入未读数
  offlinePush:
    enable: true           # 是否启用离线推送
    title: "群组创建通知"
    desc: "您被邀请加入群组"
    ext: ""

# 好友申请通知
friendApplicationApproved:
  conversation:
    reliabilityLevel: 1
    unreadCount: true
  offlinePush:
    enable: true
    title: "好友申请"
    desc: "您的好友申请已通过"
    ext: ""

# 消息撤回通知
msgRevokeNotification:
  conversation:
    reliabilityLevel: 1
    unreadCount: false
  offlinePush:
    enable: false
    title: ""
    desc: ""
    ext: ""

# 群成员踢出通知
groupMemberKicked:
  conversation:
    reliabilityLevel: 1
    unreadCount: true
  offlinePush:
    enable: true
    title: "群组通知"
    desc: "您已被移出群组"
    ext: ""

# 群成员邀请通知
groupMemberInvited:
  conversation:
    reliabilityLevel: 1
    unreadCount: true
  offlinePush:
    enable: true
    title: "群组邀请"
    desc: "您被邀请加入群组"
    ext: ""

# 群成员进入通知
groupMemberEnter:
  conversation:
    reliabilityLevel: 1
    unreadCount: true
  offlinePush:
    enable: false
    title: ""
    desc: ""
    ext: ""

# 群解散通知
groupDismissed:
  conversation:
    reliabilityLevel: 1
    unreadCount: true
  offlinePush:
    enable: true
    title: "群组通知"
    desc: "群组已解散"
    ext: ""

# 群信息变更通知
groupInfoSet:
  conversation:
    reliabilityLevel: 1
    unreadCount: true
  offlinePush:
    enable: false
    title: ""
    desc: ""
    ext: ""
```

## Webhook 配置文件 (webhooks.yaml)

### 完整配置示例

```yaml
# Webhook 全局配置
url: "http://127.0.0.1:8080/webhook"  # Webhook 回调地址
timeout: 5                            # 超时时间(秒)
failedContinue: true                  # 失败后是否继续

# 消息发送前回调
beforeSendSingleMsg:
  enable: false                       # 是否启用
  timeout: 5                         # 超时时间
  failedContinue: true               # 失败后是否继续

# 消息发送后回调
afterSendSingleMsg:
  enable: false
  timeout: 5

# 群消息发送前回调
beforeSendGroupMsg:
  enable: false
  timeout: 5
  failedContinue: true

# 群消息发送后回调
afterSendGroupMsg:
  enable: false
  timeout: 5

# 消息修改回调
msgModify:
  enable: false
  timeout: 5
  failedContinue: true

# 用户上线回调
userOnline:
  enable: false
  timeout: 5

# 用户下线回调
userOffline:
  enable: false
  timeout: 5

# 用户被踢下线回调
userKickOff:
  enable: false
  timeout: 5

# 离线推送回调
offlinePush:
  enable: false
  timeout: 5
  failedContinue: true

# 在线推送回调
onlinePush:
  enable: false
  timeout: 5
  failedContinue: true

# 添加好友前回调
beforeAddFriend:
  enable: false
  timeout: 5
  failedContinue: true

# 创建群组前回调
beforeCreateGroup:
  enable: false
  timeout: 5
  failedContinue: true

# 成员加入群组前回调
beforeMemberJoinGroup:
  enable: false
  timeout: 5
  failedContinue: true
```

## 环境变量配置

### 常用环境变量

```bash
# 基础配置
export OPENIM_IP=127.0.0.1
export OPENIM_SECRET=openIM123
export OPENIM_DATA_DIR=/data/openim

# 数据库配置
export MONGO_URI=mongodb://127.0.0.1:27017
export MONGO_DATABASE=openim_v3
export MONGO_USERNAME=""
export MONGO_PASSWORD=""

# Redis 配置
export REDIS_ADDRESS=127.0.0.1:6379
export REDIS_PASSWORD=""
export REDIS_DB=0

# etcd 配置
export ETCD_ADDRESS=127.0.0.1:2379
export ETCD_USERNAME=""
export ETCD_PASSWORD=""

# Kafka 配置
export KAFKA_ADDRESS=127.0.0.1:9092
export KAFKA_USERNAME=""
export KAFKA_PASSWORD=""

# 对象存储配置
export MINIO_ENDPOINT=http://127.0.0.1:9000
export MINIO_ACCESS_KEY=root
export MINIO_SECRET_KEY=openIM123
export MINIO_BUCKET=openim

# 推送配置
export GETUI_APP_KEY=""
export GETUI_MASTER_SECRET=""
export FCM_SERVICE_ACCOUNT_PATH=""

# 日志配置
export LOG_LEVEL=info
export LOG_FORMAT=json
export LOG_OUTPUT=file
```

## 配置最佳实践

### 1. 安全配置

#### 密钥管理
```yaml
# 使用强密钥
secret: "your-very-strong-secret-key-here"

# Token 过期时间设置
tokenPolicy:
  expire: 7  # 7天过期，生产环境建议更短

# 数据库密码
mongo:
  username: "openim_user"
  password: "strong_password_here"

redis:
  password: "redis_strong_password"
```

#### 网络安全
```yaml
# 限制监听 IP
api:
  listenIP: 127.0.0.1  # 仅本地访问

# 使用 TLS
mongo:
  uri: mongodb://user:pass@host:27017/db?ssl=true

redis:
  tls: true
```

### 2. 性能优化配置

#### 数据库连接池
```yaml
mongo:
  maxPoolSize: 100      # 根据并发量调整
  maxRetry: 3           # 重试次数

redis:
  poolSize: 50          # Redis 连接池大小
  minIdleConns: 10      # 最小空闲连接
```

#### 缓存配置
```yaml
# 缓存超时时间优化
msgCacheTimeout: 3600           # 1小时
groupMemberCacheTimeout: 7200   # 2小时
friendCacheTimeout: 86400       # 24小时
conversationCacheTimeout: 3600  # 1小时
```

#### 消息处理优化
```yaml
# 批量处理配置
batchMsgCount: 50              # 批量消息数量
chatRecordsClearTime: 90       # 聊天记录清理时间(天)

# Kafka 分区配置
kafka:
  partitions: 3                # 分区数量
  replicationFactor: 2         # 副本因子
```

### 3. 高可用配置

#### 集群配置
```yaml
# MongoDB 副本集
mongo:
  uri: mongodb://host1:27017,host2:27017,host3:27017/openim_v3?replicaSet=rs0

# Redis 集群
redis:
  clusterMode: true
  address: 
    - 127.0.0.1:7000
    - 127.0.0.1:7001
    - 127.0.0.1:7002

# etcd 集群
discovery:
  etcd:
    address: 
      - 127.0.0.1:2379
      - 127.0.0.1:2380
      - 127.0.0.1:2381
```

#### 负载均衡配置
```yaml
# 多端口配置
api:
  openImApiPort: [10002, 10003, 10004]

rpcPort:
  openImUserPort: [10110, 10111, 10112]
  openImFriendPort: [10120, 10121, 10122]
```

### 4. 监控配置

#### 日志配置
```yaml
log:
  storageLocation: /var/log/openim/
  rotationTime: 24              # 24小时轮转
  remainRotationCount: 7        # 保留7天日志
  remainLogLevel: 4             # INFO 级别
  isStdout: true               # 同时输出到控制台
  isJson: true                 # JSON 格式便于解析
  withStack: true              # 包含堆栈信息
```

#### 指标收集
```yaml
# Prometheus 指标
prometheus:
  enable: true
  port: 9090
  path: /metrics

# 健康检查
healthCheck:
  enable: true
  port: 8080
  path: /health
```

## 配置验证

### 1. 配置文件验证脚本

```bash
#!/bin/bash
# validate-config.sh

echo "验证配置文件..."

# 检查配置文件是否存在
if [ ! -f "config/config.yaml" ]; then
    echo "错误: config.yaml 文件不存在"
    exit 1
fi

# 验证 YAML 格式
python3 -c "import yaml; yaml.safe_load(open('config/config.yaml'))" 2>/dev/null
if [ $? -ne 0 ]; then
    echo "错误: config.yaml 格式不正确"
    exit 1
fi

# 检查必要的配置项
required_configs=(
    "mongo.uri"
    "redis.address"
    "secret"
)

for config in "${required_configs[@]}"; do
    value=$(yq eval ".$config" config/config.yaml)
    if [ "$value" = "null" ] || [ -z "$value" ]; then
        echo "错误: 缺少必要配置项 $config"
        exit 1
    fi
done

echo "配置文件验证通过"
```

### 2. 连接测试脚本

```bash
#!/bin/bash
# test-connections.sh

echo "测试数据库连接..."

# 测试 MongoDB 连接
mongo_uri=$(yq eval '.mongo.uri' config/config.yaml)
mongosh "$mongo_uri" --eval "db.runCommand('ping')" >/dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "✓ MongoDB 连接正常"
else
    echo "✗ MongoDB 连接失败"
fi

# 测试 Redis 连接
redis_addr=$(yq eval '.redis.address[0]' config/config.yaml)
redis-cli -h ${redis_addr%:*} -p ${redis_addr#*:} ping >/dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "✓ Redis 连接正常"
else
    echo "✗ Redis 连接失败"
fi

# 测试 etcd 连接
etcd_addr=$(yq eval '.discovery.etcd.address[0]' config/config.yaml)
etcdctl --endpoints="$etcd_addr" endpoint health >/dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "✓ etcd 连接正常"
else
    echo "✗ etcd 连接失败"
fi
```

## 配置模板

### 开发环境配置模板

```yaml
# config/config.dev.yaml
env: development

discovery:
  enable: etcd
  etcd:
    address: [127.0.0.1:2379]

mongo:
  uri: mongodb://127.0.0.1:27017/openim_dev
  maxPoolSize: 10

redis:
  address: [127.0.0.1:6379]
  db: 1

log:
  remainLogLevel: 6  # DEBUG 级别
  isStdout: true
  withStack: true

callback:
  url: http://127.0.0.1:8080/callback
  # 开发环境启用更多回调用于调试
  beforeSendSingleMsg:
    enable: true
  afterSendSingleMsg:
    enable: true
```

### 生产环境配置模板

```yaml
# config/config.prod.yaml
env: production

discovery:
  enable: etcd
  etcd:
    address: [etcd1:2379, etcd2:2379, etcd3:2379]
    username: "openim"
    password: "secure_password"

mongo:
  uri: mongodb://user:pass@mongo1:27017,mongo2:27017,mongo3:27017/openim_prod?replicaSet=rs0&ssl=true
  maxPoolSize: 100

redis:
  clusterMode: true
  address: [redis1:6379, redis2:6379, redis3:6379]
  password: "redis_secure_password"

log:
  remainLogLevel: 4  # INFO 级别
  isStdout: false
  isJson: true
  storageLocation: /var/log/openim/

tokenPolicy:
  expire: 7  # 7天过期

object:
  enable: aws
  aws:
    region: us-west-2
    bucket: openim-prod-bucket
    accessKeyID: "AWS_ACCESS_KEY"
    secretAccessKey: "AWS_SECRET_KEY"
```

这份配置指南提供了 OpenIM Server 的完整配置说明，包括各种环境下的配置模板和最佳实践，帮助用户正确配置和优化系统。
