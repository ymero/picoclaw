# PicoClaw 架构图

## 系统架构概览

```mermaid
flowchart TB
    subgraph 用户入口层
        CLI[CLI 命令行<br/>picoclaw agent/gateway]
        TUI[TUI 界面<br/>launcher-tui]
        Channels[通讯渠道]
    end

    subgraph 核心服务层
        Gateway[Gateway 网关]
        AgentLoop[Agent 循环处理器]
        MessageBus[消息总线]
    end

    subgraph Agent 引擎
        Registry[Agent 注册表]
        Instance[Agent 实例]
        Session[会话管理]
        Context[上下文管理]
    end

    subgraph 工具层
        Tools[工具系统]
        WebSearch[网页搜索]
        WebFetch[网页抓取]
        Message[消息发送]
        I2C[I2C 硬件]
        SPI[SPI 硬件]
        Skills[技能系统]
        Spawn[子Agent]
    end

    subgraph LLM 提供商
        Providers[提供商工厂]
        OpenAI[OpenAI 兼容]
        Anthropic[Anthropic Claude]
        Ollama[Ollama 本地]
        Zhipu[智谱 GLM]
        DeepSeek[DeepSeek]
    end

    subgraph 通讯渠道
        Telegram[Telegram]
        Discord[Discord]
        WhatsApp[WhatsApp]
        Feishu[飞书]
        QQ[QQ]
        WeCom[企业微信]
        DingTalk[钉钉]
        Line[LINE]
        Slack[Slack]
        MaixCam[MaixCam]
        Pico[Pico设备]
    end

    subgraph 基础设施
        Config[配置管理]
        State[状态管理]
        Logger[日志系统]
        Cron[定时任务]
        Heartbeat[心跳检测]
        Health[健康检查]
        Auth[认证授权]
        Media[媒体处理]
        Migrate[数据迁移]
        Identity[身份管理]
        Routing[消息路由]
    end

    User -.->|发送消息| Channels
    Channels -->|Inbound| Gateway
    Gateway --> MessageBus
    MessageBus --> AgentLoop
    AgentLoop --> Registry
    Registry --> Instance
    Instance --> Session
    Instance --> Context
    Instance --> Tools
    Tools -->|调用| Providers
    Instance -->|多Provider| Providers
    MessageBus <-->|Outbound| Channels
```

## 核心数据流

```mermaid
sequenceDiagram
    participant User as 用户
    participant Channel as 通讯渠道
    participant Gateway as Gateway
    participant Bus as 消息总线
    participant Agent as Agent循环
    participant Tools as 工具系统
    participant Provider as LLM提供商
    participant OutChannel as 输出渠道

    User->>Channel: 发送消息
    Channel->>Gateway: InboundMessage
    Gateway->>Bus: PublishInbound()
    
    loop Agent处理循环
        Bus->>Agent: ConsumeInbound()
        Agent->>Agent: 处理命令(/show, /list, /switch)
        
        alt 需要LLM响应
            Agent->>Provider: Chat()
            Provider-->>Agent: LLM响应
            Agent->>Tools: 执行工具调用
            Tools-->>Agent: 工具结果
            Agent->>Provider: 继续对话
        end
        
        Agent->>Bus: PublishOutbound()
    end
    
    Bus->>OutChannel: 发送响应
    OutChannel-->>User: 显示消息
```

## 模块依赖关系

```mermaid
flowchart TB
    subgraph cmd层
        Main[主程序]
    end

    subgraph pkg核心
        Config[配置]
        Logger[日志]
    end

    subgraph pkg功能模块
        Bus[消息总线]
        Agent[Agent引擎]
        Providers[LLM提供商]
        Channels[通讯渠道]
        Tools[工具系统]
    end

    subgraph pkg支撑模块
        Session[会话]
        State[状态]
        Skills[技能]
        Cron[定时]
        Heartbeat[心跳]
        Media[媒体]
        Auth[认证]
        Routing[路由]
    end

    Main --> Config
    Main --> Logger
    Main --> Bus
    Main --> Agent
    
    Agent --> Providers
    Agent --> Tools
    Agent --> Session
    Agent --> Bus
    
    Tools --> Providers
    Tools --> Skills
    
    Channels --> Bus
    Channels --> Routing
    
    Bus --> Session
    Bus --> State
    Bus --> Media
    
    Cron --> Agent
    Heartbeat --> Agent
```

## 配置文件结构

```json
{
  "model_list": [...],
  "agents": {
    "defaults": {
      "model": "gpt-5.2",
      "max_tokens": 8192,
      "temperature": 0.7,
      "workspace": "~/.picoclaw/workspace"
    }
  },
  "channels": {
    "telegram": {...},
    "discord": {...},
    "whatsapp": {...},
    "feishu": {...},
    "qq": {...}
  },
  "tools": {
    "web": {
      "brave": {...},
      "duckduckgo": {...}
    },
    "cron": {...}
  },
  "heartbeat": {...}
}
```

## 架构特点

### 1. 极简轻量
- **<10MB 内存占用** - 比 OpenClaw 减少 99%
- **<1秒 启动时间** - 在 0.6GHz 单核上
- **单一二进制** - 支持 RISC-V, ARM, x86

### 2. 模块化设计
- **消息总线解耦** - 各组件通过消息总线通信
- **Provider 抽象** - 支持多种 LLM 提供商
- **Channel 插件化** - 支持多种通讯渠道

### 3. AI 驱动开发
- 95% 代码由 AI Agent 生成
- 自我引导式架构迁移

### 4. 多渠道支持
- 即时通讯: Telegram, Discord, WhatsApp, QQ, 飞书, 钉钉, Line, Slack
- 物联网: MaixCam, Pico 设备

### 5. 丰富工具集
- Web: Brave Search, DuckDuckGo, Tavily, Perplexity
- 硬件: I2C, SPI
- 技能: 动态安装/发现技能
- 子Agent: 支持Spawn子Agent
