# Polymarket自动驾驶：自动化模拟交易

手动监控预测市场寻找套利机会并执行交易非常耗时，需要持续关注。你想在不冒真实资金风险的情况下测试和改进交易策略。

这个工作流使用自定义策略自动化Polymarket上的模拟交易：

• 通过API监控市场数据(价格、交易量、点差)
• 使用TAIL(趋势跟踪)和BONDING(逆向)策略执行模拟交易
• 跟踪投资组合表现、损益和胜率
• 将每日摘要发送到Discord，包含交易日志和见解
• 从模式中学习：根据回测结果调整策略参数

## 痛点

预测市场变化很快。手动交易意味着错失机会、情绪化决策，以及难以跟踪哪些有效。用真钱测试策略意味着在你了解市场行为之前就有损失风险。

## 功能介绍

自动驾驶系统持续扫描Polymarket寻找机会，使用可配置策略模拟交易，并记录所有内容以供分析。你醒来时会看到它一夜之间"交易"了什么、哪些有效、哪些无效的摘要。

示例策略：
- **TAIL**：当交易量激增且趋势明显时跟随趋势
- **BONDING**：当市场对新闻反应过度时买入逆向头寸
- **SPREAD**：识别有套利潜力的定价错误市场

## 所需技能

- `web_search`或`web_fetch`(用于Polymarket API数据)
- `postgres`或SQLite用于交易日志和投资组合跟踪
- Discord集成用于每日报告
- Cron作业用于持续监控
- 生成子代理用于并行市场分析

## 设置方法

1. 为模拟交易设置数据库：
```sql
CREATE TABLE paper_trades (
  id SERIAL PRIMARY KEY,
  market_id TEXT,
  market_name TEXT,
  strategy TEXT,
  direction TEXT,
  entry_price DECIMAL,
  exit_price DECIMAL,
  quantity DECIMAL,
  pnl DECIMAL,
  timestamp TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE portfolio (
  id SERIAL PRIMARY KEY,
  total_value DECIMAL,
  cash DECIMAL,
  positions JSONB,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

2. 创建一个Discord频道用于更新(例如，#polymarket-autopilot)。

3. 提示OpenClaw：
```text
你是Polymarket模拟交易自动驾驶系统。持续运行(每15分钟通过cron)：

1. 从Polymarket API获取当前市场数据
2. 使用以下策略分析机会：
   - TAIL：跟随强烈趋势(>60%概率+交易量激增)
   - BONDING：对过度反应的逆向操作(新闻导致的突然下跌>10%)
   - SPREAD：当YES+NO > 1.05时进行套利
3. 在数据库中执行模拟交易(初始资金：$10,000)
4. 跟踪投资组合状态并更新头寸

每天早上8点，发布摘要到Discord #polymarket-autopilot：
- 昨天的交易(入场/出场价格、损益)
- 当前投资组合价值和未平仓头寸
- 胜率和策略表现
- 市场见解和建议

在高交易量期间使用子代理并行分析多个市场。

永远不要使用真钱。这只是模拟交易。
```

4. 根据表现迭代策略。调整阈值、添加新策略、回测历史数据。

## 相关链接

- [Polymarket API](https://docs.polymarket.com/)
- [模拟交易最佳实践](https://www.investopedia.com/articles/trading/11/paper-trading.asp)