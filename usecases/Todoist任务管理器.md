# Todoist任务管理器：代理任务可见性

通过将内部推理和进度日志直接同步到Todoist，最大化长期运行的代理工作流的透明度。

## 痛点

当代理运行复杂的多步骤任务(如构建全栈应用或进行深度研究)时，用户常常无法跟踪代理当前正在做什么、已经完成了哪些步骤，以及代理可能在哪里卡住。对于后台任务，手动检查聊天日志很繁琐。

## 功能介绍

这个用例使用`todoist-task-manager`技能来：
1.  **可视化状态**：在特定部分创建任务，如`🟡 进行中`或`🟠 等待中`。
2.  **外部化推理**：将代理的内部"计划"发布到任务描述中。
3.  **流式日志**：实时将子步骤完成情况添加为任务评论。
4.  **自动对账**：心跳脚本检查停滞的任务并通知用户。

## 所需技能

你不需要预先构建的技能。只需提示你的OpenClaw代理创建下面**设置指南**中描述的bash脚本。由于OpenClaw可以管理自己的文件系统并执行shell命令，它会根据要求有效地为你"构建"技能。

## 详细设置指南

### 1. 配置Todoist

创建一个项目(例如，"OpenClaw工作区")并获取其ID。为不同状态创建部分：
- `🟡 进行中`
- `🟠 等待中`
- `🟢 已完成`

### 2. 实现："代理构建"的技能

无需安装技能，你可以要求OpenClaw为你创建这些脚本。每个脚本处理与Todoist API通信的不同部分。

**`scripts/todoist_api.sh`**(核心包装器)：
```bash
#!/bin/bash
# 用法: ./todoist_api.sh <端点> <方法> [数据_json]
ENDPOINT=$1
METHOD=$2
DATA=$3
TOKEN="YOUR_TODOIST_API_TOKEN"

if [ -z "$DATA" ]; then
  curl -s -X "$METHOD" "https://api.todoist.com/rest/v2/$ENDPOINT" \
    -H "Authorization: Bearer $TOKEN"
else
  curl -s -X "$METHOD" "https://api.todoist.com/rest/v2/$ENDPOINT" \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    -d "$DATA"
fi
```

**`scripts/sync_task.sh`**(任务和状态管理)：
```bash
#!/bin/bash
# 用法: ./sync_task.sh <任务内容> <状态> [任务_id] [描述] [labels_json_array]
CONTENT=$1
STATUS=$2
TASK_ID=$3
DESCRIPTION=$4
LABELS=$5
PROJECT_ID="YOUR_PROJECT_ID"

case $STATUS in
  "进行中") SECTION_ID="SECTION_ID_PROGRESS" ;;
  "等待中")     SECTION_ID="SECTION_ID_WAITING" ;;
  "已完成")        SECTION_ID="SECTION_ID_DONE" ;;
  *)             SECTION_ID="" ;;
esac

PAYLOAD="{\"content\": \"$CONTENT\""
[ -n "$SECTION_ID" ] && PAYLOAD="$PAYLOAD, \"section_id\": \"$SECTION_ID\""
[ -n "$PROJECT_ID" ] && [ -z "$TASK_ID" ] && PAYLOAD="$PAYLOAD, \"project_id\": \"$PROJECT_ID\""
if [ -n "$DESCRIPTION" ]; then
  ESC_DESC=$(echo "$DESCRIPTION" | sed ':a;N;$!ba;s/\n/\\n/g' | sed 's/"/\\"/g')
  PAYLOAD="$PAYLOAD, \"description\": \"$ESC_DESC\""
fi
[ -n "$LABELS" ] && PAYLOAD="$PAYLOAD, \"labels\": $LABELS"
PAYLOAD="$PAYLOAD}"

if [ -n "$TASK_ID" ]; then
  ./scripts/todoist_api.sh "tasks/$TASK_ID" POST "$PAYLOAD"
else
  ./scripts/todoist_api.sh "tasks" POST "$PAYLOAD"
fi
```

**`scripts/add_comment.sh`**(进度记录)：
```bash
#!/bin/bash
# 用法: ./add_comment.sh <任务_id> <评论文本>
TASK_ID=$1
TEXT=$2
ESC_TEXT=$(echo "$TEXT" | sed ':a;N;$!ba;s/\n/\\n/g' | sed 's/"/\\"/g')
PAYLOAD="{\"task_id\": \"$TASK_ID\", \"content\": \"$ESC_TEXT\"}"
./scripts/todoist_api.sh "comments" POST "$PAYLOAD"
```

### 3. 使用提示

你可以给你的代理这个提示来**设置**和**使用**可见性系统：

```text
我希望你为自己的运行构建一个基于Todoist的任务可见性系统。

首先，在'scripts/'文件夹中创建三个bash脚本：
1. todoist_api.sh (Todoist REST API的curl包装器)
2. sync_task.sh (创建或更新任务，为进行中、等待中和已完成状态使用特定的section_ids)
3. add_comment.sh (将进度日志发布为评论)

使用这些变量进行设置：
- 令牌：[你的Todoist API令牌]
- 项目ID：[你的项目ID]
- 部分ID：[进行中ID、等待中ID、已完成ID]

创建完成后，对于我给你的每个复杂任务：
1. 在'进行中'部分创建一个任务，任务描述中包含你的完整计划。
2. 对于每个子步骤的完成，调用add_comment.sh，记录你所做的事情。
3. 完成后将任务移动到'已完成'部分。
```

## 相关链接

- [Todoist REST API文档](https://developer.todoist.com/rest/v2/)