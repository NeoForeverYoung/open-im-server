# OpenIM Server 部署指南

## 部署概述

OpenIM Server 支持多种部署方式，包括源码部署、Docker 部署和 Kubernetes 部署。本文档将详细介绍各种部署方式的具体步骤。

## 环境要求

### 系统要求
- **操作系统**: Linux (推荐 Ubuntu 20.04+, CentOS 7+)
- **CPU**: 2 核心以上
- **内存**: 4GB 以上
- **存储**: 50GB 以上可用空间

### 软件依赖
- **Go**: 1.22.7+
- **MongoDB**: 4.4+
- **Redis**: 6.0+
- **etcd**: 3.5+
- **Kafka**: 2.8+ (可选)
- **Docker**: 20.10+ (Docker 部署)
- **Kubernetes**: 1.20+ (K8s 部署)

## 源码部署

### 1. 环境准备

#### 安装 Go
```bash
# 下载 Go
wget https://golang.org/dl/go1.22.7.linux-amd64.tar.gz

# 解压安装
sudo tar -C /usr/local -xzf go1.22.7.linux-amd64.tar.gz

# 配置环境变量
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
source ~/.bashrc
```

#### 安装 MongoDB
```bash
# Ubuntu/Debian
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# 启动 MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod
```

#### 安装 Redis
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install redis-server

# 启动 Redis
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

#### 安装 etcd
```bash
# 下载 etcd
ETCD_VER=v3.5.13
wget https://github.com/etcd-io/etcd/releases/download/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz

# 解压安装
tar xzf etcd-${ETCD_VER}-linux-amd64.tar.gz
sudo mv etcd-${ETCD_VER}-linux-amd64/etcd* /usr/local/bin/

# 启动 etcd
etcd --data-dir=/tmp/etcd-data --name node1 --initial-advertise-peer-urls http://localhost:2380 --listen-peer-urls http://localhost:2380 --advertise-client-urls http://localhost:2379 --listen-client-urls http://localhost:2379 --initial-cluster node1=http://localhost:2380
```

### 2. 下载源码

```bash
# 克隆项目
git clone https://github.com/openimsdk/open-im-server.git
cd open-im-server

# 切换到稳定版本
git checkout main
```

### 3. 配置文件

#### 复制配置文件
```bash
# 复制配置文件模板
cp config/config.yaml.example config/config.yaml
cp config/notification.yaml.example config/notification.yaml
```

#### 修改主配置文件 `config/config.yaml`
```yaml
# 服务配置
api:
  openImApiPort: [10002]
  
# 数据库配置
mongo:
  uri: mongodb://localhost:27017
  database: openIM_v3
  maxPoolSize: 100
  
# Redis 配置
redis:
  address: [localhost:6379]
  username: ""
  password: ""
  
# etcd 配置
etcd:
  address: [localhost:2379]
  
# 对象存储配置 (可选)
object:
  enable: "minio"
  minio:
    bucket: "openim"
    endpoint: "http://localhost:9000"
    accessKeyID: "minioadmin"
    secretAccessKey: "minioadmin"
```

### 4. 编译项目

```bash
# 安装依赖
go mod download

# 编译所有服务
make build

# 或者单独编译
make build-api
make build-rpc
make build-msggateway
make build-msgtransfer
make build-push
```

### 5. 初始化数据

```bash
# 初始化 MongoDB 数据
./scripts/mongo-init.sh

# 初始化配置
./scripts/init-config.sh
```

### 6. 启动服务

#### 启动所有服务
```bash
# 使用脚本启动
./scripts/start-all.sh

# 或者手动启动各个服务
nohup ./bin/openim-api --config_folder_path ./config > ./logs/api.log 2>&1 &
nohup ./bin/openim-rpc-user --config_folder_path ./config > ./logs/rpc-user.log 2>&1 &
nohup ./bin/openim-rpc-friend --config_folder_path ./config > ./logs/rpc-friend.log 2>&1 &
nohup ./bin/openim-rpc-group --config_folder_path ./config > ./logs/rpc-group.log 2>&1 &
nohup ./bin/openim-rpc-msg --config_folder_path ./config > ./logs/rpc-msg.log 2>&1 &
nohup ./bin/openim-msggateway --config_folder_path ./config > ./logs/msggateway.log 2>&1 &
nohup ./bin/openim-msgtransfer --config_folder_path ./config > ./logs/msgtransfer.log 2>&1 &
nohup ./bin/openim-push --config_folder_path ./config > ./logs/push.log 2>&1 &
```

#### 验证服务状态
```bash
# 检查进程
ps aux | grep openim

# 检查端口
netstat -tlnp | grep -E "(10001|10002|10003|10004|10005|10006|10007|10008)"

# 检查日志
tail -f ./logs/api.log
```

## Docker 部署

### 1. 安装 Docker

```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 启动 Docker
sudo systemctl start docker
sudo systemctl enable docker

# 安装 Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 2. 使用 Docker Compose

#### 下载 docker-compose.yml
```bash
# 克隆项目
git clone https://github.com/openimsdk/open-im-server.git
cd open-im-server

# 或者直接下载 docker-compose 文件
wget https://raw.githubusercontent.com/openimsdk/open-im-server/main/docker-compose.yml
```

#### 配置环境变量
```bash
# 创建 .env 文件
cat > .env << EOF
# 基础配置
OPENIM_IP=your_server_ip
OPENIM_SECRET=openIM123

# 数据库配置
MONGO_URI=mongodb://mongo:27017
REDIS_ADDRESS=redis:6379
ETCD_ADDRESS=etcd:2379

# 对象存储配置
MINIO_ENDPOINT=http://minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
EOF
```

#### 启动服务
```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f openim-api
```

### 3. 自定义 Docker 镜像

#### Dockerfile 示例
```dockerfile
FROM golang:1.22.7-alpine AS builder

WORKDIR /app
COPY . .

RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux go build -o openim-api ./cmd/openim-api

FROM alpine:latest

RUN apk --no-cache add ca-certificates tzdata
WORKDIR /app

COPY --from=builder /app/openim-api .
COPY --from=builder /app/config ./config

EXPOSE 10002

CMD ["./openim-api", "--config_folder_path", "./config"]
```

#### 构建镜像
```bash
# 构建镜像
docker build -t openim/openim-api:latest .

# 推送到仓库
docker push openim/openim-api:latest
```

## Kubernetes 部署

### 1. 准备 K8s 集群

#### 安装 kubectl
```bash
# 下载 kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# 安装 kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### 2. 部署依赖服务

#### MongoDB 部署
```yaml
# mongodb.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:4.4
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: "admin"
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: "password"
        volumeMounts:
        - name: mongodb-data
          mountPath: /data/db
      volumes:
      - name: mongodb-data
        persistentVolumeClaim:
          claimName: mongodb-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  selector:
    app: mongodb
  ports:
  - port: 27017
    targetPort: 27017
```

#### Redis 部署
```yaml
# redis.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6.2
        ports:
        - containerPort: 6379
---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
```

### 3. 部署 OpenIM 服务

#### ConfigMap 配置
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: openim-config
data:
  config.yaml: |
    api:
      openImApiPort: [10002]
    mongo:
      uri: mongodb://admin:password@mongodb:27017
      database: openIM_v3
    redis:
      address: [redis:6379]
    etcd:
      address: [etcd:2379]
```

#### API 服务部署
```yaml
# openim-api.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: openim-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: openim-api
  template:
    metadata:
      labels:
        app: openim-api
    spec:
      containers:
      - name: openim-api
        image: openim/openim-api:latest
        ports:
        - containerPort: 10002
        volumeMounts:
        - name: config
          mountPath: /app/config
        env:
        - name: CONFIG_PATH
          value: "/app/config"
      volumes:
      - name: config
        configMap:
          name: openim-config
---
apiVersion: v1
kind: Service
metadata:
  name: openim-api
spec:
  selector:
    app: openim-api
  ports:
  - port: 10002
    targetPort: 10002
  type: LoadBalancer
```

#### 部署命令
```bash
# 应用配置
kubectl apply -f mongodb.yaml
kubectl apply -f redis.yaml
kubectl apply -f configmap.yaml
kubectl apply -f openim-api.yaml

# 查看部署状态
kubectl get pods
kubectl get services

# 查看日志
kubectl logs -f deployment/openim-api
```

### 4. Helm 部署

#### 创建 Helm Chart
```bash
# 创建 Chart
helm create openim

# 编辑 values.yaml
cat > openim/values.yaml << EOF
replicaCount: 2

image:
  repository: openim/openim-api
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: LoadBalancer
  port: 10002

mongodb:
  enabled: true
  auth:
    rootPassword: password

redis:
  enabled: true
EOF
```

#### 安装 Chart
```bash
# 安装
helm install openim ./openim

# 升级
helm upgrade openim ./openim

# 卸载
helm uninstall openim
```

## 监控与日志

### 1. Prometheus 监控

```yaml
# prometheus.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'openim-api'
      static_configs:
      - targets: ['openim-api:10002']
```

### 2. 日志收集

```yaml
# fluentd.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## 性能调优

### 1. 系统参数优化

```bash
# 修改系统限制
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf

# 内核参数优化
echo "net.core.somaxconn = 65535" >> /etc/sysctl.conf
echo "net.ipv4.tcp_max_syn_backlog = 65535" >> /etc/sysctl.conf
sysctl -p
```

### 2. 数据库优化

```javascript
// MongoDB 索引优化
db.users.createIndex({"userID": 1})
db.messages.createIndex({"sendTime": -1})
db.messages.createIndex({"conversationID": 1, "sendTime": -1})
```

### 3. 缓存优化

```bash
# Redis 配置优化
echo "maxmemory 2gb" >> /etc/redis/redis.conf
echo "maxmemory-policy allkeys-lru" >> /etc/redis/redis.conf
```

## 故障排查

### 1. 常见问题

#### 服务启动失败
```bash
# 检查配置文件
cat config/config.yaml

# 检查端口占用
netstat -tlnp | grep 10002

# 检查依赖服务
systemctl status mongod
systemctl status redis-server
```

#### 连接问题
```bash
# 测试数据库连接
mongo mongodb://localhost:27017

# 测试 Redis 连接
redis-cli ping

# 测试 API 接口
curl -X POST http://localhost:10002/api/v3/auth/user_token
```

### 2. 日志分析

```bash
# 查看错误日志
grep -i error ./logs/*.log

# 实时监控日志
tail -f ./logs/api.log | grep -i error

# 分析访问日志
awk '{print $1}' ./logs/access.log | sort | uniq -c | sort -nr
```

这份部署指南涵盖了 OpenIM Server 的各种部署方式，为不同场景提供了详细的部署步骤和最佳实践。