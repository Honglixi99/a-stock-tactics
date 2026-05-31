# a-stock-tactics

> A 股短线战法工具包 —— 给 AI 编程助手用的「短线打板/强势股」分析 Skill

把一套 100 条的短线战法，变成 AI 一句话就能执行的能力：拉 A 股行情，自动算均线/量能/K线形态/技术指标，按战法规则打分，输出 **买入候选 / 持有 / 卖出 / 回避**。

核心是一个自包含的 `SKILL.md`（结构化 Markdown + 内嵌可运行 Python），兼容 [Claude Code](https://github.com/anthropics/claude-code)、Codex 等支持上下文注入的 AI 编程助手。

---

## ✨ 能做什么

- **个股短线体检**：`stock_diagnosis("600879")` 一键输出战法结论 + 命中的规则号
- **战法计算层**：均线（MA5/MA24生命线/MA60季线）、均量线金叉死叉、K线形态（涨停/光头大阳/红三兵/跳空/十字星）、MACD/威廉指标、大盘多空环境、周线确认
- **选股**：拉当日强势股（同花顺热点，带题材归因）→ 批量体检筛候选
- **题材联动 / 资金验证**：概念板块归属、行业板块强弱、个股资金流、龙虎榜席位、限售解禁
- **数据源不封 IP**：行情走通达信(mootdx, TCP) + 腾讯，东财接口内置限流防封；全部免费无 Key

战法三要素：**① 均线位置 · ② 量能状态 · ③ 所处高低位**。生命线（MA24）是多空总分界——股价在其上方才谈买点，下方一律回避。

---

## 📦 别人怎么使用这个 Skill（安装与用法）

### 方式一：作为 Claude Code 的 Skill（推荐）

```bash
# 1. 创建 skill 目录
mkdir -p ~/.claude/skills/a-stock-tactics      # Windows: %USERPROFILE%\.claude\skills\a-stock-tactics

# 2. 下载 SKILL.md 放进去
curl -o ~/.claude/skills/a-stock-tactics/SKILL.md \
  https://raw.githubusercontent.com/Honglixi99/a-stock-tactics/main/SKILL.md

# 3. 安装 Python 依赖
pip install mootdx requests pandas stockstats
```

启动 Claude Code，直接说人话即可自动激活：

| 你说 | 它做 |
|------|------|
| 「帮我体检一下 600879」 | 跑 `stock_diagnosis`，给买/卖/回避结论 |
| 「今天哪些强势股值得关注」 | 拉同花顺热点 → 批量体检筛候选 |
| 「000858 跌破生命线了吗」 | 算均线，判断 MA24 位置 |
| 「605277 是不是放量涨停」 | 算量能 + K线形态 |
| 「现在大盘什么环境」 | `market_regime()` 看指数多空 |
| 「我持仓这几只要不要跑」 | 逐只跑卖出预警 |

> **Windows 用户**：把上面路径换成 `%USERPROFILE%\.claude\skills\a-stock-tactics\`。

### 方式二：其他 AI 助手（Codex 等）

把 `SKILL.md` 的内容贴进系统 prompt 或项目上下文文件即可，内嵌的 Python 代码可直接执行。

### 方式三：当普通 Python 库直接用

`SKILL.md` 里的代码段可直接复制到自己的脚本里跑：

```python
# 摘自 SKILL.md「Layer 2 战法计算层」
d = stock_diagnosis("600879", "航天电子")
print(d["结论"])          # 回避/卖出
print(d["卖出回避信号"])   # ['18/88:跌破生命线', '88:跌破季线', ...]

d = stock_diagnosis("605277", "新亚电子")
print(d["结论"])          # 强势买入候选
```

---

## 🗂️ 数据架构（7 层）

```
Layer 1 行情层    mootdx(K线/盘口/逐笔) + 腾讯(实时价/量比/换手) + 百度K线
Layer 2 战法计算层 ★均线 / 均量线金叉死叉 / K线形态 / MACD威廉 / 大盘 / 周线 / stock_diagnosis
Layer 3 信号题材   同花顺热点题材 + 概念板块 + 行业排名 + 资金流 + 龙虎榜 + 解禁
Layer 4 资金流向   个股资金流 120 日（验证放量真假）
Layer 5 新闻层     东财个股新闻（消息面避雷）
Layer 6 基础数据   mootdx F10 + 东财个股信息（流通盘/题材/上市日期）
Layer 7 公告层     巨潮 cninfo 公告全文
```

详见 [`SKILL.md`](./SKILL.md)。

---

## 🔧 环境要求

| 依赖 | 用途 |
|------|------|
| `mootdx>=0.10` | 通达信 TCP 行情（K线/盘口/F10/指数），战法计算层数据源 |
| `requests` | HTTP 数据源直连 |
| `pandas` | 均线/均量线计算 |
| `stockstats` | MACD / 威廉 / RSI 指标 |

- **mootdx 走 TCP 直连通达信服务器，需国内 IP 才稳定**；海外环境建议走代理。
- 全部数据源免费无 Key。

---

## ⚠️ 免责声明

本工具仅把战法中**可量化**的均线/量能/形态规则编码为自动评分，是「技术形态体检」，**不含**基本面、消息面、盘口博弈，**不构成任何投资建议**。据此操作风险自负，股市有风险，入市需谨慎。

---

## 📄 License

[MIT](./LICENSE)
