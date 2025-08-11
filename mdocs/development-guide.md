# OpenIM Server 开发指南

## 开发环境搭建

### 1. 环境要求

- **Go**: 1.22.7+
- **Git**: 2.0+
- **IDE**: VS Code / GoLand (推荐)
- **操作系统**: Linux / macOS / Windows

### 2. 开发工具安装

#### Go 开发环境
```bash
# 安装 Go
wget https://golang.org/dl/go1.22.7.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.7.linux-amd64.tar.gz

# 配置环境变量
export PATH=$PATH:/usr/local/go/bin
export GOPROXY=https://goproxy.cn,direct
export GO111MODULE=on
```

#### 开发工具
```bash
# 安装代码格式化工具
go install golang.org/x/tools/cmd/goimports@latest
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# 安装调试工具
go install github.com/go-delve/delve/cmd/dlv@latest

# 安装 protobuf 工具
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

### 3. 项目结构

```
open-im-server/
├── cmd/                    # 应用程序入口
│   ├── openim-api/        # API 服务
│   ├── openim-rpc-user/   # 用户 RPC 服务
│   ├── openim-rpc-friend/ # 好友 RPC 服务
│   ├── openim-rpc-group/  # 群组 RPC 服务
│   ├── openim-rpc-msg/    # 消息 RPC 服务
│   ├── openim-msggateway/ # 消息网关
│   ├── openim-msgtransfer/# 消息传输
│   └── openim-push/       # 推送服务
├── internal/              # 内部包
│   ├── api/              # API 接口层
│   ├── rpc/              # RPC 服务层
│   ├── msggateway/       # 消息网关
│   ├── msgtransfer/      # 消息传输
│   └── push/             # 推送服务
├── pkg/                   # 公共包
│   ├── common/           # 通用工具
│   ├── config/           # 配置管理
│   ├── db/               # 数据库操作
│   ├── utils/            # 工具函数
│   └── proto/            # Protocol Buffers
├── scripts/              # 脚本文件
├── config/               # 配置文件
├── deployments/          # 部署配置
├── docs/                 # 文档
├── test/                 # 测试文件
├── tools/                # 工具集
├── Makefile             # 构建脚本
├── go.mod               # Go 模块文件
└── go.sum               # 依赖校验文件
```

## 代码规范

### 1. 命名规范

#### 包命名
```go
// 好的包名
package user
package message
package config

// 避免的包名
package userService
package messageHandler
package configManager
```

#### 变量命名
```go
// 局部变量使用驼峰命名
var userName string
var messageCount int

// 常量使用大写字母和下划线
const MAX_MESSAGE_LENGTH = 1000
const DEFAULT_TIMEOUT = 30

// 导出的变量使用大写字母开头
var DefaultConfig = &Config{}
```

#### 函数命名
```go
// 导出函数使用大写字母开头
func GetUserInfo(userID string) (*User, error) {
    // 实现
}

// 私有函数使用小写字母开头
func validateUserID(userID string) bool {
    // 实现
}
```

### 2. 代码格式

#### 使用 gofmt 格式化
```bash
# 格式化单个文件
gofmt -w main.go

# 格式化整个项目
gofmt -w .

# 使用 goimports (推荐)
goimports -w .
```

#### 代码注释
```go
// Package user 提供用户管理相关功能
package user

// User 表示系统用户
type User struct {
    ID       string `json:"id"`       // 用户ID
    Nickname string `json:"nickname"` // 用户昵称
    Email    string `json:"email"`    // 邮箱地址
}

// GetUserByID 根据用户ID获取用户信息
// 参数:
//   userID: 用户唯一标识符
// 返回:
//   *User: 用户信息，如果用户不存在则返回 nil
//   error: 错误信息
func GetUserByID(userID string) (*User, error) {
    if userID == "" {
        return nil, errors.New("用户ID不能为空")
    }
    
    // 实现逻辑
    return nil, nil
}
```

### 3. 错误处理

#### 错误定义
```go
// pkg/common/errors.go
package common

import "errors"

var (
    ErrUserNotFound     = errors.New("用户不存在")
    ErrInvalidParameter = errors.New("参数无效")
    ErrPermissionDenied = errors.New("权限不足")
)

// 自定义错误类型
type APIError struct {
    Code    int    `json:"code"`
    Message string `json:"message"`
}

func (e *APIError) Error() string {
    return e.Message
}

func NewAPIError(code int, message string) *APIError {
    return &APIError{
        Code:    code,
        Message: message,
    }
}
```

#### 错误处理模式
```go
// 标准错误处理
func ProcessUser(userID string) error {
    user, err := GetUserByID(userID)
    if err != nil {
        return fmt.Errorf("获取用户失败: %w", err)
    }
    
    if user == nil {
        return ErrUserNotFound
    }
    
    // 处理逻辑
    return nil
}

// 错误包装
func HandleUserRequest(w http.ResponseWriter, r *http.Request) {
    userID := r.URL.Query().Get("userID")
    
    err := ProcessUser(userID)
    if err != nil {
        if errors.Is(err, ErrUserNotFound) {
            http.Error(w, "用户不存在", http.StatusNotFound)
            return
        }
        
        log.Printf("处理用户请求失败: %v", err)
        http.Error(w, "内部服务器错误", http.StatusInternalServerError)
        return
    }
    
    // 成功响应
    w.WriteHeader(http.StatusOK)
}
```

## 开发流程

### 1. 分支管理

#### Git Flow 工作流
```bash
# 主分支
main        # 生产环境代码
develop     # 开发环境代码

# 功能分支
feature/user-management
feature/message-system

# 发布分支
release/v1.0.0

# 热修复分支
hotfix/critical-bug-fix
```

#### 分支操作
```bash
# 创建功能分支
git checkout -b feature/new-feature develop

# 开发完成后合并到 develop
git checkout develop
git merge --no-ff feature/new-feature

# 创建发布分支
git checkout -b release/v1.0.0 develop

# 发布完成后合并到 main 和 develop
git checkout main
git merge --no-ff release/v1.0.0
git checkout develop
git merge --no-ff release/v1.0.0
```

### 2. 提交规范

#### Commit Message 格式
```
<type>(<scope>): <subject>

<body>

<footer>
```

#### 类型说明
- `feat`: 新功能
- `fix`: 修复 bug
- `docs`: 文档更新
- `style`: 代码格式调整
- `refactor`: 代码重构
- `test`: 测试相关
- `chore`: 构建过程或辅助工具的变动

#### 示例
```bash
git commit -m "feat(user): 添加用户注册功能

- 实现用户注册 API
- 添加邮箱验证
- 增加密码强度检查

Closes #123"
```

### 3. 代码审查

#### Pull Request 模板
```markdown
## 变更描述
简要描述本次变更的内容和目的

## 变更类型
- [ ] 新功能
- [ ] Bug 修复
- [ ] 文档更新
- [ ] 代码重构
- [ ] 性能优化

## 测试
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 手动测试完成

## 检查清单
- [ ] 代码符合规范
- [ ] 添加了必要的注释
- [ ] 更新了相关文档
- [ ] 没有引入新的警告
```

## 测试指南

### 1. 单元测试

#### 测试文件结构
```go
// internal/api/user_test.go
package api

import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
)

// 测试用户注册
func TestUserRegister(t *testing.T) {
    tests := []struct {
        name    string
        input   RegisterRequest
        want    *RegisterResponse
        wantErr bool
    }{
        {
            name: "正常注册",
            input: RegisterRequest{
                UserID:   "user123",
                Nickname: "测试用户",
                Email:    "test@example.com",
            },
            want: &RegisterResponse{
                UserID: "user123",
            },
            wantErr: false,
        },
        {
            name: "用户ID为空",
            input: RegisterRequest{
                UserID:   "",
                Nickname: "测试用户",
                Email:    "test@example.com",
            },
            want:    nil,
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := UserRegister(tt.input)
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            
            assert.NoError(t, err)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

#### Mock 测试
```go
// 定义 Mock 接口
type MockUserService struct {
    mock.Mock
}

func (m *MockUserService) GetUser(userID string) (*User, error) {
    args := m.Called(userID)
    return args.Get(0).(*User), args.Error(1)
}

// 使用 Mock 进行测试
func TestUserHandler(t *testing.T) {
    mockService := new(MockUserService)
    handler := NewUserHandler(mockService)
    
    // 设置 Mock 期望
    mockService.On("GetUser", "user123").Return(&User{
        ID:       "user123",
        Nickname: "测试用户",
    }, nil)
    
    // 执行测试
    result, err := handler.HandleGetUser("user123")
    
    // 验证结果
    assert.NoError(t, err)
    assert.Equal(t, "user123", result.ID)
    
    // 验证 Mock 调用
    mockService.AssertExpectations(t)
}
```

### 2. 集成测试

#### 数据库测试
```go
// test/integration/user_test.go
package integration

import (
    "testing"
    "context"
    "go.mongodb.org/mongo-driver/mongo"
    "go.mongodb.org/mongo-driver/mongo/options"
)

func TestUserCRUD(t *testing.T) {
    // 连接测试数据库
    client, err := mongo.Connect(context.Background(), options.Client().ApplyURI("mongodb://localhost:27017"))
    assert.NoError(t, err)
    defer client.Disconnect(context.Background())
    
    db := client.Database("test_openim")
    userRepo := NewUserRepository(db)
    
    // 测试创建用户
    user := &User{
        ID:       "test_user",
        Nickname: "测试用户",
        Email:    "test@example.com",
    }
    
    err = userRepo.Create(user)
    assert.NoError(t, err)
    
    // 测试查询用户
    found, err := userRepo.GetByID("test_user")
    assert.NoError(t, err)
    assert.Equal(t, user.Nickname, found.Nickname)
    
    // 清理测试数据
    err = userRepo.Delete("test_user")
    assert.NoError(t, err)
}
```

### 3. 性能测试

#### 基准测试
```go
// 基准测试示例
func BenchmarkUserValidation(b *testing.B) {
    user := &User{
        ID:       "user123",
        Nickname: "测试用户",
        Email:    "test@example.com",
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        ValidateUser(user)
    }
}

// 并发基准测试
func BenchmarkUserValidationParallel(b *testing.B) {
    user := &User{
        ID:       "user123",
        Nickname: "测试用户",
        Email:    "test@example.com",
    }
    
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            ValidateUser(user)
        }
    })
}
```

## 调试技巧

### 1. 使用 Delve 调试器

#### 安装和使用
```bash
# 安装 Delve
go install github.com/go-delve/delve/cmd/dlv@latest

# 调试程序
dlv debug ./cmd/openim-api

# 在调试器中设置断点
(dlv) break main.main
(dlv) break internal/api/user.go:25

# 运行程序
(dlv) continue

# 查看变量
(dlv) print userID
(dlv) locals
(dlv) args
```

### 2. 日志调试

#### 结构化日志
```go
import (
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

// 初始化日志
func InitLogger() *zap.Logger {
    config := zap.NewProductionConfig()
    config.Level = zap.NewAtomicLevelAt(zapcore.DebugLevel)
    
    logger, _ := config.Build()
    return logger
}

// 使用日志
func ProcessUser(userID string) error {
    logger := InitLogger()
    defer logger.Sync()
    
    logger.Info("开始处理用户",
        zap.String("userID", userID),
        zap.Time("timestamp", time.Now()),
    )
    
    user, err := GetUserByID(userID)
    if err != nil {
        logger.Error("获取用户失败",
            zap.String("userID", userID),
            zap.Error(err),
        )
        return err
    }
    
    logger.Debug("用户信息",
        zap.String("userID", user.ID),
        zap.String("nickname", user.Nickname),
    )
    
    return nil
}
```

### 3. 性能分析

#### pprof 性能分析
```go
import (
    _ "net/http/pprof"
    "net/http"
)

func main() {
    // 启动 pprof 服务
    go func() {
        log.Println(http.ListenAndServe("localhost:6060", nil))
    }()
    
    // 主程序逻辑
    // ...
}
```

#### 使用 pprof 工具
```bash
# CPU 性能分析
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# 内存分析
go tool pprof http://localhost:6060/debug/pprof/heap

# 查看 goroutine
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

## 最佳实践

### 1. 代码组织

#### 依赖注入
```go
// 定义接口
type UserService interface {
    GetUser(userID string) (*User, error)
    CreateUser(user *User) error
}

// 实现接口
type userService struct {
    repo UserRepository
    logger *zap.Logger
}

func NewUserService(repo UserRepository, logger *zap.Logger) UserService {
    return &userService{
        repo:   repo,
        logger: logger,
    }
}

// 使用依赖注入
func main() {
    logger := InitLogger()
    repo := NewUserRepository(db)
    service := NewUserService(repo, logger)
    handler := NewUserHandler(service)
    
    // 启动服务
}
```

### 2. 配置管理

#### 配置结构
```go
// pkg/config/config.go
type Config struct {
    Server   ServerConfig   `yaml:"server"`
    Database DatabaseConfig `yaml:"database"`
    Redis    RedisConfig    `yaml:"redis"`
}

type ServerConfig struct {
    Port    int    `yaml:"port"`
    Host    string `yaml:"host"`
    Timeout int    `yaml:"timeout"`
}

// 加载配置
func LoadConfig(path string) (*Config, error) {
    data, err := ioutil.ReadFile(path)
    if err != nil {
        return nil, err
    }
    
    var config Config
    err = yaml.Unmarshal(data, &config)
    if err != nil {
        return nil, err
    }
    
    return &config, nil
}
```

### 3. 并发处理

#### Worker Pool 模式
```go
type Job struct {
    ID   int
    Data interface{}
}

type Worker struct {
    ID         int
    JobChannel chan Job
    QuitChan   chan bool
}

func (w *Worker) Start() {
    go func() {
        for {
            select {
            case job := <-w.JobChannel:
                // 处理任务
                fmt.Printf("Worker %d processing job %d\n", w.ID, job.ID)
                
            case <-w.QuitChan:
                fmt.Printf("Worker %d stopping\n", w.ID)
                return
            }
        }
    }()
}

// 工作池
type WorkerPool struct {
    Workers    []*Worker
    JobQueue   chan Job
    QuitChan   chan bool
}

func NewWorkerPool(numWorkers int) *WorkerPool {
    pool := &WorkerPool{
        Workers:  make([]*Worker, numWorkers),
        JobQueue: make(chan Job, 100),
        QuitChan: make(chan bool),
    }
    
    for i := 0; i < numWorkers; i++ {
        worker := &Worker{
            ID:         i,
            JobChannel: make(chan Job),
            QuitChan:   make(chan bool),
        }
        pool.Workers[i] = worker
        worker.Start()
    }
    
    return pool
}
```

这份开发指南为 OpenIM Server 的开发者提供了完整的开发环境搭建、代码规范、测试方法和最佳实践，帮助团队保持代码质量和开发效率。