# 多渠道AI客户服务平台

小型企业要在多个应用之间管理WhatsApp、Instagram私信、电子邮件和Google评论。客户期望全天候即时响应，但雇佣人员提供全天候服务成本高昂。

这个用例将所有客户接触点整合到一个由AI驱动的收件箱中，代表你智能地回复客户。

## 功能介绍

- **统一收件箱**：WhatsApp Business、Instagram私信、Gmail和Google评论集中在一个地方
- **AI自动回复**：自动处理常见问题、预约请求和常见咨询
- **人工移交**：将复杂问题升级或标记为需要审核
- **测试模式**：在不影响真实客户的情况下向客户演示系统
- **业务上下文**：根据你的服务、定价和政策进行训练

## 真实业务示例

在Futurist Systems，我们为本地服务企业(餐厅、诊所、沙龙)部署了这个系统。一家餐厅将响应时间从4个多小时缩短到2分钟以内，自动处理80%的咨询。

## 所需技能

- WhatsApp Business API集成
- Instagram Graph API(通过Meta Business)
- 用于Gmail的`gog` CLI
- 用于评论的Google Business Profile API
- AGENTS.md中的消息路由逻辑

## 设置方法

1. 通过OpenClaw配置**连接频道**：
   - WhatsApp Business API(通过360dialog或官方API)
   - 通过Meta Business Suite连接Instagram
   - 通过`gog` OAuth连接Gmail
   - Google Business Profile API令牌

2. **创建业务知识库**：
   - 服务和定价
   - 营业时间和位置
   - 常见问题解答
   - 升级触发条件(例如，投诉、退款请求)

3. 配置AGENTS.md**的路由逻辑**：

```text
## 客户服务模式

收到客户消息时：

1. 识别频道(WhatsApp/Instagram/电子邮件/评论)
2. 检查此客户是否启用了测试模式
3. 分类意图：
   - 常见问题 → 从知识库中回复
   - 预约 → 检查可用性，确认预订
   - 投诉 → 标记为需要人工审核，确认收到
   - 评论 → 感谢反馈，解决问题

回复风格：
- 友好、专业、简洁
- 匹配客户的语言(ES/EN/UA)
- 永远不要编造知识库中没有的信息
- 以企业名称结尾

测试模式：
- 回复前加上[TEST]
- 记录但不发送到真实频道
```

4. 设置心跳检查**以监控响应**：

```text
## 心跳：客户服务检查

每30分钟：
- 检查超过5分钟未回复的消息
- 如果响应队列积压，发出警报
- 记录每日响应指标
```

## 关键见解

- **语言检测很重要**：自动检测并使用客户的语言回复
- **测试模式至关重要**：客户需要在上线前看到它的工作效果
- **移交规则**：定义明确的升级触发条件，避免AI越权
- **回复模板**：敏感话题(退款、投诉)的预先批准模板

## 相关链接

- [WhatsApp Business API](https://developers.facebook.com/docs/whatsapp)
- [Instagram消息API](https://developers.facebook.com/docs/instagram-api/guides/messaging)
- [Google Business Profile API](https://developers.google.com/my-business)