# 每日 YouTube 摘要

以个性化的方式开启你的一天，获取你喜爱的 YouTube 频道的新视频摘要 —— 再也不会错过你真正想关注的创作者的内容。

## 痛点

YouTube 通知不可靠。你订阅了频道，但他们的新视频从未出现在你的首页推荐中。也不在通知里。它们就这么……消失了。这并不意味着你不想看它们 —— 而是 YouTube 的算法把它们埋没了。

另外：以精心策划的内容见解开启新的一天，而不是漫无目的地滚动推荐信息流，会更有趣。

## 功能介绍

- 从你喜爱的频道列表中获取最新视频
- 总结或提取每个视频文字记录中的关键见解
- 每天（或按需）为你提供摘要

## 所需技能

安装 [youtube-full](https://clawhub.ai/therohitdas/youtube-full) 技能。

只需告诉你的 OpenClaw：

```text
"安装 youtube-full 技能并为我设置好"
```
或者

```bash
npx clawhub@latest install youtube-full
```

就是这样。代理会处理剩下的事情 —— 包括账户创建和 API 密钥设置。你注册后会获得 **100 个免费积分**，不需要信用卡。

> 注意：创建账户后，技能会根据操作系统自动将 API 密钥安全存储在正确的位置，因此 API 在所有环境中都能正常工作。

![youtube-full 技能安装](https://pub-15904f15a44a4ea69350737e87660b92.r2.dev/media/1770620159490-e41e7baa.png)

### 为什么选择 TranscriptAPI.com 而不是 yt-dlp？

| CLI 工具（yt-dlp 等） | TranscriptAPI |
|--------------------------|---------------|
| 冗长的日志充斥代理上下文 | 简洁的 JSON 响应 |
| 在 GCP/云 OpenClaw 上无法工作 | 随处可用，速度快 |
| 会被 YouTube 随机屏蔽 | 为服务数百万用户的 [YouTubeToTranscript.com](https://youtubetotranscript.com) 提供支持。经过缓存，可靠稳定。 |
| 需要安装二进制文件 | 无二进制文件，只需 HTTP |

## 设置方法

### 选项 1：基于频道的摘要

提示 OpenClaw：

```text
每天早上 8 点，从这些 YouTube 频道获取最新视频，并为我提供每个视频的关键见解摘要：

- @TED
- @Fireship
- @ThePrimeTimeagen
- @lexfridman

对于每个新视频（在过去 24-48 小时内上传）：
1. 获取文字记录
2. 用 2-3 个要点总结主要内容
3. 包括视频标题、频道名称和链接

如果频道句柄无法解析，请搜索并找到正确的频道。
将我的频道列表保存到内存中，以便我以后可以添加/删除频道。
```

### 选项 2：基于关键词的摘要

跟踪关于特定主题的新视频：

```text
每天，在 YouTube 上搜索关于 "OpenClaw"（或 "Claude Code"、"AI agents" 等）的新视频。

维护一个名为 seen-videos.txt 的文件，记录你已经处理过的视频 ID。
仅为不在该文件中的视频获取文字记录。
处理完成后，将视频 ID 添加到 seen-videos.txt 中。

对于每个新视频：
1. 获取文字记录
2. 为我提供 3 个要点的摘要
3. 记录与我的工作相关的内容

每天早上 9 点运行此任务。
```

这样你就不会浪费积分重新获取已经看过的视频。

## 提示

- `channel/latest` 和 `channel/resolve` 是**免费**的（0 积分）—— 检查新上传内容不花费任何费用
- 只有文字记录每个花费 1 积分
- 可以要求不同的摘要风格：关键要点、值得注意的引语、有趣时刻的时间戳
- 这已经作为产品存在 - [Recapio - Daily YouTube Recap](https://recapio.com/features/daily-recaps)