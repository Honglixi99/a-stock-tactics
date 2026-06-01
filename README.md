# a-stock-tactics

> A 股短线战法工具包 —— 给 AI 编程助手用的「短线打板/强势股」分析 Skill

一句话，让 AI 像一支**分析师团队**那样给 A 股个股做体检——内置 9 个分析师，各出一份观点：

> 📜 **战法分析师**（一套 100 条短线打板战法，生命线 MA24 多空总纲）· 🔧 **技术分析师**（均线/量能/K线形态/MACD-威廉-RSI/周线）· 🌏 **大盘分析师**（指数多空环境）· 🎯 **游资分析师**（量价异动/连板语义）· 💰 **资金分析师**（主力vs散户博弈四态）· 📊 **基本面分析师**（盈利/估值/ROE/负债/现金流质量）· 🔓 **解禁分析师**（解禁压力评级）· 📋 **纪律提示官**（30条经验铁律）· ⚖️ **多空辩手**（🐂多头席 vs 🐻空头席对辩）

**AI（Claude）担任裁判**，综合这 9 位分析师的观点，输出 **五档评级（Buy/增持/持有/减持/卖出）+ 多空对辩报告 + 命中的规则号**，并能筛"可低吸位置"（不追涨停）。

核心是一个自包含的 `SKILL.md`（结构化 Markdown + 内嵌可运行 Python），兼容 [Claude Code](https://github.com/anthropics/claude-code)、Codex 等支持上下文注入的 AI 编程助手。

> **当前版本 v1.6.2** ·「技术打分 + 基本面避雷 + 人工规则提示」体检 +「低吸选股」+「资金博弈四态/连板语义/解禁压力/ROE质量」+「五档评级 + 多空对辩 + T+1陷阱」（吸收自 TradingAgents 全套分析框架）。**v1.6.2 修复送转/分红股均线失真：`get_daily()` 自动前复权**。

---

## ✨ 能做什么

一次 `stock_diagnosis(code)` 给出**九分析师协作的完整体检**（Claude 综合后输出五档评级）：

**① 技术打分（战法计算层）**
- 均线（MA5/MA24生命线/MA60季线 + MA50/MA200金叉死叉 + VWMA量价加权）、均量线金叉死叉、K线形态（涨停/光头大阳/红三兵/跳空/十字星）、MACD/威廉/RSI/布林带、大盘多空环境、周线确认
- **自动前复权（v1.6.2）**：`get_daily()` 自动检测送转/分红除权缺口并前复权，避免把刚除权的强势股正常回调误判为"跌破生命线/季线"的空头崩塌（mootdx 默认不复权，曾把刚 10转5 的中微公司误判「卖出」，前复权后实为站稳生命线的「持有」）
- **RSI 按 A 股修正**：强势股 RSI 60-85 是常态而非超买（仅 >85 提示过热），避免误杀强势股
- 按 100 条战法可量化规则打分 → **五档评级（Buy/增持/持有/减持/卖出）**，并给出 **🐂多头席 / 🐻空头席 / ⚖️战法裁决** 多空对辩（T+1陷阱：今日涨停追高自动降级）

**② 基本面避雷（risk_check）**
- 自动查**亏损**（净利润为负）、**估值异常**（PE 负 / >200）、**ST 风险**
- **财务质量**：ROE、资产负债率（>70%预警）、经营现金流/净利润匹配度（<0.3提示利润含金量低）
- **A股估值参照系**：PE中位数30-50x为常态，对标同行业横向比，勿套用美股15-25x
- **妖股顶部识别**：近10日涨停≥3次 / 20日涨≥50% / 天量≥4倍 / 三板以上 → 把"爆炒亏损股"评级下调为"减持"
- **解禁压力评级**：占流通比 >20%重大 / 5~20%中等 / <5%轻微

**③ 人工规则提示（manual_reminders）**
- 100 条里约 30 条靠经验的纪律（波段计数、追高、暴跌后第一反弹、洗盘判断）无法自动算
- 按形态特征**主动提醒**该结合哪条人工规则（如：近期跌停+今日反弹→提示规则93不追第一反弹；涨停偏离5日线过远→提示规则27/28不追高）

**④ 低吸选股（find_pullback_buy / scan_pullback）— 不追涨停**
- 筛战法真正的低吸买点，而非追已涨停的票（规则28：不在高位追涨停）
- **涨停回踩企稳**（规则59/26：牛股第一次回调到5日线、企稳）+ **横盘突破前夜**（规则40/85：整理后第一根放量、尚未涨停）
- `scan_pullback([代码列表])` 批量筛"可低吸位置"的标的

**⑤ 资金面 + 游资框架（fund_flow_em / capital_battle / volume_anomaly）**
- 当日主力净流入 / 超大单 / 大单 / 中单 / 小单（散户≈中小单），走东财 push2（间歇可达，拿不到则标 `[数据缺失]`，不影响主流程）
- **资金博弈四态**：主力吸筹 / 主力出货 / 游资接力 / 散户主导诱多（主力净+散户净+量价组合判定）
- **量价异动 + 连板语义**：日量>20日均量2倍=放量异动；首板放量=分歧/缩量=一致；三板以上=妖股模式
- 吸收自兄弟项目 [TradingAgents-astock](https://github.com/simonlin1212/TradingAgents-astock) 游资追踪分析师的量化判据

**其它能力**
- **选股**：拉当日强势股（同花顺热点，带题材归因）→ 批量体检筛候选
- **题材联动 / 资金验证**：概念板块归属、行业板块强弱、个股资金流、龙虎榜席位、限售解禁
- **数据源不封 IP**：行情走通达信(mootdx, TCP) + 腾讯，东财接口内置限流防封；全部免费无 Key

战法三要素：**① 均线位置 · ② 量能状态 · ③ 所处高低位**。生命线（MA24）是多空总分界——股价在其上方才谈买点，下方一律偏空（减持/卖出）。

---

## 📦 别人怎么使用这个 Skill（安装与用法）

### 方式一：作为 Claude Code 插件市场一键安装（推荐）

本仓库是一个 Claude Code 插件市场（marketplace）。在 Claude Code 里：

```
/plugin marketplace add Honglixi99/a-stock-tactics
/plugin install a-stock-tactics@a-stock-tactics
```

然后安装 Python 依赖：

```bash
pip install mootdx requests pandas stockstats
```

### 方式二：手动放入 skill 目录

```bash
# 1. 创建 skill 目录
mkdir -p ~/.claude/skills/a-stock-tactics      # Windows: %USERPROFILE%\.claude\skills\a-stock-tactics

# 2. 下载 SKILL.md（注意：仓库已是插件市场结构，SKILL.md 在 plugins 子目录）
curl -o ~/.claude/skills/a-stock-tactics/SKILL.md \
  https://raw.githubusercontent.com/Honglixi99/a-stock-tactics/main/plugins/a-stock-tactics/skills/a-stock-tactics/SKILL.md

# 3. 安装 Python 依赖
pip install mootdx requests pandas stockstats
```

启动 Claude Code，直接说人话即可自动激活：

| 你说 | 它做 |
|------|------|
| 「帮我体检一下 600879」 | 跑 `stock_diagnosis`，九分析师协作 → 五档评级 + 多空对辩 |
| 「今天哪些强势股值得关注」 | 拉同花顺热点 → 批量体检筛候选 |
| 「000858 跌破生命线了吗」 | 算均线，判断 MA24 位置 |
| 「605277 是不是放量涨停」 | 算量能 + K线形态 |
| 「现在大盘什么环境」 | `market_regime()` 看指数多空 |
| 「我持仓这几只要不要跑」 | 逐只跑卖出预警 |
| 「有没有可以低吸的票（别追涨停）」 | `scan_pullback` 筛涨停回踩/横盘突破位 |

> **Windows 用户**：把上面路径换成 `%USERPROFILE%\.claude\skills\a-stock-tactics\`。

### 方式三：其他 AI 助手（Codex 等）

把 `SKILL.md` 的内容贴进系统 prompt 或项目上下文文件即可，内嵌的 Python 代码可直接执行。

### 方式四：当普通 Python 库直接用

`SKILL.md` 里的代码段可直接复制到自己的脚本里跑：

```python
# 摘自 SKILL.md「Layer 2 战法计算层」
d = stock_diagnosis("600879", "航天电子")
print(d["结论"])          # 卖出（五档之一）
print(d["卖出回避信号"])   # ['18/88:跌破生命线', '88:跌破季线', ...]
print(d["多空对辩"])       # {🐂多头席, 🐻空头席, ⚖️战法裁决}

d = stock_diagnosis("605277", "新亚电子")
print(d["结论"])          # 持有（T+1陷阱使涨停股不盲目判买入）
```

---

## 🗂️ 数据架构（7 层）

```
Layer 1 行情层    mootdx(K线/盘口/逐笔) + 腾讯(实时价/量比/换手) + 百度K线
Layer 2 战法计算层 ★均线 / 均量线金叉死叉 / K线形态 / MACD威廉 / 大盘 / 周线 / stock_diagnosis / find_pullback_buy(低吸) / fund_flow_em(主力散户)
Layer 3 信号题材   同花顺热点题材 + 概念板块 + 行业排名 + 资金流 + 龙虎榜 + 解禁
Layer 4 资金流向   个股资金流 120 日（验证放量真假）
Layer 5 新闻层     东财个股新闻（消息面避雷）
Layer 6 基础数据   mootdx F10 + 东财个股信息（流通盘/题材/上市日期）
Layer 7 公告层     巨潮 cninfo 公告全文
```

详见 [`SKILL.md`](./plugins/a-stock-tactics/skills/a-stock-tactics/SKILL.md)、战法原文 [`reference/战法100条.md`](./plugins/a-stock-tactics/skills/a-stock-tactics/reference/战法100条.md) 与分析框架手册 [`reference/分析框架.md`](./plugins/a-stock-tactics/skills/a-stock-tactics/reference/分析框架.md)。

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

本工具以**技术形态**为主（均线/量能/K线形态评分），辅以**基本面避雷**（亏损/估值/妖股顶部）和**人工规则提示**。它不含盘口实时博弈、不替代深度基本面分析，**不构成任何投资建议**。据此操作风险自负，股市有风险，入市需谨慎。

---

## 🙏 致谢

本项目的多维分析框架（游资/连板语义、解禁压力评级、基本面质量、技术指标解读、多空对辩、五档评级等量化判据）借鉴自 [TauricResearch/TradingAgents](https://github.com/TauricResearch/TradingAgents)（Apache 2.0）及其 A 股特化版 [TradingAgents-astock](https://github.com/simonlin1212/TradingAgents-astock)。感谢原作者的出色工作和开源精神。

原始论文：*TradingAgents: Multi-Agents LLM Financial Trading Framework*。

本项目仅吸收其**可量化的分析判据**编码为离线纯函数规则，不含其多 Agent / LLM 架构，保持轻量 skill 定位。

---

## 📄 License

[MIT](./LICENSE)
