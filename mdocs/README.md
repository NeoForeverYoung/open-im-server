# OpenIM Server 项目文档

## 项目概述

OpenIM Server 是一个开源的即时通讯服务器解决方案，专为开发者设计，提供完整的即时通讯功能和服务。与传统的独立聊天应用不同，OpenIM 提供的是一套完整的开发工具和框架，帮助开发者在自己的应用中集成即时通讯功能。

## 核心特性

### 🌟 主要功能
- **微服务架构**：支持集群模式，包括网关(gateway)和多个 rpc 服务
- **多样化部署**：支持源代码、Kubernetes 或 Docker 部署
- **海量用户支持**：支持十万级超大群组，千万级用户和百亿级消息
- **跨平台支持**：支持 Linux、Windows、Mac 系统以及 ARM 和 AMD CPU 架构

### 📚 核心组件
1. **OpenIM SDK**：客户端集成SDK，支持多平台
2. **OpenIM Server**：服务端核心，提供完整的IM服务
3. **REST API**：为业务系统提供后台接口
4. **Webhooks**：事件回调机制，扩展业务形态

## 技术架构

### 技术栈
- **开发语言**：Go 1.22.7
- **数据库**：MongoDB
- **缓存**：Redis
- **消息队列**：Kafka
- **服务发现**：etcd
- **容器化**：Docker & Kubernetes
- **监控**：Prometheus

### 架构设计
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Apps   │    │   Web/H5 Apps   │    │  Admin Panel    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   API Gateway   │
                    └─────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Auth Service  │    │  Message Service │    │  User Service   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   Data Layer    │
                    │ MongoDB + Redis │
                    └─────────────────┘
```

## 项目结构

```
open-im-server/
├── cmd/                    # 应用程序入口
├── internal/              # 内部包
│   ├── api/              # API 接口层
│   ├── rpc/              # RPC 服务层
│   ├── msggateway/       # 消息网关
│   ├── msgtransfer/      # 消息传输
│   └── push/             # 推送服务
├── pkg/                   # 公共包
├── config/               # 配置文件
├── scripts/              # 脚本文件
├── deployments/          # 部署配置
├── docs/                 # 官方文档
└── tools/                # 工具集
```

## 快速开始

### 环境要求
- Go 1.22.7+
- MongoDB 4.4+
- Redis 6.0+
- etcd 3.5+

### 安装部署

#### 1. 源码部署
```bash
# 克隆项目
git clone https://github.com/openimsdk/open-im-server.git
cd open-im-server

# 安装依赖
go mod download

# 构建项目
make build

# 启动服务
make start
```

#### 2. Docker 部署
```bash
# 使用 Docker Compose
docker-compose up -d
```

#### 3. Kubernetes 部署
```bash
# 应用 K8s 配置
kubectl apply -f deployments/kubernetes/
```

## 核心功能模块

### 1. 用户管理
- 用户注册/登录
- 用户信息管理
- 好友关系管理
- 黑名单管理

### 2. 消息系统
- 单聊消息
- 群聊消息
- 系统消息
- 消息推送

### 3. 群组管理
- 群组创建/解散
- 成员管理
- 权限控制
- 群组设置

### 4. 文件存储
- 图片/视频/文件上传
- 多云存储支持
- CDN 加速

## API 接口

### REST API
提供完整的 REST API 接口，支持：
- 用户管理接口
- 消息发送接口
- 群组管理接口
- 系统管理接口

### WebSocket API
实时通信接口：
- 消息实时推送
- 在线状态同步
- 心跳保持

## 配置说明

### 主要配置文件
- `config/config.yaml`：主配置文件
- `config/notification.yaml`：通知配置
- `config/webhooks.yaml`：Webhook 配置

### 关键配置项
```yaml
# 服务配置
api:
  openImApiPort: [10002]
  
# 数据库配置
mongo:
  uri: mongodb://localhost:27017
  database: openIM_v3
  
# Redis 配置
redis:
  address: [localhost:6379]
  
# etcd 配置
etcd:
  address: [localhost:2379]
```

## 开发指南

### 代码规范
- 遵循 Go 官方代码规范
- 使用 gofmt 格式化代码
- 编写单元测试
- 添加必要的注释

### 贡献流程
1. Fork 项目
2. 创建功能分支
3. 提交代码
4. 创建 Pull Request

## 监控与运维

### 监控指标
- 服务健康状态
- 消息吞吐量
- 用户在线数
- 系统资源使用

### 日志管理
- 结构化日志
- 日志级别控制
- 日志轮转

## 社区与支持

### 官方资源
- **官网**：https://openim.io
- **文档**：https://docs.openim.io
- **GitHub**：https://github.com/openimsdk/open-im-server

### 社区交流
- **Slack**：加入官方 Slack 群组
- **微信群**：扫码加入微信交流群
- **Twitter**：关注官方 Twitter

## 许可证

本项目采用 Apache License 2.0 开源许可证。

## 版本历史

查看 [CHANGELOG.md](https://github.com/openimsdk/open-im-server/blob/main/CHANGELOG.md) 了解详细的版本更新历史。