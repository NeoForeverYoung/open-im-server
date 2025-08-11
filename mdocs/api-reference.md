# OpenIM Server API 参考文档

## API 概述

OpenIM Server 提供完整的 REST API 和 WebSocket API，支持用户管理、消息处理、群组管理等核心功能。

### 基础信息

- **Base URL**: `https://your-domain.com/api/v3`
- **认证方式**: JWT Token
- **数据格式**: JSON
- **字符编码**: UTF-8

### 通用响应格式

```json
{
  "errCode": 0,
  "errMsg": "success",
  "data": {}
}
```

### 错误码说明

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 1001 | 参数错误 |
| 1002 | Token 无效 |
| 1003 | 权限不足 |
| 1004 | 用户不存在 |
| 1005 | 群组不存在 |

## 认证 API

### 用户注册

**接口地址**: `POST /auth/user_register`

**请求参数**:
```json
{
  "secret": "openIM123",
  "users": [
    {
      "userID": "user123",
      "nickname": "张三",
      "faceURL": "https://example.com/avatar.jpg",
      "ex": "扩展字段"
    }
  ]
}
```

**响应示例**:
```json
{
  "errCode": 0,
  "errMsg": "success",
  "data": {}
}
```

### 用户登录

**接口地址**: `POST /auth/user_token`

**请求参数**:
```json
{
  "secret": "openIM123",
  "userID": "user123",
  "platformID": 1
}
```

**响应示例**:
```json
{
  "errCode": 0,
  "errMsg": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expireTimeSeconds": 604800
  }
}
```

## 用户管理 API

### 获取用户信息

**接口地址**: `POST /user/get_users_info`

**请求头**:
```
Authorization: Bearer <token>
operationID: <operation_id>
```

**请求参数**:
```json
{
  "userIDs": ["user123", "user456"]
}
```

**响应示例**:
```json
{
  "errCode": 0,
  "errMsg": "success",
  "data": {
    "usersInfo": [
      {
        "userID": "user123",
        "nickname": "张三",
        "faceURL": "https://example.com/avatar.jpg",
        "createTime": 1640995200000,
        "ex": "扩展字段"
      }
    ]
  }
}
```

### 更新用户信息

**接口地址**: `POST /user/update_user_info`

**请求参数**:
```json
{
  "userInfo": {
    "userID": "user123",
    "nickname": "新昵称",
    "faceURL": "https://example.com/new_avatar.jpg",
    "ex": "新的扩展字段"
  }
}
```

## 好友管理 API

### 添加好友

**接口地址**: `POST /friend/add_friend`

**请求参数**:
```json
{
  "fromUserID": "user123",
  "toUserID": "user456",
  "reqMsg": "我是张三，请添加我为好友"
}
```

### 获取好友列表

**接口地址**: `POST /friend/get_friend_list`

**请求参数**:
```json
{
  "userID": "user123",
  "pagination": {
    "pageNumber": 1,
    "showNumber": 20
  }
}
```

**响应示例**:
```json
{
  "errCode": 0,
  "errMsg": "success",
  "data": {
    "friendsInfo": [
      {
        "ownerUserID": "user123",
        "friendUser": {
          "userID": "user456",
          "nickname": "李四",
          "faceURL": "https://example.com/avatar2.jpg"
        },
        "addSource": 1,
        "operatorUserID": "user123",
        "createTime": 1640995200000
      }
    ],
    "total": 1
  }
}
```

## 群组管理 API

### 创建群组

**接口地址**: `POST /group/create_group`

**请求参数**:
```json
{
  "memberUserIDs": ["user123", "user456"],
  "groupInfo": {
    "groupName": "技术交流群",
    "introduction": "这是一个技术交流群",
    "faceURL": "https://example.com/group_avatar.jpg",
    "groupType": 2
  },
  "adminUserIDs": ["user123"]
}
```

**响应示例**:
```json
{
  "errCode": 0,
  "errMsg": "success",
  "data": {
    "groupInfo": {
      "groupID": "group123",
      "groupName": "技术交流群",
      "ownerUserID": "user123",
      "createTime": 1640995200000,
      "memberCount": 2
    }
  }
}
```

### 获取群组信息

**接口地址**: `POST /group/get_groups_info`

**请求参数**:
```json
{
  "groupIDs": ["group123"]
}
```

### 邀请用户入群

**接口地址**: `POST /group/invite_user_to_group`

**请求参数**:
```json
{
  "groupID": "group123",
  "reason": "邀请加入技术交流群",
  "invitedUserIDs": ["user789"]
}
```

## 消息 API

### 发送消息

**接口地址**: `POST /msg/send_msg`

**请求参数**:
```json
{
  "sendID": "user123",
  "recvID": "user456",
  "groupID": "",
  "senderNickname": "张三",
  "senderFaceURL": "https://example.com/avatar.jpg",
  "senderPlatformID": 1,
  "content": {
    "text": "你好，这是一条测试消息"
  },
  "contentType": 101,
  "sessionType": 1,
  "isOnlineOnly": false,
  "offlinePushInfo": {
    "title": "新消息",
    "desc": "你收到了一条新消息",
    "ex": ""
  }
}
```

**响应示例**:
```json
{
  "errCode": 0,
  "errMsg": "success",
  "data": {
    "serverMsgID": "msg123456",
    "clientMsgID": "client_msg_123",
    "sendTime": 1640995200000
  }
}
```

### 获取历史消息

**接口地址**: `POST /msg/get_conversation_msg`

**请求参数**:
```json
{
  "conversationID": "single_user123_user456",
  "startClientMsgID": "",
  "count": 20
}
```

## WebSocket API

### 连接建立

**连接地址**: `ws://your-domain.com/msg_gateway`

**连接参数**:
- `token`: JWT Token
- `userID`: 用户ID
- `platformID`: 平台ID

### 消息格式

**发送消息**:
```json
{
  "reqIdentifier": 1001,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "sendID": "user123",
  "operationID": "op123456",
  "msgIncr": "1",
  "data": {
    "clientMsgID": "client_msg_123",
    "serverMsgID": "",
    "createTime": 1640995200000,
    "sendID": "user123",
    "recvID": "user456",
    "msgFrom": 100,
    "contentType": 101,
    "platformID": 1,
    "senderNickname": "张三",
    "senderFaceURL": "https://example.com/avatar.jpg",
    "sessionType": 1,
    "msgType": 1,
    "content": "你好，这是一条实时消息",
    "seq": 0,
    "isRead": false,
    "status": 1
  }
}
```

**接收消息**:
```json
{
  "reqIdentifier": 1001,
  "msgIncr": "1",
  "operationID": "op123456",
  "data": {
    "clientMsgID": "client_msg_456",
    "serverMsgID": "server_msg_456",
    "createTime": 1640995200000,
    "sendTime": 1640995200000,
    "sendID": "user456",
    "recvID": "user123",
    "msgFrom": 100,
    "contentType": 101,
    "platformID": 1,
    "senderNickname": "李四",
    "senderFaceURL": "https://example.com/avatar2.jpg",
    "sessionType": 1,
    "msgType": 1,
    "content": "收到，谢谢！",
    "seq": 1,
    "isRead": false,
    "status": 2
  }
}
```

## 管理员 API

### 获取用户统计

**接口地址**: `POST /manager/get_users_online_status`

**请求参数**:
```json
{
  "userIDs": ["user123", "user456"]
}
```

### 封禁用户

**接口地址**: `POST /manager/forbid_user`

**请求参数**:
```json
{
  "userID": "user123",
  "forbiddenType": 1,
  "reason": "违规行为"
}
```

## SDK 集成示例

### JavaScript SDK

```javascript
import OpenIMSDK from 'open-im-sdk-wasm';

// 初始化 SDK
const openIM = new OpenIMSDK();

// 登录
await openIM.login({
  userID: 'user123',
  token: 'your_jwt_token',
  platformID: 5
});

// 发送消息
await openIM.sendMessage({
  recvID: 'user456',
  groupID: '',
  message: {
    contentType: 101,
    content: JSON.stringify({ text: '你好' })
  }
});

// 监听消息
openIM.on('onRecvNewMessage', (data) => {
  console.log('收到新消息:', data);
});
```

### Go SDK

```go
package main

import (
    "github.com/openimsdk/openim-sdk-core/v3/open_im_sdk"
    "github.com/openimsdk/openim-sdk-core/v3/pkg/sdk_params_callback"
)

func main() {
    // 初始化 SDK
    openIM := open_im_sdk.NewOpenIMSDK()
    
    // 登录
    err := openIM.Login(&sdk_params_callback.LoginParams{
        UserID:     "user123",
        Token:      "your_jwt_token",
        PlatformID: 1,
    })
    
    if err != nil {
        panic(err)
    }
    
    // 发送消息
    _, err = openIM.SendMessage(&sdk_params_callback.SendMessageParams{
        RecvID:      "user456",
        GroupID:     "",
        ContentType: 101,
        Content:     `{"text":"你好"}`,
    })
}
```

## 最佳实践

### 1. 认证安全
- 定期刷新 Token
- 使用 HTTPS 传输
- 妥善保管密钥

### 2. 消息处理
- 实现消息去重
- 处理网络异常
- 缓存离线消息

### 3. 性能优化
- 合理使用分页
- 避免频繁请求
- 使用连接池

### 4. 错误处理
- 统一错误码处理
- 实现重试机制
- 记录错误日志

这份 API 文档涵盖了 OpenIM Server 的主要接口，为开发者提供了完整的集成指南。