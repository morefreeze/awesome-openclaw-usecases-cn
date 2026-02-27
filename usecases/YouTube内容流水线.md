# YouTube内容流水线

作为日常YouTube创作者，在网络和X/Twitter上寻找新鲜、及时的视频创意非常耗时。跟踪你已经涵盖的内容可以避免重复，并帮助你领先于趋势。

这个工作流自动化了整个内容侦查和研究流水线：

• 每小时cron作业扫描突发AI新闻(网页+X/Twitter)并将视频创意发布到Telegram
• 维护包含观看次数和主题分析的90天视频目录，以避免重复涵盖主题
• 将所有创意存储在带有向量嵌入的SQLite数据库中，用于语义去重(这样你永远不会得到相同的创意两次)
• 当你在Slack中分享链接时，OpenClaw研究该主题，在X上搜索相关帖子，查询你的知识库，并创建一个带有完整大纲的Asana卡片

## 所需技能

- `web_search`(内置)
- [x-research-v2](https://clawhub.ai)或自定义X/Twitter搜索技能
- [knowledge-base](https://clawhub.ai)技能用于RAG
- Asana集成(或Todoist)
- 用于YouTube Analytics的`gog` CLI
- 用于接收创意的Telegram主题

## 设置方法

1. 为视频创意设置Telegram主题并在OpenClaw中配置。
2. 安装knowledge-base技能和x-research技能。
3. 为创意跟踪创建SQLite数据库：
```sql
CREATE TABLE pitches (
  id INTEGER PRIMARY KEY,
  timestamp TEXT,
  topic TEXT,
  embedding BLOB,
  sources TEXT
);
```
4. 提示OpenClaw：
```text
运行每小时cron作业来：
1. 在网页和X/Twitter上搜索突发AI新闻
2. 对照我的90天YouTube目录(通过gog从YouTube Analytics获取)
3. 对照数据库中所有过去的创意进行语义相似度检查
4. 如果是新颖的，将创意发布到我的Telegram "video ideas" 主题并附上来源

另外：当我在Slack #ai_trends中分享链接时，自动：
1. 研究该主题
2. 在X上搜索相关帖子
3. 查询我的知识库
4. 在Video Pipeline中创建一个带有完整大纲的Asana卡片
```