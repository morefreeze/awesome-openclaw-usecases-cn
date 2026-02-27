# 带自动联系人发现的个人CRM

手动跟踪你见过的人、时间和讨论的内容是不可能的。重要的跟进会被遗漏，在重要会议前你会忘记上下文。

这个工作流自动构建和维护个人CRM：

• 每日cron作业扫描电子邮件和日历以查找新联系人和互动
• 将联系人存储在带有关系上下文的结构化数据库中
• 自然语言查询："我了解[某人]什么？"、"谁需要跟进？"、"我上次和[某人]谈话是什么时候？"
• 每日会议准备简报：每天会议前，通过CRM + 电子邮件历史研究外部与会者并提供简报

## 所需技能

- `gog` CLI(用于Gmail和Google Calendar)
- 自定义CRM数据库(SQLite或类似)，如果可用，使用[crm-query](https://clawhub.ai)技能
- 用于CRM查询的Telegram主题

## 设置方法

1. 创建CRM数据库：
```sql
CREATE TABLE contacts (
  id INTEGER PRIMARY KEY,
  name TEXT,
  email TEXT,
  first_seen TEXT,
  last_contact TEXT,
  interaction_count INTEGER,
  notes TEXT
);
```
2. 创建一个名为"personal-crm"的Telegram主题用于查询。
3. 提示OpenClaw：
```text
每天早上6点运行cron作业：
1. 扫描我过去24小时的Gmail和日历
2. 提取新联系人并更新现有联系人
3. 记录互动(会议、电子邮件)，包括时间戳和上下文

另外，每天早上7点：
1. 查看我今天的会议日历
2. 对于每个外部与会者，搜索我的CRM和电子邮件历史
3. 向Telegram发送简报：他们是谁，我们上次谈话的时间，我们讨论的内容，以及任何后续项目

当我在personal-crm主题中询问联系人时，搜索数据库并告诉我你所知道的一切。
```