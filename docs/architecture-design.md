# PicoClaw 架构设计文档

## 概述

PicoClaw 是一个用 Go 语言编写的超轻量级个人 AI 助手，专为低资源硬件设计（<10MB 内存，<1秒启动时间）。

## 核心设计原则

1. **极简主义** - 最少依赖，单一二进制
2. **模块化** - 通过消息总线解耦组件
3. **可扩展** - 插件化渠道和 Provider
4. **AI 驱动** - 95% 代码由 AI 生成

## 分层架构

### 1. 命令行层 (cmd/)

```
cmd/
├── picoclaw/              # 主程序入口
│   ├── main.go
│   └── internal/
│       ├── agent/          # Agent CLI 命令
│       ├── gateway/       # Gateway 服务
│       ├── cron/           # 定时任务
│       ├── onboard/       # 初始化引导
│       └── status/        # 状态查看
├── picoclaw-launcher/     # 启动器
└── picoclaw-launcher-tui/ # TUI 界面
```

### 2. 核心包 (pkg/)

#### 2.1 消息总线 (pkg/bus/)

消息总线是系统核心，负责各组件间的通信：

- **Inbound**: 从各渠道接收用户消息
- **Outbound**: 向用户发送响应
- 支持发布/订阅模式

```go
type MessageBus struct {
    inbound  chan InboundMessage
    outbound chan OutboundMessage
}
```

#### 2.2 Agent 引擎 (pkg/agent/)

核心 AI 处理单元：

- **AgentLoop**: 主循环，处理消息
- **AgentRegistry**: Agent 注册与管理
- **AgentInstance**: 单个 Agent 实例
- **Session**: 会话历史管理
- **Context**: 上下文窗口管理

```go
type AgentLoop struct {
    bus      *bus.MessageBus
    cfg      *config.Config
    registry *AgentRegistry
    state    *state.Manager
}
```

#### 2.3 Provider 系统 (pkg/providers/)

支持多种 LLM 提供商：

| Provider | Protocol | Features |
|----------|----------|----------|
| OpenAI | OpenAI Compatible | GPT 系列 |
| Anthropic | Anthropic | Claude 系列 |
| Ollama | OpenAI Compatible | 本地模型 |
| 智谱 GLM | OpenAI Compatible | GLM 系列 |
| DeepSeek | OpenAI Compatible | DeepSeek 系列 |
| Groq | OpenAI Compatible | 快速推理 |

#### 2.4 通讯渠道 (pkg/channels/)

多渠道接入支持：

**即时通讯**:
- Telegram
- Discord
- WhatsApp
- 飞书
- QQ
- 企业微信
- 钉钉
- Line
- Slack

**物联网设备**:
- MaixCam
- Pico 设备

#### 2.5 工具系统 (pkg/tools/)

Agent 可调用的工具：

**Web 工具**:
- Brave Search
- DuckDuckGo
- Tavily
- Perplexity
- Web Fetch

**硬件工具**:
- I2C 读写
- SPI 通信

**消息工具**:
- 发送消息到各渠道

**技能工具**:
- 发现技能 (Find Skills)
- 安装技能 (Install Skill)
- 子 Agent Spawn

### 3. 基础设施层

#### 3.1 配置管理 (pkg/config/)

```json
{
  "model_list": [...],
  "agents": {...},
  "channels": {...},
  "tools": {...},
  "heartbeat": {...}
}
```

#### 3.2 状态管理 (pkg/state/)

- 会话状态持久化
- Agent 状态跟踪

#### 3.3 日志系统 (pkg/logger/)

- 结构化日志
- 分级输出 (debug, info, warn, error)

#### 3.4 定时任务 (pkg/cron/)

- Cron 表达式支持
- 定时提醒
- 周期性任务

#### 3.5 心跳检测 (pkg/heartbeat/)

- 定期健康检查
- 状态上报

## 数据流

```
用户消息
    ↓
Channel (Telegram/Discord/...)
    ↓
Gateway
    ↓
MessageBus.PublishInbound()
    ↓
AgentLoop.ConsumeInbound()
    ↓
┌─────────────────────────────────────┐
│  1. 检查命令 (/show, /list, /switch)│
│  2. 加载会话历史                     │
│  3. 调用 LLM                        │
│  4. 执行工具调用                     │
│  5. 循环直到完成                     │
└─────────────────────────────────────┘
    ↓
MessageBus.PublishOutbound()
    ↓
Channel (发送响应)
    ↓
用户收到回复
```

## 关键特性

### 内存优化

- **<10MB 内存**: 比 OpenClaw 减少 99%
- **上下文压缩**: 自动摘要长对话
- **高效序列化**: JSON 优化

### 启动速度

- **<1秒启动**: 在 0.6GHz 单核上
- **无外部依赖**: 静态编译
- **延迟加载**: 按需加载模块

### 多平台支持

- **架构**: x86_64, ARM64, RISC-V
- **操作系统**: Linux, Android (Termux)
- **硬件**: 
  - Raspberry Pi ($35)
  - LicheeRV Nano ($9.9)
  - NanoKVM ($30-50)
  - MaixCAM ($50)
  - 旧 Android 手机

### Provider 负载均衡

支持同一模型配置多个端点，自动轮询：

```json
{
  "model_list": [
    {"model": "openai/gpt-5.2", "api_base": "https://api1.com/v1"},
    {"model": "openai/gpt-5.2", "api_base": "https://api2.com/v1"}
  ]
}
```

### Fallback 机制

当主 Provider 失败时自动切换：

```go
type FallbackChain struct {
    providers []LLMProvider
    cooldown  *CooldownTracker
}
```

## 目录结构总览

```
picoclaw/
├── cmd/                    # 命令行入口
│   ├── picoclaw/          # 主程序
│   ├── picoclaw-launcher/ # 启动器
│   └── picoclaw-launcher-tui/ # TUI
├── pkg/                    # 核心包
│   ├── agent/             # Agent 引擎
│   ├── auth/              # 认证
│   ├── bus/               # 消息总线
│   ├── channels/          # 通讯渠道
│   ├── config/            # 配置
│   ├── constants/         # 常量
│   ├── cron/              # 定时任务
│   ├── devices/           # 设备
│   ├── health/            # 健康检查
│   ├── heartbeat/         # 心跳
│   ├── identity/          # 身份
│   ├── logger/            # 日志
│   ├── media/             # 媒体处理
│   ├── migrate/           # 数据迁移
│   ├── providers/         # LLM 提供商
│   ├── routing/           # 路由
│   ├── session/           # 会话
│   ├── skills/            # 技能
│   ├── state/             # 状态
│   ├── tools/             # 工具
│   ├── utils/             # 工具函数
│   └── voice/             # 语音
├── config/                # 配置示例
├── docker/                # Docker 配置
├── docs/                  # 文档
└── workspace/            # 工作空间
```

## 扩展开发

### 添加新渠道

1. 实现 `Channel` 接口
2. 注册到 channel manager
3. 配置 enable 开关

### 添加新 Provider

1. 实现 `LLMProvider` 接口
2. 注册到 provider factory
3. 在配置中添加模型

### 添加新工具

1. 实现 `Tool` 接口
2. 注册到 agent tools
3. 定义 tool schema

## 性能基准

| 指标 | PicoClaw | OpenClaw | NanoBot |
|------|----------|----------|---------|
| 内存 | <10MB | >1GB | >100MB |
| 启动 | <1s | >500s | >30s |
| 成本 | $10 | $599 | $50 |

## 总结

PicoClaw 通过极简的设计实现了在超低资源硬件上运行 AI 助手的目标。其核心优势在于：

1. **消息总线架构** - 解耦组件，易于扩展
2. **Provider 抽象** - 灵活支持多种 LLM
3. **插件化渠道** - 丰富的接入方式
4. **Go 语言** - 高效、静态编译、单二进制
