# OpenClaw + n8n 工作流编排

让你的AI代理直接管理API密钥和调用外部服务是导致安全事件的原因。每个新集成意味着`.env.local`中又多了一个凭据，又多了一个代理可能意外泄露或滥用的渠道。

这个用例描述了一种模式，其中OpenClaw通过webhooks将所有外部API交互委托给n8n工作流——代理永远不会接触凭据，每个集成都可以可视化检查和锁定。

## 痛点

当OpenClaw直接处理所有事情时，你会遇到三个复杂的问题：

- **缺乏可见性**：当代理构建的内容隐藏在JavaScript技能文件或shell脚本中时，很难检查它实际做了什么
- **凭据扩散**：每个API密钥都存在于代理的环境中，一次错误提交就可能暴露
- **令牌浪费**：确定性子任务(发送电子邮件、更新电子表格)会消耗LLM推理令牌，而它们本可以作为简单工作流运行

## 功能介绍

- **代理模式**：OpenClaw使用传入webhooks编写n8n工作流，然后在所有未来的API交互中调用这些webhooks
- **凭据隔离**：API密钥存在于n8n的凭据存储中——代理只知道webhook URL
- **可视化调试**：每个工作流都可以在n8n的拖放UI中检查
- **可锁定工作流**：工作流构建和测试完成后，你可以锁定它，这样代理就无法修改它与API交互的方式
- **保护步骤**：你可以在n8n中添加验证、速率限制和批准门，然后再执行任何外部调用

## 工作原理

1. **代理设计工作流**：告诉OpenClaw你需要什么(例如，"创建一个工作流，当新的GitHub问题被标记为`urgent`时发送Slack消息")
2. **代理在n8n中构建**：OpenClaw通过n8n的API创建工作流，包括传入webhook触发器
3. **你添加凭据**：打开n8n的UI，手动添加你的Slack令牌/GitHub令牌
4. **你锁定工作流**：防止代理进一步修改
5. **代理调用webhook**：从现在开始，OpenClaw使用JSON有效负载调用`http://n8n:5678/webhook/my-workflow`——它永远不会看到API密钥

```text
┌──────────────┐     webhook call      ┌─────────────────┐     API call     ┌──────────────┐
│   OpenClaw   │ ───────────────────→  │   n8n Workflow   │ ─────────────→  │  External    │
│   (agent)    │   (no credentials)    │  (locked, with   │  (credentials   │  Service     │
│              │                       │   API keys)      │   stay here)    │  (Slack, etc)│
└──────────────┘                       └─────────────────┘                  └──────────────┘
```

## 所需技能

- `n8n` API访问(用于创建/触发工作流)
- `fetch`或`curl`用于webhook调用
- Docker(如果使用预配置的堆栈)
- n8n凭据管理(手动，每个集成一次性设置)

## 设置方法

### 选项1：预配置的Docker堆栈

一个社区维护的Docker Compose设置([openclaw-n8n-stack](https://github.com/caprihan/openclaw-n8n-stack))在共享Docker网络上预先配置了所有内容：

```bash
git clone https://github.com/caprihan/openclaw-n8n-stack.git
cd openclaw-n8n-stack
cp .env.template .env
# 将你的Anthropic API密钥添加到.env
docker-compose up -d
```

这会为你提供：
- OpenClaw在端口3456
- n8n在端口5678
- 共享Docker网络，因此OpenClaw可以直接调用`http://n8n:5678/webhook/...`
- 预构建的工作流模板(多LLM事实核查、电子邮件分类、社交监控)

### 选项2：手动设置

1. 安装n8n(`npm install n8n -g`或通过Docker运行)
2. 配置OpenClaw以了解n8n基本URL
3. 将此添加到你的AGENTS.md：

```text
## n8n集成模式

当我需要与外部API交互时：

1. 永远不要将API密钥存储在我的环境或技能文件中
2. 检查是否已经存在针对此集成的n8n工作流
3. 如果没有，通过n8n API创建一个带有webhook触发器的工作流
4. 通知用户添加凭据并锁定工作流
5. 对于所有未来的调用，使用带有JSON有效负载的webhook URL

工作流命名：openclaw-{service}-{action}
示例：openclaw-slack-send-message

Webhook调用格式：
curl -X POST http://n8n:5678/webhook/{workflow-name} \
  -H "Content-Type: application/json" \
  -d '{"channel": "#general", "message": "Hello from OpenClaw"}'
```

## 关键见解

- **一举三得**：可观察性(可视化UI)、安全性(凭据隔离)和性能(确定性工作流不消耗令牌)
- **测试后锁定**："构建→测试→锁定"周期至关重要——不锁定的话，代理可能会悄悄修改工作流
- **n8n有400+集成**：你想要连接的大多数外部服务已经有n8n节点，节省了代理编写自定义API调用的时间
- **免费审计跟踪**：n8n记录每个工作流执行的输入/输出数据

## 灵感来源

这种模式由[Simon Høiberg](https://x.com/SimonHoiberg/status/2020843874382487959)描述，他概述了这种方法优于让OpenClaw直接处理API交互的三个原因：通过n8n的可视化UI实现可观察性，通过凭据隔离实现安全性，以及通过将确定性子任务作为工作流运行而不是LLM调用来提高性能。[openclaw-n8n-stack](https://github.com/caprihan/openclaw-n8n-stack)存储库提供了实现此模式的现成Docker Compose设置。

## 相关链接

- [n8n文档](https://docs.n8n.io/)
- [openclaw-n8n-stack(Docker设置)](https://github.com/caprihan/openclaw-n8n-stack)
- [n8n Webhook触发器文档](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)