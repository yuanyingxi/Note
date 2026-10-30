# 1. 项目结构

```
MiniChat/
├── client/                     # Vue 3 + Electron 前端
│   ├── electron/               # Electron 主进程 / preload
│   ├── src/
│   │   ├── api/                # REST 封装 + WebSocket 客户端
│   │   ├── components/         # 11 个 Vue 组件
│   │   ├── stores/             # Pinia（user / chat）
│   │   ├── views/              # Login / Register / ChatView
│   │   ├── types/              # TS 类型定义
│   │   └── utils/              # request(axios) / env(地址推导)
│   └── release/                # Electron 打包产物
├── server/                     # Maven 父工程（壳）
│   ├── pom.xml                 # 依赖版本统一管理
│   ├── minichat-common/        # 公共 DTO / Result / 常量 / JacksonConfig
│   ├── minichat-gateway/       # API 网关 :8080
│   ├── minichat-user/          # 用户服务 :8081
│   ├── minichat-message/       # 消息服务 :8082
│   ├── start.ps1 / start.sh    # 一键编译 + 启动
│   └── rebuild.ps1             # 仅编译
└── deploy/                     # Docker 中间件编排
    ├── docker-compose.yml
    ├── .env / .env.example
    ├── sql/user-init.sql       # 用户库建表（未入 git）
    └── data/                   # 中间件持久化数据（未入 git）
```

**四个 Maven 模块**：`minichat-common` → `minichat-message` → `minichat-user` → `minichat-gateway`

# 2 技术栈与版本

### 框架版本

| 框架 | 版本 |
|---|---|
| Java | 21 |
| Spring Boot | 3.2.5 |
| Spring Cloud | 2023.0.2 |
| Spring Cloud Alibaba | 2023.0.1.0 |

### 中间件版本

| 层 | 技术 |
|---|---|
| 前端 | Vue 3.5 + TypeScript + Pinia 3 + Element Plus 2.14 + Axios + Electron 31 |
| 网关 | Spring Cloud Gateway + Nacos 注册中心 + JWT 鉴权 |
| 用户服务 | Spring Boot 3.2 + Security + MyBatis-Plus + MySQL + Redis + Elasticsearch |
| 消息服务 | Spring Boot 3.2 + WebSocket + RocketMQ + Elasticsearch + OSS + Parquet |
| 中间件 | Nacos 2.2.3 / MySQL 8 / Redis 7 / RocketMQ 5.1.4 / ES 7.17.10 / Kibana / Nginx |

### 第三方依赖版本

| 依赖 | 版本 | 用途 |
|---|---|---|
| MyBatis-Plus | 3.5.7 | ORM |
| Hutool | 5.8.28（message 模块覆盖为 5.8.40） | Java 工具类 |
| 阿里云 OSS | user: `aliyun-sdk-oss` 3.17.4（v1）；message: `alibabacloud-oss-v2` 0.4.0（v2） | 文件存储 |
| Knife4j | 4.4.0 | API 文档 |
| Redisson | 3.34.0 | 分布式锁 / RMapCache |
| JJWT | 0.12.5 | JWT |
| RocketMQ | 5.1.4（message 模块覆盖为 5.2.0 / starter 2.3.5） | 消息队列 |
| Parquet | 1.17.1 | 冷数据归档 |

> 父 `pom.xml` 统一 `dependencyManagement`，但 message 模块内显式覆盖了 Hutool / RocketMQ 版本，绕开了父 pom。

# 3 快速开始

### 前置条件

| 工具 | 要求 | 验证命令 |
|---|---|---|
| JDK | 21 | `java -version` |
| Maven | 3.9 | `mvn --version` |
| Node | ^20.19 \|\| >=22.12 | `node -v` |
| Docker Desktop | 稳定版 | `docker --version` |

### 启动步骤

**1. 启动中间件**

```bash
cd deploy
cp .env.example .env      # 首次
docker-compose up -d
```

**2. 初始化数据库**

```bash
docker exec -i minichat-mysql mysql -uroot -p123456 --default-character-set=utf8mb4 \
  minichat_user_db < deploy/sql/user-init.sql
```

**3. 编译并启动后端**

```powershell
cd server
.\start.ps1               # 编译 + 启动（3 个独立窗口）
# 或
.\rebuild.ps1             # 仅编译
.\start.ps1 run           # 跳过编译直接启动
```

启动顺序：gateway(8080) → user(8081) → message(8082)，间隔 10 秒。

**4. 启动前端**

```bash
cd client
npm install               # 首次
npm run dev               # Web: http://localhost:5173
npm run electron:dev      # Electron 开发模式
npm run electron:build    # 打包桌面端
```

### 默认账号与入口

| 服务 | 地址 | 账号/密码 |
|---|---|---|
| 前端 | http://localhost:5173 | — |
| 网关 | http://localhost:8080 | — |
| 用户服务文档 | http://localhost:8081/doc.html | — |
| Nacos | http://localhost:8848/nacos | nacos / nacos |
| Kibana | http://localhost:5601 | — |
| MySQL | localhost:3306 | root / 123456 |
| Redis | localhost:6379 | 123456 |

# 4 架构与核心链路

## 4.1 服务拓扑

```
客户端(Web/Electron)
   │  HTTP :8080 (REST)  +  WS :8080/ws?token=xxx
   ▼
minichat-gateway :8080   ← AuthGlobalFilter 验 JWT，注入 userId header
   ├── lb://minichat-user     :8081  /auth/** /user/** /friend/** /group/** /file/**
   └── lb://minichat-message  :8082  /message/** /conversation/** /oss/** /chat/** /ws
                                        │
                                        └── Feign → /internal/group/{id}/members (user 服务)
```

## 4.2 鉴权链路（关键设计）

1. 客户端登录 → user 服务返回 JWT（subject = userId）
2. 后续请求带 `Authorization: Bearer <token>`
3. **网关注入身份**：`AuthGlobalFilter` 验签后，把 userId 写入请求头 `userId` 转发给下游
4. **下游信任网关**：user 服务 `SecurityConfig` 除 `/internal/**` 外一律 `permitAll()`，Controller 用 `@RequestHeader("userId") Long userId` 取身份

```java
// SecurityConfig 核心
.requestMatchers("/internal/**").denyAll()      // 内部 RPC 禁止外部访问
.requestMatchers("/auth/login","/auth/register","/auth/sms/code").permitAll()
.anyRequest().permitAll()                        // 其余信任网关
```

**网关白名单**：`/auth/login`、`/auth/register`、`/auth/sms/code`、`/doc.html`、`/webjars/**`、`/**/v3/api-docs`、`/swagger-resources`、`/favicon.ico`

**WebSocket 鉴权**：浏览器 WS API 不支持自定义 header，所以网关对 `Upgrade: websocket` 请求从 URL query 取 `?token=`，验签后注入 header。

## 4.3 消息收发链路

项目通过 **WebSocket + MySQL + RocketMQ + Elasticsearch** 实现聊天消息的发送、持久化、实时推送和检索同步。

### 4.3.1 核心组件的职责

|组件|主要职责|
|---|---|
|WebSocket|客户端与服务端之间的双向实时通信|
|`ChatWebSocketHandler`|接收并处理客户端发来的 WebSocket 消息|
|`MessageServiceImpl`|处理聊天消息发送业务，包括消息入库、ACK 返回和异步任务投递|
|MySQL|持久化聊天消息|
|RocketMQ|解耦消息实时推送与 Elasticsearch 同步等异步业务|
|`MessagePusher`|消费 `chat-message` 消息，执行私聊或群聊的实时推送|
|`SessionManager`|管理本地 WebSocket Session，根据用户 ID 查找目标连接|
|`ChatMessageSyncConsumer`|消费 `chat-sync-topic` 消息，将消息数据同步到 Elasticsearch|
|Elasticsearch（ES）|保存消息索引，支持后续检索|

**需要注意：** WebSocket 和 RocketMQ 不是相互替代的通信方式。

- WebSocket 负责客户端与服务端之间的实时通信。
    
- RocketMQ 负责服务端内部的异步消息传递。
    
- MySQL 负责聊天消息的持久化。
    
- Elasticsearch 负责消息索引与检索。
    

### 4.3.2 客户端发送消息的完整流程

```text
客户端 A
  │
  │ ① WebSocket 发送消息
  │    {type: 1, data: SendMessageReq}
  ▼
ChatWebSocketHandler
  │
  │ ② 接收并解析客户端消息
  │    获取发送者身份、校验消息参数
  ▼
MessageServiceImpl.sendMessage()
  │
  ├── ③ messageMapper.insert()
  │       │
  │       ▼
  │     MySQL：持久化聊天消息
  │
  ├── ④ 返回 WebSocket ACK 给发送者 A
  │       {type: 2, data: {messageId}}
  │
  ├── ⑤ 投递 RocketMQ：chat-message
  │       │
  │       ▼
  │     RocketMQ Broker
  │       │
  │       │ push-group 消费消息
  │       ▼
  │     MessagePusher
  │       │
  │       ├── 1. 获取分布式锁
  │       │      RLock: msg:push:{messageId}
  │       │
  │       ├── 2. 检查 Redis 幂等标记
  │       │      RBucket: msg:consumed:{messageId}
  │       │      │
  │       │      ├── 已处理：直接跳过
  │       │      │
  │       │      └── 未处理：继续推送
  │       │
  │       ├── 3. 根据 chatType 分发
  │       │      │
  │       │      ├── 私聊（chatType = 1）
  │       │      │      定位接收者的 WebSocket Session
  │       │      │      通过 WebSocket 推送消息
  │       │      │
  │       │      └── 群聊（chatType = 2）
  │       │             Feign 查询群成员（一次）
  │       │             排除发送者
  │       │             将消息序列化为 JSON（一次）
  │       │             循环定位各成员的 Session
  │       │             逐个推送消息
  │       │
  │       ├── 4. 设置 Redis 幂等标记
  │       │      consumed = true，TTL = 7 天
  │       │
  │       ├── 5. finally 释放分布式锁
  │       │
  │       └── 6. 消费方法正常返回
  │              RocketMQ 确认消费成功
  │
  │
  └── ⑥ 投递 RocketMQ：chat-sync-topic
          │
          ▼
        RocketMQ Broker
          │
          │ chat-sync-consumer-group 消费消息
          ▼
        ChatMessageSyncConsumer
          │
          │ 将消息同步到 Elasticsearch
          ▼
        Elasticsearch


接收方 B / 群聊其他成员
  │
  │ 接收服务端 WebSocket 下行消息
  │ {type: 0, data: ChatMessage}
  ▼
客户端展示聊天消息
```

1. **WebSocket ACK（type = 2）**：消息服务返回给发送者，表示服务端已经接受并处理了发送请求，通常意味着消息已经保存到 MySQL，不代表接收者已经收到消息。
    
2. **MessagePusher**：负责消费 `chat-message`，通过 WebSocket 将消息实时推送给接收者。分布式锁用于避免多个实例同时处理同一条消息，幂等标记用于跳过已经处理过的消息。
    
3. **Redis 分布式锁与幂等标记**：
    
    - `msg:push:{messageId}`：控制同一条消息的并发处理。
        
    - `msg:consumed:{messageId}`：记录该消息是否已经完成推送处理，设置 7 天过期时间。
        
4. **私聊与群聊**：
    
    - 私聊：根据接收者 ID 定位 WebSocket Session，直接推送。
        
    - 群聊：通过 Feign 查询群成员，排除发送者，再逐个推送给其他成员。
        
5. **chat-sync-topic**：负责将聊天消息异步同步到 Elasticsearch，供后续消息搜索使用。它与 `chat-message` 是两条独立的消息处理链路。

#### 第一步：接收客户端消息

客户端通过 WebSocket 发送消息，帧中包含消息类型和业务数据：

```json
{
  "type": 1,
  "data": {
    "...": "SendMessageReq 对应的消息字段"
  }
}
```

服务端由 `ChatWebSocketHandler.handleTextMessage()` 接收并处理消息，再调用 `MessageServiceImpl.sendMessage()` 执行业务逻辑。

这里的 `type: 1` 是项目自定义的 WebSocket 消息类型，不是 WebSocket 协议本身规定的业务类型。

#### 第二步：将消息写入 MySQL

`MessageServiceImpl.sendMessage()` 调用：

```java
messageMapper.insert(...)
```

将聊天消息写入 MySQL。

这样，聊天消息可以持久化保存，而不是仅存在于 WebSocket 连接或内存中。

#### 第三步：向发送方返回 ACK

消息入库后，服务端向发送方返回 ACK，示意格式如下：

```json
{
  "type": 2,
  "data": {
    "messageId": "消息ID"
  }
}
```

ACK 用于告知客户端该消息已经被服务端接收并完成相应的入库处理。

**ACK 不等于接收方已经收到消息。** 接收方的实时推送由后续的 RocketMQ 消费流程完成。

#### 第四步：投递 RocketMQ，异步执行后续工作

消息服务将后续任务投递到不同的 RocketMQ Topic：

- `chat-message`：交给 `MessagePusher`，负责聊天消息的实时推送。
    
- `chat-sync-topic`：交给 `ChatMessageSyncConsumer`，负责将消息数据同步到 Elasticsearch。
    

两条链路相互独立，不需要等待实时推送完成后再开始 ES 同步。

#### 第五步：MessagePusher 推送消息

`MessagePusher` 消费 `chat-message` 后，根据消息类型执行不同的推送逻辑。

**私聊流程：**

```text
MessagePusher
  │
  ▼
获取接收者 userId
  │
  ▼
sendToUser(toId)
  │
  ▼
SessionManager 查找用户对应的 WebSocket Session
  │
  ▼
通过 WebSocketSession 推送消息
  │
  ▼
接收方客户端
```

**群聊流程：**

```text
MessagePusher
  │
  ▼
通过 OpenFeign 调用用户服务
  │
  ▼
查询群成员
  │
  ▼
排除发送者
  │
  ▼
遍历接收成员
  │
  ▼
逐个调用 sendToUser(userId)
  │
  ▼
SessionManager 定位对应的 WebSocket Session
  │
  ▼
向在线成员推送消息
```

这里的 OpenFeign 用于跨微服务调用：消息服务需要通过用户服务查询群成员信息。

`SessionManager` 则负责管理本地 WebSocket Session。对于没有可用连接的用户，服务端无法通过当前 WebSocket Session 直接完成实时推送；消息持久化与实时送达是不同的概念。

**重要：项目没有使用 Redis Pub/Sub 进行聊天消息推送。** 实时推送链路是 RocketMQ → `MessagePusher` → `SessionManager` → `WebSocketSession`。

#### 第六步：同步消息到 Elasticsearch

```text
MessageServiceImpl.sendMessage()
  │
  ▼
RocketMQ：chat-sync-topic
  │
  ▼
ChatMessageSyncConsumer
  │
  ▼
Elasticsearch：chat_message 索引
```

该消费者负责将消息数据同步到 ES，支持消息数据的 INSERT、UPDATE、DELETE 等同步操作。

MySQL 是聊天消息的持久化存储，ES 则是用于检索的索引存储，两者的职责不同。

### 4.3.3 WebSocket 消息类型约定

项目使用自定义的 `type` 区分不同业务消息。

|type|含义|方向|
|---|---|---|
|`1`|客户端发送聊天消息|客户端 → 服务端|
|`2`|ACK 确认|服务端 → 客户端|
|`3`|心跳|客户端 → 服务端|
|`0`|服务端推送新消息|服务端 → 客户端|

注意：这些编号是项目内部的消息帧约定，不是 WebSocket 标准协议定义的消息类型。

### 4.3.4 为什么要使用 RocketMQ？

如果将消息持久化、实时推送和 ES 同步全部写在一个同步调用链路中，就会增加业务之间的耦合。

引入 RocketMQ 后：

1. **异步解耦：** 消息入库后，实时推送与 ES 同步通过消息队列分别处理。
    
2. **降低同步等待：** 发送消息的主业务不必同步等待所有后续工作完成。
    
3. **便于扩展：** 可以独立扩展消费者，以应对推送和索引同步的不同负载。
    
4. **消费侧可处理重复投递：** 项目在推送流程中结合 Redisson 分布式锁和消费标记处理并发及重复消费问题。
    

需要注意，使用消息队列并不意味着整个链路天然具备端到端的恰好一次送达保证。消息投递、消费者处理和客户端收到消息是不同阶段，仍需分别考虑失败与重试。

## 4.4 RocketMQ Topic

项目使用 RocketMQ 将不同的异步业务分开处理。目前笔记中记录的三个 Topic 分别负责实时消息推送、ES 同步和历史消息读取时间更新。

### 4.4.1 Topic 与消费者对应关系

|Topic|consumerGroup|消费者|主要作用|
|---|---|---|---|
|`chat-message`|`push-group`|`MessagePusher`|将私聊、群聊消息通过 WebSocket 实时推送给接收者|
|`chat-sync-topic`|`chat-sync-consumer-group`|`ChatMessageSyncConsumer`|将消息数据同步到 Elasticsearch，处理 INSERT、UPDATE、DELETE|
|`history-read`|`history-group`|`MessageTimeUpdater`|用户读取历史消息后，异步刷新相应记录的 `update_time`|

**三个 Topic 是彼此独立的异步业务通道，不是依次执行的三个步骤。**

### 4.4.2 三条消费链路

#### ① chat-message：实时消息推送

```text
MessageServiceImpl.sendMessage()
  │
  ▼
RocketMQ：chat-message
  │
  ▼
MessagePusher
  │
  ├── 私聊
  │     └── sendToUser(toId)
  │
  └── 群聊
        └── Feign 查询群成员
              → 排除发送者
              → 逐个 sendToUser(userId)
  │
  ▼
SessionManager
  │
  ▼
WebSocketSession
  │
  ▼
接收方客户端
```

**核心职责：** 把已经由消息服务处理并持久化的聊天消息，通过异步消费转化为面向接收方的 WebSocket 推送。

`MessagePusher` 负责决定推送对象和执行推送，`SessionManager` 负责找到对应的本地 WebSocket 连接。

#### ② chat-sync-topic：同步到 Elasticsearch

```text
MessageServiceImpl.sendMessage()
  │
  ▼
RocketMQ：chat-sync-topic
  │
  ▼
ChatMessageSyncConsumer
  │
  ▼
同步消息数据
  │
  ▼
Elasticsearch：chat_message 索引
```

**核心职责：** 将消息数据同步到 ES，减少消息主业务与索引维护之间的直接耦合。

MySQL 和 ES 之间的同步通过异步消费完成，因此两者可能短暂存在数据更新时差。不能因为消息已经写入 MySQL，就认定 ES 已经同步完成。

#### ③ history-read：异步更新历史消息读取时间

```text
用户读取历史消息
  │
  ▼
产生历史消息读取时间更新任务
  │
  ▼
RocketMQ：history-read
  │
  ▼
MessageTimeUpdater
  │
  ▼
更新相应记录的 update_time
```

**核心职责：** 将历史消息读取后的时间更新操作异步化，避免把所有更新操作都放在读取请求的同步链路中。

`MessageTimeUpdater` 使用 Redisson 分布式锁，对同一会话相关的并发更新时间操作进行协调，以减少并发更新带来的相互干扰。

### 4.4.3 MessagePusher 为什么还需要 Redisson？

RocketMQ 的消费者可能遇到重复投递，也可能出现同一消息被并发处理的情况。因此，项目在 `MessagePusher` 中结合 Redisson 处理这两个问题。

|机制|项目中的用途|解决的问题|
|---|---|---|
|Redisson `RLock`|`msg:push:{messageId}`|控制同一消息推送关键区的并发执行|
|Redisson `RBucket<Boolean>`|`msg:consumed:{id}`，TTL 7 天|记录消息消费状态，辅助判断重复消费|

**分布式锁和幂等标记不是一回事：**

- 分布式锁解决的是并发问题：避免多个执行者同时进入同一个受保护的关键区。
    
- 幂等标记解决的是重复处理问题：记录某条消息是否已经处理，帮助消费者识别重复投递。
    

因此，不能只加锁就认为重复消费问题已经完全解决，也不能仅凭消费标记就认为并发访问一定安全。两者需要结合具体的业务执行顺序使用。

### 4.4.4 面试时如何解释这套设计？

可以这样回答：

> 我们项目使用 RocketMQ 将聊天消息的实时推送和 Elasticsearch 同步解耦。**消息服务**负责接收聊天请求、将**消息写入 MySQL**，并投**递异步任务**。`chat-message` 由 `MessagePusher` 消费，通过 `SessionManager` 查找 WebSocket Session，完成私聊或群聊推送；`chat-sync-topic` 由 `ChatMessageSyncConsumer` 消费，将消息数据同步到 ES。另外，`history-read` 负责异步更新历史消息的读取时间。推送消费侧还结合 Redisson 分布式锁和消费标记处理并发与重复消费问题。

**一句话记忆：**

- `chat-message`：推送给用户。
    
- `chat-sync-topic`：同步到 ES。
    
- `history-read`：更新历史消息读取时间。
    
- Redisson 锁：控制并发。
    
- 消费标记：辅助处理重复消费。

## 4.5 在线状态与心跳机制

即时通讯系统需要知道用户当前是否在线，同时还要处理网络波动、客户端断开等情况。

MiniChat 使用 **Redis + 本地会话**维护在线状态，并通过 **WebSocket 心跳机制**定期续期。

### 4.5.1 在线状态如何维护？

项目采用 **Redis + 本地内存**保存用户的在线会话：

```
用户建立 WebSocket 连接
        │
        ▼
SessionManager
        │
        ├── 本地内存
        │    └── userId → sessionId → WebSocketSession
        │
        └── Redis
             └── online:users
                  └── 保存用户在线状态，并设置过期时间
```

其中：

- **本地内存**：保存当前服务实例上的 WebSocket Session，负责真正进行消息发送。
    
- **Redis**：记录用户是否在线，并通过 TTL 自动过期，避免用户异常断开后状态一直保持在线。
    
- **SessionManager**：统一管理用户登录、Session 保存、消息发送以及连接自愈。
    

因此可以简单理解为：

> **Redis 负责记录“用户是否在线”，本地 Session 负责“怎么把消息发出去”。**

### 4.5.2 为什么需要心跳？

WebSocket 建立连接后，如果客户端突然断网、网络切换或者电脑休眠，服务端不一定能够立即感知连接已经失效。

因此客户端会定期发送心跳：

```
客户端
   │
   │ WebSocket 心跳
   ▼
服务端
   │
   ▼
更新在线状态 TTL
```

项目中客户端大约每 **30 秒**发送一次心跳。

Redis 中的在线状态设置了过期时间，如果持续收到心跳，就不断续期：

```
收到心跳
   ↓
更新在线状态 TTL
   ↓
继续保持在线

长时间没有收到心跳
   ↓
Redis Key 自动过期
   ↓
用户状态变为离线
```

这样即使客户端异常断开，也不需要依赖客户端主动发送“下线通知”，在线状态最终也能够自动恢复。

### 4.5.3 断线重连

如果 WebSocket 连接断开，客户端会尝试重新建立连接。

项目中采用：

```
WebSocket 连接断开
        ↓
等待约 3 秒
        ↓
重新建立 WebSocket 连接
        ↓
重新注册 Session
        ↓
恢复在线状态
```

因此，网络出现短暂波动时，用户不需要重新登录即可恢复连接。

### 4.5.4 单端登录与 WebSocket 会话管理

在线状态解决的是：

> **“这个用户是否在线？”**

而 `SessionManager` 进一步解决的是：

> **“这个用户在线时，对应的是哪个 WebSocket 连接？”**

因此，用户建立 WebSocket 连接后，服务端需要保存用户和 WebSocket Session 之间的对应关系。

可以简单理解为：

```
userId
  ↓
sessionId
  ↓
WebSocketSession
```

其中：

- `userId`：当前登录用户
    
- `sessionId`：这次 WebSocket 连接对应的会话 ID
    
- `WebSocketSession`：服务端真正用于发送 WebSocket 消息的连接对象
    

#### ① 为什么需要 SessionManager？

假设用户 `123` 收到一条新消息：

```
新消息
  ↓
找到接收者 userId = 123
  ↓
SessionManager 根据 userId 找到对应 Session
  ↓
通过 WebSocketSession 发送给用户 123
```

所以 `SessionManager` 相当于维护了一张：

> **用户 → WebSocket 连接**

的映射关系。

它主要负责：

- 保存 WebSocket Session
    
- 根据 `userId` 查找对应 Session
    
- 向指定用户发送消息
    
- 连接异常后的 Session 自愈
    

#### ② 什么是单端登录？

项目支持**单端登录**，也就是一个用户同时只维护一个有效的 WebSocket 会话。

**单端登录和多端登录的根本区别： 就是“一个 userId 允许绑定几个 WebSocket 连接”，在 Redis 里的映射结构，就是映射的 Value 是单值（String）还是多值（List/Set） 的区别。**

例如：

```
用户 123

第一次登录
    ↓
userId=123 → Session A

之后又建立新的连接
    ↓
userId=123 → Session B
```

Session 管理机制会维护用户当前有效的连接，从而避免同一个用户同时存在多个有效会话。

因此这里的核心不是：

> “SessionManager 本身实现了登录认证”

而是：

> **SessionManager 负责维护用户当前的 WebSocket 会话，为单端登录和消息定向推送提供基础。**

#### ③ Session 自愈

如果 WebSocket 连接出现异常，原来的 Session 可能已经失效。

项目中的 `SessionManager` 会对 Session 进行管理和自愈，使连接状态发生变化后，用户重新建立连接时能够重新注册 Session。

整体可以理解为：

```
WebSocket 建立连接
        ↓
SessionManager 保存 Session
        ↓
userId → sessionId → WebSocketSession
        ↓
正常进行消息收发


如果连接异常
        ↓
旧 Session 失效
        ↓
客户端重新连接
        ↓
重新建立 WebSocketSession
        ↓
SessionManager 更新会话关系
```

### 这一部分和心跳机制的区别

这两个概念容易混淆：

| 机制                  | 主要解决的问题                |
| ------------------- | ---------------------- |
| **Redis 在线状态 + 心跳** | 判断用户现在是不是在线            |
| **SessionManager**  | 找到用户对应的 WebSocket 连接   |
| **断线重连**            | 连接断开后重新建立 WebSocket    |
| **单端登录**            | 控制用户当前有效的 WebSocket 会话 |

所以可以记成一句话：

> **Redis + 心跳负责“在线状态”，SessionManager 负责“连接管理”，断线重连负责“恢复连接”，单端登录负责“控制当前会话”。**

### 4.5.5 整体流程

把在线状态、心跳和断线重连放在一起：

```
                WebSocket 建立连接
                       │
                       ▼
                 SessionManager
                  /          \
                 /            \
                ▼              ▼
         本地 Session          Redis
         保存连接             保存在线状态
                              TTL
                               ▲
                               │
                         客户端定期心跳
                               │
                               │
                         每约 30 秒一次
                               │
                               ▼
                          更新在线状态

              如果 WebSocket 断开
                       │
                       ▼
                  等待约 3 秒
                       │
                       ▼
                   尝试重连
                       │
                       ▼
                恢复 WebSocket Session
```

### 面试怎么说？

如果面试官问：

> **“你们项目是怎么维护用户在线状态的？”**

可以回答：

> 我们使用 **WebSocket 建立长连接**，通过 **Redis 和本地 Session** 共同维护在线状态。**本地**主要**保存 WebSocket Session**，用于**实际的消息推送**；**Redis** 保存**用户在线状态**，并设置 **TTL**。客户端**定期发送心跳**，服务端收到后**续期**，这样可以检测连接是否仍然有效。如果客户端异常断开，没有继续发送心跳，Redis 中的在线状态会自动过期。同时客户端支持断线重连，重新建立 WebSocket 连接后恢复 Session。

## 4.6 冷数据归档

即时通讯系统会不断产生聊天消息，如果所有历史消息都长期保存在 MySQL 中，数据库的数据量会越来越大，影响存储成本和查询性能。

因此，项目采用 **“MySQL 热数据 + OSS 冷数据”** 的方式进行消息冷热分离：

```
近期消息
   ↓
MySQL
   ↓
正常查询、写入

较老的历史消息
   ↓
Parquet 压缩归档
   ↓
OSS 对象存储
```

可以简单理解为：

> **经常使用的数据放 MySQL，不经常使用的历史数据压缩后放 OSS。**

### 4.6.1 为什么要进行冷数据归档？

假设系统运行了很长时间：

```
MySQL
├── 最近消息
├── 几个月前的消息
├── 半年前的消息
└── 很久以前的消息
```

如果所有数据一直留在 MySQL：

- 数据表会越来越大
    
- 数据库存储压力增加
    
- 长期历史数据实际上访问频率较低
    

所以项目会定期把较老的消息从 MySQL 中归档到 OSS。

### 4.6.2 每日自动归档

项目通过定时任务 `ScheduledCleanupTask` 定期执行冷数据归档。

整体流程：

```
每天凌晨 2 点
      ↓
ScheduledCleanupTask
      ↓
ClodDataArchiveService
      ↓
获取需要归档的历史消息
      ↓
写入 Parquet 文件
      ↓
SNAPPY 压缩
      ↓
上传 OSS
```

归档后的文件按照日期保存，例如：

```
OSS
└── cold-data/
      ├── message_2026-07-01.parquet
      ├── message_2026-07-02.parquet
      └── message_2026-07-03.parquet
```

这样历史消息就从数据库中的大量记录，变成了按日期组织的压缩归档文件。

### 4.6.3 为什么使用 Parquet？

Parquet 是一种适合存储结构化数据的列式文件格式。

项目使用 Parquet 保存归档消息，并进行 **SNAPPY 压缩**。

这里重点记住：

> **Parquet 负责结构化存储，SNAPPY 负责压缩，从而降低冷数据的存储空间。**

所以整体可以理解为：

```
MySQL 中的历史消息
        ↓
Parquet
        ↓
SNAPPY 压缩
        ↓
OSS
```

### 4.6.4 为什么归档后还要删除 MySQL 数据？

归档到 OSS 后，原来的历史数据就不需要一直占用 MySQL 的存储空间。

因此流程实际上是：

```
MySQL
  ↓
读取历史消息
  ↓
生成 Parquet
  ↓
上传 OSS
  ↓
确认归档
  ↓
删除 MySQL 中对应的旧数据
```

这样 MySQL 主要保留近期、访问频率较高的数据，而较老的数据进入 OSS。

项目删除历史数据时采用**分批删除**，避免一次删除大量数据给 MySQL 带来过大的压力。

### 4.6.5 如果用户查询很久以前的消息怎么办？

冷数据虽然已经从 MySQL 中删除，但并不代表历史消息丢失。

如果用户查询某个时间范围的历史消息，而 MySQL 中没有查询到结果：

```
查询历史消息
      ↓
MySQL 查询
      ↓
没有找到对应数据
      ↓
用户指定了时间范围？
      ↓
      是
      ↓
从 OSS 查找对应 Parquet
      ↓
下载并读取归档数据
      ↓
恢复到 MySQL
      ↓
继续查询历史消息
```

这个过程可以理解为：

> **OSS 是历史消息的“冷存储”，需要访问较老数据时，再把它恢复回来。**

### 4.6.6 整体流程

把“归档”和“查询恢复”放在一起：

```
                 消息数据
                    │
             ┌──────┴──────┐
             │             │
          近期数据        历史数据
             │             │
             ▼             ▼
           MySQL       Parquet
                           │
                      SNAPPY 压缩
                           │
                           ▼
                          OSS
                           │
                     需要查询历史
                           │
                           ▼
                    从 OSS 恢复
                           │
                           ▼
                         MySQL
```

因此项目的冷热数据架构可以概括为：

> **MySQL 负责热数据，OSS 负责冷数据；定时任务负责将历史消息归档，查询冷数据时再从 OSS 恢复。**

### 面试怎么说？

如果面试官问：

> **“你们项目是怎么处理大量历史聊天消息的？”**

可以回答：

> 我们采用了**消息冷热分离**的方案。**MySQL** 主要保存**近期的热数据**，**定时任务**会把较老的历史消息生成 **Parquet 文件**并进行**压缩**，然后**归档到 OSS**，之后**删除 MySQL 中对应的旧数据**，从而控制数据库规模。如果用户查询较老的历史消息，而 MySQL 中已经没有对应数据，系统会根据查询时间范围**从 OSS 获取归档文件并恢复数据**，再完成历史消息查询。


## 4.7 Redisson 与 Redis

### 4.7.1 Redisson 是什么？

**Redisson 不是 Redis 本身，而是一个基于 Redis 的 Java 客户端/高级封装。**

可以简单理解为：

```text
                    Redis
              （Redis 服务端）
                    ▲
                    │
        ┌───────────┴───────────┐
        │                       │
StringRedisTemplate          Redisson
        │                       │
   简单 KV 操作              高级 Redis 对象
   get / set / delete          RLock
                              RBucket
                              RMapCache
```

也就是说：

> **Redis 是真正提供数据存储和共享能力的服务；Redisson 是 Java 应用访问 Redis 的一种客户端，并在此基础上提供了分布式锁、分布式对象等高级功能。**

项目中同时使用了：

- `StringRedisTemplate`：适合简单的 Redis KV 操作
    
- `Redisson`：适合分布式锁、带 TTL 的 Map、幂等标记等高级场景
    

### 4.7.2 为什么项目同时使用 Redis 和 Redisson？

两者并不是重复的。

可以简单理解成：

```text
Redis
 ↓
真正的 Redis 服务端

StringRedisTemplate
 ↓
简单操作 Redis
 ↓
验证码、JWT 黑名单等

Redisson
 ↓
高级操作 Redis
 ↓
分布式锁、幂等控制、在线状态
```

例如用户服务中的短信验证码：

```text
StringRedisTemplate
      ↓
sms:{phone}
      ↓
保存验证码
      ↓
TTL 5 分钟
```

这种场景只需要：

```text
GET
SET
DELETE
```

所以使用 `StringRedisTemplate` 就足够了。

而消息服务需要：

```text
RLock
RBucket
RMapCache
```

这些高级能力，因此使用 Redisson。

### 4.7.3 项目中 Redisson 的三个主要用途

项目中的 Redisson 主要有三个用途：

|Redisson API|项目用途|解决的问题|
|---|---|---|
|`RLock`|消息推送分布式锁|防止多个实例同时处理同一条消息|
|`RBucket<Boolean>`|MQ 消费记录|防止同一条消息重复处理|
|`RMapCache`|在线状态|保存 `userId → sessionId`，并支持 TTL|

因此不要简单记成：

> Redisson = 分布式锁

更准确的是：

> **Redisson 是 Redis 的高级 Java 客户端，本项目主要利用它实现分布式锁、消费幂等和带 TTL 的在线状态管理。**

### 4.7.4 RLock：分布式锁

`MessagePusher` 消费 `chat-message` 时，需要执行 WebSocket 消息推送。

由于 RocketMQ 可能发生重复投递，或者多个消费者实例同时处理同一条消息，因此需要控制并发。

项目使用：

```text
RLock
  ↓
msg:push:{messageId}
```

流程：

```text
RocketMQ
   ↓
MessagePusher
   ↓
尝试获取 Redis 分布式锁
   ↓
msg:push:{messageId}
   │
   ├── 获取成功
   │      ↓
   │   进入推送临界区
   │      ↓
   │   WebSocket 推送
   │
   └── 获取失败
          ↓
       不同时处理
```

项目实际使用：

```text
tryLock(3, 30, SECONDS)
```

表示：

- 最多等待 3 秒获取锁
    
- 获取成功后，锁最多持有 30 秒
    
- 因为显式传入了 `leaseTime=30s`，所以 **Redisson Watchdog 不会自动续期**
    
- 因此 30 秒必须覆盖业务正常执行时间
    

### 4.7.5 为什么需要分布式锁？

因为微服务可能存在多个 `message` 服务实例：

```text
                 RocketMQ
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
      message-1  message-2  message-3
          │         │         │
          └─────────┼─────────┘
                    ↓
                  Redis
                    │
             msg:push:{id}
```

如果使用 Java 本地锁，例如：

```java
synchronized
```

它只能控制**同一个 JVM 内部**的线程。

如果：

```text
message-1 JVM
message-2 JVM
```

同时处理同一条消息，本地锁无法互相感知。

而 Redis 是多个服务实例都可以访问的共享存储，因此可以利用 Redis 实现跨 JVM 的分布式锁。

所以：

> **分布式锁的核心是：多个服务实例通过一个共享存储协调“谁可以进入临界区”。**

Redisson 则负责把这套 Redis 分布式锁机制封装成：

```java
RLock
```

让业务代码不需要自己实现复杂的 Redis 加锁、续期和解锁逻辑。

### 4.7.6 RLock 和幂等不是一回事

项目的 `MessagePusher` 同时使用：

```text
RLock
+
RBucket
```

两者解决的问题不同。

#### RLock：防并发

```text
同一时刻
多个线程 / 实例
       ↓
只有一个进入临界区
```

解决：

> **“现在有没有其他线程正在处理？”**

#### RBucket：防重复消费

项目使用：

```text
msg:consumed:{messageId}
```

记录消息是否已经处理过。

```text
第一次消费
   ↓
没有 consumed 标记
   ↓
处理消息
   ↓
写入 msg:consumed:{id}

第二次消费
   ↓
发现已经存在 consumed 标记
   ↓
跳过处理
```

解决：

> **“这条消息以前是不是已经处理过？”**

所以：

```text
RLock
 ↓
防并发
 ↓
解决“同时处理”

RBucket
 ↓
防重复
 ↓
解决“以前已经处理过”
```

**锁 ≠ 幂等。**

只加锁不能解决 MQ 重复消费：

```text
第一次消费
 ↓
拿到锁
 ↓
推送
 ↓
释放锁

第二次重复消费
 ↓
再次拿到锁
 ↓
再次推送
```

因此项目需要同时使用锁和消费记录。

### 4.7.7 RMapCache：在线状态

#### RMapCache 是什么？

`RMapCache` 是 Redisson 提供的一种类似 Java `Map` 的 Redis 数据结构，支持为 **每个键值条目（entry）单独设置 TTL**。

在 MiniChat 项目中，`SessionManager` 使用：

```
RMapCache<String, String> onlineUsers;
```

Redis 中对应的逻辑结构为：

```
Redis Map：online:users
│
├── key: "123" → value: "session-A"（TTL：30 秒）
├── key: "456" → value: "session-B"（TTL：30 秒）
└── key: "789" → value: "session-C"（TTL：30 秒）
```

这里需要区分三个概念：

|概念|含义|项目中的例子|
|---|---|---|
|Redis 外层 key|标识整个数据结构|`online:users`|
|Map 内部的 key|标识某个条目|`userId = "123"`|
|value|该条目实际存储的数据|`sessionId = "session-A"`|
|TTL|该条目允许存活的时间|30 秒|

**注意：TTL 不是 value 的内容，而是条目的过期时间设置。**

#### Redis 的两层 key-value 结构

普通 Redis key-value：

```
Redis key                 value
online:user:123    →      "1"
```

这里，`online:user:123` 是一个完整的 Redis key，`"1"` 是它存储的 value。

Redis Hash 则是在一个 Redis key 内部存储多组 field-value：

```
Redis key：online:users
│
├── field: "123" → value: "session-A"
├── field: "456" → value: "session-B"
└── field: "789" → value: "session-C"
```

|层级|名称|示例|
|---|---|---|
|外层|Redis key|`online:users`|
|内层|Hash field|`"123"`、`"456"`、`"789"`|
|内层对应的数据|Hash value|`"session-A"`、`"session-B"`、`"session-C"`|

因此，`field` 就是 Hash 内部的键，`value` 是这个 field 对应的数据。

对于 Redisson 的 `RMapCache`，我们可以用类似的 Map 视角理解：内部 key 对应用户 ID，value 对应会话 ID。但它的底层实现包含额外的过期管理机制，不能简单认为它就是一个原生 Redis Hash。

#### 为什么使用 RMapCache，而不是普通 Redis Hash？

原生 Redis Hash 可以给整个 Hash 设置 TTL：

```
EXPIRE online:users 30
```

这表示整个 `online:users` 数据结构在过期后失效，而不是只让某个用户的条目过期。

原生 Redis Hash **不能通过普通的 `EXPIRE` 命令直接给某个 field 单独设置 TTL**。

假设有三个用户：

```
online:users
├── 123 → session-A
├── 456 → session-B
└── 789 → session-C
```

如果把 TTL 设置在整个 `online:users` 上，那么它控制的是整个 Hash 的有效期，而不是每个用户各自的有效期。

如果每次用户心跳都刷新整个 Hash 的 TTL，就会出现一个问题：用户 123 的心跳可能让整个 Hash 的有效期延长，其他用户的条目也会随之继续保留。

而在线状态需要满足：

- 用户 123 的心跳只续期用户 123 的条目。
    
- 用户 456 没有心跳时，其条目可以独立过期。
    
- 一个用户的在线状态变化，不应影响其他用户条目的过期时间。
    

`RMapCache` 提供的 entry 级 TTL 正好适合这种需求：

```
RMapCache：online:users
├── 123 → session-A（独立 TTL：30 秒）
├── 456 → session-B（独立 TTL：30 秒）
└── 789 → session-C（独立 TTL：30 秒）
```

用户 123 续期，不会因此自动续期用户 456 和 789 的条目。

#### RMapCache 是专门为多用户 TTL 设计的吗？

不是。它是一个通用数据结构，并不专属于在线用户管理。

只要一个 Map 中的不同条目需要独立设置过期时间，就可以考虑使用 `RMapCache`，例如：

- 在线用户与会话映射。
    
- 需要独立过期的临时缓存条目。
    
- 具有不同有效期的业务状态数据。
    

它的优势不是简单地“可以存储多个用户”，而是 **多个条目可以分别管理过期时间**。

#### MiniChat 中的在线状态维护流程

```
客户端建立 WebSocket 连接
        ↓
SessionManager 保存本地 Session
        ↓
更新 Redis 的 online:users
        ↓
userId → sessionId，设置约 30 秒 TTL
        ↓
客户端定期发送心跳
        ↓
服务端刷新对应用户条目的有效期
        ↓
持续收到心跳：在线状态持续有效
长时间没有心跳：条目过期，在线状态失效
```

注意：TTL 到期不代表应用一定会在到期的那个瞬间完成所有清理。业务代码仍需要考虑连接关闭、心跳超时及本地 Session 清理等情况。

此外，Redis 中的在线状态与本机内存中的 `WebSocketSession` 不是同一份数据：

- Redis：记录用户 ID 与会话 ID 的映射。
    
- 本地 `SessionManager`：保存当前服务实例可使用的 WebSocket Session。
    

因此，Redis 中存在映射，并不绝对保证当前服务实例仍持有有效的 WebSocket 连接。

#### 面试总结

`RMapCache` 是 Redisson 提供的支持 entry 级 TTL 的 **Map 数据结构**。MiniChat 使用它维护 Redis 中的在线用户映射，将 **`userId` 映射到 `sessionId`**，并为每个条目设置约 30 秒的有效期。客户端持续发送心跳时，服务端续期对应条目；长时间没有心跳时，条目过期，在线状态随之失效。

相比给整个 Redis Hash 设置 TTL，**`RMapCache` 能让每个用户的映射独立过期**，避免一个用户的心跳影响其他用户的在线状态。

### 4.7.8 项目中两种 Redis 客户端的分工

|客户端|使用场景|原因|
|---|---|---|
|`StringRedisTemplate`|短信验证码、JWT 黑名单|简单 KV 操作即可|
|`Redisson`|分布式锁、MQ 幂等、在线状态|需要 `RLock`、`RBucket`、`RMapCache` 等高级对象|

所以项目并不是：

```text
Redis 和 Redisson 二选一
```

而是：

```text
                     Redis
                       │
          ┌────────────┴────────────┐
          │                         │
StringRedisTemplate              Redisson
          │                         │
     简单 KV 操作                高级 Redis 操作
          │                         │
     ┌────┴────┐             ┌─────┼─────┐
     ↓         ↓             ↓     ↓     ↓
   验证码    Token黑名单     RLock RBucket RMapCache
                              │     │      │
                              ↓     ↓      ↓
                           分布式锁 幂等   在线状态
```

### 4.7.9 面试怎么说？

如果面试官问：

> **“你们为什么使用 Redisson？它和 Redis 是什么关系？”**

可以回答：

> **Redis 是我们项目使用的缓存和共享存储，Redisson 是基于 Redis 的 Java 客户端和高级封装。我们在 message 服务中使用 Redisson，主要有三个场景：第一，通过 `RLock` 实现消息推送的分布式锁，避免多个实例同时处理同一条消息；第二，通过 `RBucket` 保存 MQ 消费记录，实现重复消费的幂等控制；第三，通过 `RMapCache` 保存用户在线状态，并利用 TTL 实现心跳过期。对于短信验证码和 Token 黑名单这种简单 KV 场景，则直接使用 `StringRedisTemplate`。**

### 一句话记忆

> **Redis 是底层服务，Redisson 是 Java 高级客户端；本项目用 Redisson 的 `RLock` 做分布式锁、`RBucket` 做消费幂等、`RMapCache` 做带 TTL 的在线状态管理。**

# 5. 后端模块详解

## 5.1 minichat-common

| 类 | 作用 |
|---|---|
| `JacksonConfig` | 注册 `Long → String` 序列化器，**防止前端 JS 雪花 ID 精度丢失**（需 `scanBasePackages="com.minichat"` 才生效） |
| `Result<T>` | 统一响应 `{code, msg, data}`，成功码 `200` |
| `AuthConstant` | `SMS_CODE_EXPIRE_MINUTES=5`、`SMS_REDIS_KEY_PREFIX="sms:"` |
| `InternalUserResponse` | 内部 RPC 用户资料（裸对象，不包 Result） |
| `InternalGroupResponse` | 内部 RPC 群成员 `{userId, nickname, avatar}` |

## 5.2 minichat-gateway :8080

**路由规则**

| 目标服务 | 路径 |
|---|---|
| minichat-user | `/auth/**` `/user/**` `/friend/**` `/group/**` `/file/**` |
| minichat-message | `/message/**` `/conversation/**` `/oss/**` `/chat/**` `/ws` |

**CORS（跨域资源共享）**：当前配置 `allowed-origin-patterns: "*"`，表示允许任意 Origin 发起浏览器跨域请求。这通常方便本地开发，但生产环境一般应限制为实际前端域名。CORS 是浏览器跨域访问控制，不等于身份认证，也不会替代 JWT 鉴权。

**Knife4j 网关聚合**：用户服务与消息服务分别生成 OpenAPI 文档，网关把它们聚合到统一入口，方便开发人员集中查看和调试：
- 用户服务文档：`/user/v3/api-docs`
- 消息服务文档：`/message/v3/api-docs`

CORS 解决跨域访问问题；Knife4j 聚合解决微服务接口文档分散的问题，两者是不同功能。

> gateway 未依赖 common、未配 `scanBasePackages`，因此 `JacksonConfig` 在网关**不生效**（WebFlux 场景也不需要）。

## 5.3 minichat-user :8081

#### 接口清单

**AuthController** `/auth`

| 方法 | 路径 | 功能 |
|---|---|---|
| POST | `/auth/sms/code` | 发送短信验证码（60 秒防重，Redis TTL 5 分钟） |
| POST | `/auth/register` | 注册，返回 token |
| POST | `/auth/login` | 密码登录，返回 token |
| POST | `/auth/logout` | 退出，token 加入 Redis 黑名单 |

**UserController** `/user`

| 方法 | 路径 | 功能 |
|---|---|---|
| GET | `/user/{id}` | 查询用户资料 |
| PUT | `/user/{id}` | 修改资料（只能改自己） |
| PUT | `/user/password` | 修改密码 |
| POST | `/user/avatar` | 上传头像到 OSS |
| DELETE | `/user/{id}` | 注销账号（软删除 status=3） |

**FriendController** `/friend`

| 方法 | 路径 | 功能 |
|---|---|---|
| POST | `/friend/request` | 发送好友请求 |
| POST | `/friend/request/{id}/accept` | 同意 |
| POST | `/friend/request/{id}/reject` | 拒绝 |
| GET | `/friend/requests` | 待处理请求列表 |
| GET | `/friend/list` | 好友列表 |
| DELETE | `/friend/{friendId}` | 删除好友（**单向**） |
| POST | `/friend/{friendId}/block` | 拉黑/取消拉黑（开关） |
| GET | `/friend/search?keyword=` | 搜索用户 |

**GroupController** `/group`

| 方法 | 路径 | 功能 |
|---|---|---|
| POST | `/group` | 创建群（2~200 人） |
| POST | `/group/{groupId}/quit` | 退群（**群主不能退**） |
| POST | `/group/{groupId}/kick/{memberId}` | 踢人（群主） |
| GET | `/group/list` | 我的群列表 |
| GET | `/group/{groupId}/members` | 群成员列表 |
| PUT | `/group/{groupId}` | 改群名/公告（群主） |
| DELETE | `/group/{groupId}` | 解散群（群主） |

**内部接口**（返回裸对象，网关 403 拦截，仅 Feign 可调用）

| 方法 | 路径 | 功能 |
|---|---|---|
| GET | `/internal/user/{id}` | 内部查用户资料 |
| GET | `/internal/group/{groupId}/members` | 内部查群成员（**不做权限校验**） |

### 5.3.1 核心业务规则

- **注册**：手机查重 → 验短信码 → BCrypt 加密 → 默认昵称 `"用户"+phone.substring(7)` → 同步 ES
- **登录**：统一错误提示「手机号或密码错误」防枚举；`status==3` 提示已注销
- **注销**（事务）：`status=3` → 删 ES 文档 → 清好友关系+请求 → 退所有群（群主则连带解散）
- **好友请求**：五重校验（不能加自己 / 对方存在 / 未是好友 / 无待处理请求）
- **同意好友**：插入**双向**两条 `Friend` 记录
- **删好友**：单向，只把 A→B 置 `status=0`
- **搜索用户**：三级 —— ES 昵称 match → ES 手机号精确 → **ES 为空时** MySQL LIKE 兜底（ES standard 分词对中文效果差）
- **改密码**：验旧密码 → 新密码不能与旧密码相同 → BCrypt → updateById

### 5.3.2 短信验证码

项目使用 **Redis 临时保存短信验证码**，用于注册等需要短信验证的场景。

整体流程：

```text
用户输入手机号
      ↓
请求发送验证码
      ↓
生成验证码
      ↓
保存到 Redis
      ↓
发送短信
      ↓
用户收到验证码
      ↓
提交手机号 + 验证码
      ↓
从 Redis 获取验证码进行校验
      ↓
校验成功 → 删除验证码 → 验证通过
```

#### ① 验证码存在哪里？

验证码保存在 Redis 中，例如：

```text
Key：
sms:{phone}

Value：
验证码

TTL：
5 分钟
```

例如：

```text
sms:13417794546 → 123456
```

验证码只在短时间内有效，5 分钟后 Redis 会自动删除。

因此不需要把短信验证码长期保存到数据库。

#### ② 为什么还需要 60 秒防重？

如果用户不断点击“获取验证码”，可能导致短时间内发送大量短信。

因此项目使用一个独立的 Redis Key 限制验证码发送频率：

```text
sms:lock:{phone}
```

发送验证码时：

```text
请求发送验证码
      ↓
检查 sms:lock:{phone}
      ↓
已经存在？
   ├── 是 → 拒绝重复发送
   │
   └── 否 → 生成并发送验证码
                ↓
           设置 60 秒锁
```

也就是说：

> **同一个手机号 60 秒内不能重复发送验证码。**

这里需要区分两个 TTL：

|Redis Key|作用|TTL|
|---|---|--:|
|`sms:{phone}`|保存实际验证码|5 分钟|
|`sms:lock:{phone}`|防止重复发送|60 秒|

#### ③ 为什么验证码校验成功后要删除？

验证码是**一次性凭证**。

例如：

```text
用户提交验证码
      ↓
Redis 查询 sms:{phone}
      ↓
验证码正确
      ↓
删除 sms:{phone}
      ↓
验证成功
```

删除之后，同一个验证码就不能再次使用。

因此：

> **5 分钟是验证码的最大有效时间，一旦成功使用，会提前失效。**

#### ④ 为什么验证码不能直接从接口返回？

发送验证码的接口不能直接返回验证码，例如：

```text
❌
{
    "code": "123456"
}
```

否则调用接口的人直接就能拿到验证码，短信验证就失去了意义。

正确流程是：

```text
服务端
  ↓
生成验证码
  ↓
保存 Redis
  ↓
发送到用户手机
  ↓
接口只返回“发送成功”等结果
```

用户需要从自己的手机短信中获取验证码，再提交给服务端验证。

### ⑤ 整体流程

把**防重复发送、验证码有效期和一次性使用**放在一起：

```text
                    发送验证码
                        │
                        ▼
                  检查 60 秒防重锁
                   /          \
                 有             无
                 │              │
                 ▼              ▼
              拒绝发送       生成验证码
                                │
                                ▼
                         Redis 保存验证码
                          sms:{phone}
                          TTL = 5 分钟
                                │
                                ▼
                             发送短信
                                │
                                ▼
                          用户提交验证码
                                │
                                ▼
                         Redis 查询验证码
                          /          \
                       错误            正确
                        │               │
                        ▼               ▼
                     验证失败        删除验证码
                                        │
                                        ▼
                                     验证成功
```

#### 面试怎么说？

如果面试官问：

> **“你们项目的短信验证码是怎么实现的？”**

可以回答：

> 我们使用 Redis 临时保存短信验证码。验证码以手机号作为标识保存，并设置 5 分钟过期时间，同时通过独立的 Redis Key 设置 60 秒防重锁，避免同一个手机号频繁发送验证码。用户提交验证码后，服务端从 Redis 中进行校验，校验成功后立即删除验证码，使验证码只能使用一次。验证码本身不会通过接口返回，而是通过短信发送给用户。

### 5.3.3 头像上传 8 步校验

空文件 → Content-Type 必须 `image/` → ≤2MB → **魔数校验**（JPEG `FF D8 FF` / PNG `89 50 4E 47` / GIF `47 49 46 38`，防伪造）→ 路径 `avatars/{userId}/{UUID}{ext}` → 传 OSS → 拼 URL → 更新 user 并同步 ES

### 5.3.4 实体与配置

| 实体 | 表名 | 主键策略 | status 含义 |
|---|---|---|---|
| `User` | `tb_user` | 雪花 ASSIGN_ID | 1正常 2封禁 3注销 |
| `Friend` | `tb_friend` | AUTO | 1正常 0删除 2拉黑 |
| `FriendRequest` | `tb_friend_request` | AUTO | 0待处理 1同意 2拒绝 3忽略 |
| `Group` | `tb_group` | 雪花 | 1正常 2已解散 |
| `GroupMember` | `tb_group_member` | AUTO | 1正常 2已退群 3被踢出 |
| `UserDocument` | ES `user` 索引 | — | — |

**关键配置**：`${MYSQL_DB:minichat_user_db}`、`${MYSQL_PASSWORD}`（无默认值，不配启动失败）、`${JWT_SECRET}`、`spring.elasticsearch.uris`

### 5.3.5 测试（7 个，约 1407 行）

#### 测试：Mockito 单元测试

项目使用 **JUnit + Mockito** 对核心 Service 进行单元测试，共 7 个测试类，约 1407 行。

测试类包括：

- `AuthServiceTest`
    
- `UserServiceTest`
    
- `FriendServiceTest`
    
- `GroupServiceTest`
    
- `FileServiceTest`
    
- `UserIndexServiceTest`
    
- `JwtServiceTest`
    

#### 什么是“纯 Mockito 单测”？

这里的“纯”主要指：

> **测试时不启动 Spring 容器，只测试目标类本身的业务逻辑。**

例如要测试：

```text
UserService
```

但是 `UserService` 可能依赖：

```text
UserService
   ├── UserMapper
   ├── RedisService
   └── 其他 Service
```

如果直接运行真实环境，这些依赖都需要初始化。

Mockito 可以把这些依赖替换成 **Mock 对象**：

```text
             UserServiceTest
                   │
                   ▼
              UserService
              /    |     \
             /     |      \
            ▼      ▼       ▼
       MockMapper MockRedis MockService
```

测试时只关注：

> **UserService 自己的代码逻辑是否正确。**

而不是去测试 MySQL、Redis 或其他 Service 本身。

#### 为什么不启动 Spring？

如果启动 Spring：

```text
测试
 ↓
启动 Spring Boot
 ↓
加载 Bean
 ↓
连接数据库 / Redis / 其他组件
 ↓
执行测试
```

测试会比较慢，而且依赖真实的运行环境。

而纯 Mockito 单测：

```text
测试
 ↓
直接创建被测试对象
 ↓
Mock 外部依赖
 ↓
执行方法
 ↓
验证结果
```

启动速度更快，也更容易定位问题。

#### 一个简单例子

假设要测试：

```java
userService.getUserById(123);
```

但是 `UserService` 需要通过 `UserMapper` 查询数据库。

测试时不真正查询 MySQL，而是告诉 Mockito：

```text
当调用 userMapper.selectById(123)
时，就假装数据库返回 User 对象
```

然后测试：

```text
UserService
     ↓
调用 Mock UserMapper
     ↓
得到预设的数据
     ↓
执行自己的业务逻辑
     ↓
验证最终结果是否正确
```

所以这里测试的是：

> **UserService 的业务逻辑是否正确。**

而不是：

> **MySQL 能不能正常查询。**

#### 这种测试主要测什么？

在这个项目中，Mockito 单测主要用于验证核心 Service 的业务逻辑，例如：

```text
输入参数
   ↓
Service 业务逻辑
   ↓
Mock 外部依赖
   ↓
返回结果 / 异常
```

重点可以测试：

- 正常业务流程
    
- 参数校验
    
- 不同条件下的分支逻辑
    
- 异常情况
    
- 外部依赖是否被正确调用
    

#### 和集成测试的区别

|类型|Spring 容器|真实数据库/Redis|主要目的|
|---|---|---|---|
|**Mockito 单元测试**|❌ 不启动|❌ 通常不使用|测试单个类的业务逻辑|
|**Spring 集成测试**|✅ 启动|可以使用|测试多个组件能否正确协作|
|**手工测试**|通常运行完整项目|✅|从真实使用场景验证系统|

所以你这个项目里的：

> **“纯 Mockito 单测”**

重点就是：

> **把 Service 的依赖全部 Mock 掉，不启动 Spring，只验证 Service 自身的业务逻辑。**

面试时可以简单说：

> **项目中使用 JUnit 和 Mockito 对核心 Service 编写单元测试。测试时不启动 Spring 容器，而是通过 Mock 隔离数据库、Redis 等外部依赖，重点验证 Service 的业务逻辑、异常处理和不同分支。**

### 5.4 minichat-message :8082

#### 接口清单

**MessageController** `/message`

| 方法 | 路径 | 功能 |
|---|---|---|
| GET | `/message/history/all` | 全部历史（调试用） |
| GET | `/message/history/private` | 私聊历史（`targetId` + 可选时间范围） |
| GET | `/message/history/group` | 群聊历史 |

**ChatSearchController** `/chat`

| 方法 | 路径 | 功能 |
|---|---|---|
| GET | `/chat/history/content` | 分页历史/搜索 |

参数：`conversationId` / `fromId` / `toId` / `keyword` / `sortField`(默认 createTime) / `direction`(默认 desc) / `page`(默认 0) / `size`(默认 20)

> 现行走 **MySQL + MyBatis-Plus 分页**；基于 ES 的旧实现已整段注释，`ChatSearchService` 目前**未被任何 Controller 调用**。

**OssController** `/oss`

| 方法 | 路径 | 功能 |
|---|---|---|
| POST | `/oss/upload` | 上传文件，返回**签名 URL（有效期 300 秒）** |

对象 key 格式：`chat/{image|video|voice|other}/{yyyy/MM/dd}/{UUID}{后缀}`

#### WebSocket 组件

| 类 | 作用 |
|---|---|
| `AuthHandshakeInterceptor` | 握手时取网关注入的 `userId` header，null 则拒绝握手 |
| `ChatWebSocketHandler` | 分发 `type`：`1`发消息 / `2`ACK（空实现） / `3`心跳续期 |
| `SessionManager` | 三级会话存储 + 自愈 + `sendToUser` |

**单端登录语义**：新连接建立时若该 userId 已在线，先 `removeSession` 踢掉旧连接。

#### 消息构造

`messageType`：`1`文本 `TextContent{text}` / `2`图片 `ImageContent{url,...}` / `3`视频 / `4`语音，其他抛「暂不支持的消息类型」。ID 用 Hutool 雪花 `IdUtil.getSnowflakeNextId()`。

会话（Conversation）相关代码已整段注释，**会话是纯前端概念，后端无此表**。

#### 实体

| 类 | 映射 | 说明 |
|---|---|---|
| `ChatMessage` | `@TableName("tb_message")` | 消息主表 |
| `MessageArchive` | 无 ORM 注解 | 仅用于 Parquet 序列化（时间为 String） |
| `ChatMessageDocument` | ES `chat_message` 索引 | content 为 Text（`standard` 分词器） |

## 6. 数据库表结构

**`minichat_user_db`**（`utf8mb4_unicode_ci`，InnoDB）共 5 张表：

### tb_user
| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | bigint | 雪花 ID |
| `phone` | varchar(20) | 登录手机号，**UNIQUE** |
| `password_hash` | varchar(255) | BCrypt |
| `nickname` | varchar(50) | 昵称 |
| `avatar` / `signature` | varchar(255) | 头像 / 签名 |
| `gender` | tinyint | 0未知 1男 2女 |
| `status` | tinyint | 1正常 2封禁 3注销 |
| `last_login_time` / `last_login_ip` | | |
| `create_time` / `update_time` | datetime | |

索引：`uk_phone(phone)`、`idx_create_time(create_time)`

### tb_friend
`id`(AI) / `user_id` / `friend_id` / `remark` / `status`(1正常 0删除 2拉黑) / `create_time`
索引：`uk_user_friend(user_id, friend_id)`

### tb_friend_request
`id`(AI) / `from_id` / `to_id` / `remark` / `status`(0待处理 1同意 2拒绝 3忽略) / `create_time` / `update_time`
索引：`idx_to_id_status(to_id, status)`

### tb_group
`id`(雪花) / `name` / `owner_id` / `notice` / `max_members`(默认200) / `status`(1正常 2已解散) / `create_time` / `update_time`
索引：`idx_owner_id(owner_id)`

### tb_group_member
`id`(AI) / `group_id` / `user_id` / `role`(0成员 1管理员 2群主) / `alias` / `status`(1正常 2已退群 3被踢) / `join_time` / `quit_time`
索引：`uk_group_user(group_id, user_id)`、`idx_user_id(user_id)`

> ⚠️ **消息表 `tb_message` 无建表 SQL**（见[已知问题](#10-已知问题与技术债)）

# 7 部署配置

## 7.1 docker-compose 服务清单

| 服务 | 容器名 | 端口 |
|---|---|---|
| Nacos | `minichat-nacos` | 8848 / 9848 / 9849 |
| MySQL 8 | `minichat-mysql` | 3306 |
| Redis 7 | `minichat-redis` | 6379 |
| RocketMQ NameServer | `rmqnamesrv` | 9876 |
| RocketMQ Broker | `rmqbroker` | 10911 / 10909 |
| Elasticsearch | `minichat-elasticsearch` | 9200 |
| Kibana | `minichat-kibana` | 5601 |
| Nginx | `minichat-nginx` | 80 |

单网络 `minichat-network`，全部 `restart: always`。**全部使用 bind mount，无 named volume**。

## 7.2 数据持久化

`.env` 中 `DATA_VOLUME_BASE=./data`，MySQL bind mount 到 `deploy/data/mysql/` → **容器重建数据不丢，只要不删 `deploy/data/`**。

> Windows 下 bind mount 可用，遇到 ES Permission denied / RocketMQ NPE 先**对比可跑通的分支**，别急着改 docker-compose.yml。

## 7.3 环境变量

`.env` 关键项：`MYSQL_ROOT_PASSWORD=123456`、`MYSQL_DATABASE=minichat_db`、`MYSQL_PORT=3306`、`REDIS_PASSWORD=123456`、`JWT_SECRET=MiniChatSecretKey2026...`

ECS/SMS 的 `ALIYUN_OSS_*` / `ALIYUN_SMS_*` 由 `server/start.ps1` 读取并注入后端进程。

## 7.4 Nacos 配置中心

Nacos 同时承担**服务注册发现**与**配置管理**两个职责。

各服务通过 `spring.config.import` 拉取远程配置：

```yaml
spring:
  config:
    import:
      - optional:nacos:minichat-user.yaml
```

| Data ID | 服务 |
|---|---|
| `minichat-gateway.yaml` | minichat-gateway |
| `minichat-user.yaml` | minichat-user |
| `minichat-message.yaml` | minichat-message |

配置模板见 `deploy/nacos/`，导入方式见 `deploy/nacos/README.md`。

> `optional:` 前缀是关键 —— Nacos 不可用或 Data ID 不存在时服务仍按本地默认值启动，
> 避免配置中心故障导致整个集群起不来。

**动态刷新**：`refresh-enabled: true` 已开启。`@Value` 注入需配合 `@RefreshScope`，
`@ConfigurationProperties` 绑定默认自动刷新。

## 7.5 敏感配置与配置中心

项目中的数据库密码、JWT 密钥、OSS/SMS 等敏感信息不直接写在代码或配置文件中，而是通过 **环境变量和 Nacos 配置中心**进行管理。

核心目标是：

> **敏感信息不硬编码，同时让不同环境可以使用不同配置。**

### 7.5.1 为什么不能直接把密码写进 `application.yml`？

例如不推荐：

```yaml
password: 123456
jwt-secret: abcdef123456
```

因为配置文件可能进入 Git 仓库，一旦泄露，数据库密码、JWT 密钥等敏感信息也可能暴露。

因此项目采用环境变量占位符：

```yaml
password: ${DB_PASSWORD:123456}
jwt-secret: ${JWT_SECRET:default-secret}
```

这里的格式：

```text
${环境变量名:默认值}
```

表示：

```text
优先使用环境变量
        ↓
如果环境变量不存在
        ↓
使用默认值
```

例如：

```yaml
jwt-secret: ${JWT_SECRET:dev-secret}
```

如果系统设置了：

```text
JWT_SECRET=真正的密钥
```

那么最终使用环境变量中的密钥。

### 7.5.2 Nacos 配置中心

项目的三个微服务都接入了 **Nacos 配置中心**。

Nacos 主要负责：

- 集中管理服务配置
    
- 不同服务使用各自的配置
    
- 不需要把大量配置全部写在本地 `application.yml`
    
- 方便不同环境使用不同配置
    

可以理解为：

```text
                  Nacos
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Gateway     User     Message
       配置        配置       配置
```

### 7.5.3 为什么使用 `optional:`？

项目使用 `optional:` 方式加载 Nacos 配置。

它解决的是：

> **如果 Nacos 暂时不可用，服务还能不能启动？**

例如：

```text
服务启动
   ↓
尝试加载 Nacos 配置
   ↓
Nacos 可用？
  ├── 是 → 加载 Nacos 配置
  │
  └── 否 → 不强制阻止启动
              ↓
         使用本地配置/默认值
```

因此，Nacos 在这里不是一个“服务启动的绝对前提”。

这是一种**降级策略**：

> **配置中心不可用时，服务仍可以使用本地配置继续启动。**

### 7.5.4 敏感配置应该放在哪里？

不同环境可以采用不同方式。

#### 开发环境

可以使用：

```text
application.yml
      ↓
${ENV:默认值}
```

方便本地开发和调试。

#### 生产环境

敏感信息不应该使用简单的默认值，而应该通过：

```text
环境变量
    或
Nacos 中的安全配置
```

注入，例如：

```text
JWT Secret
OSS AK/SK
SMS AK/SK
数据库密码
```

整体可以理解为：

```text
开发环境
   ↓
本地配置 + 环境变量

生产环境
   ↓
Nacos 配置 / 环境变量
   ↓
敏感信息安全注入
```

### 7.5.5 整体配置流程

```text
                服务启动
                    │
                    ▼
            加载本地配置文件
                    │
                    ▼
             尝试加载 Nacos
              /          \
           成功            失败
            │               │
            ▼               ▼
       使用 Nacos 配置    使用本地配置
            │               │
            └───────┬───────┘
                    ▼
              环境变量覆盖
                    │
                    ▼
                最终配置
```

这里需要注意：

> **具体配置的优先级要以 Spring Boot / Spring Cloud 的实际配置加载规则为准，不建议简单地把所有配置概括成“Nacos 一定高于环境变量”。**

#### 面试怎么说？

如果面试官问：

> **“你们项目是怎么管理敏感配置的？”**

可以回答：

> 我们没有把数据库密码、JWT 密钥以及 OSS、短信服务的 AK/SK 等敏感信息直接硬编码到项目中，而是使用环境变量和 Nacos 配置中心进行管理。本地配置通过 `${ENV:默认值}` 的方式读取环境变量，三个微服务都接入了 Nacos。配置加载使用 optional 策略，因此 Nacos 不可用时不会直接导致服务无法启动，可以降级使用本地配置。生产环境则应该通过环境变量或安全的 Nacos 配置注入敏感信息。**