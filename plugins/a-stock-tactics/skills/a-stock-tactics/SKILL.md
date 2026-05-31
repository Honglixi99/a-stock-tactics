---
name: a-stock-tactics
description: A股短线战法工具包 — 行情数据(通达信mootdx+腾讯)+战法计算层(均线MA5/10/24生命线/60季线、均量线金叉死叉、K线形态识别、MACD/威廉指标、大盘环境、周线确认、涨停板质量)+一键战法体检(stock_diagnosis)+信号资金题材(同花顺热点题材归因、概念板块、行业排名、个股资金流、龙虎榜、解禁、个股新闻、F10/公告)。基于"均线位置+量能状态+所处高低位"三要素的100条短线打板战法，自动按规则打分输出 买入候选/持有/卖出/回避。适用于涨停打板、强势股筛选、均线生命线判断、量能金叉死叉、题材联动、龙虎榜跟踪、个股短线体检等场景。
origin: custom
version: 1.5.0
---

# A股短线战法工具包 V1.5.0

基于一套 100 条短线打板/强势股战法，把**原始行情数据**转成战法决策信号。核心三要素：**均线位置（MA5/MA24生命线/MA60季线）· 量能状态（均量线金叉死叉、持续放量）· 所处高低位**。覆盖主板/中小板/科创板/创业板/ST。

> **V1.5.0（吸收 TradingAgents 游资分析框架）：**
> - **新增 `volume_anomaly()`**：量价异动（日量>20日均量2倍=放量异动）+ **连板放缩量语义**（首板放量=分歧/缩量=一致；三板以上=妖股模式）。`risk_check` 妖股识别接入连板语义，三板以上直接标妖股模式。
> - **新增 `capital_battle()`**：综合主力净/散户净+量价，判定**资金博弈四态**（主力吸筹/主力出货/游资接力/散户主导诱多）。`stock_diagnosis` 资金面升级为四态判定。
> - 吸收自兄弟项目 TradingAgents-astock 游资追踪分析师的量化判据。
>
> **V1.4.2（移除回测库，回归纯体检工具）：** 移除 `save_diagnosis`/`save_batch`/`backtest` 及本地 CSV 存档——保持轻量纯函数定位，体检结果即时输出不落盘。
>
> **V1.4.1（资金面 + 成交量）：**
> - **新增 `fund_flow_em()`**：当日主力/超大单/大单/中单/小单净额。走 push2 域名（`fflow/kline/get` + `klt=101`，比历史接口 push2his 连通性好；间歇可达，已内置重试+绕过系统代理）。`stock_diagnosis` 输出新增「资金面」（主力净/超大单/散户净=中小单），区分主力 vs 散户。
>
> **V1.3.0（新增低吸选股，解决"只会追涨停"缺陷）：**
> - **新增 `find_pullback_buy()` / `scan_pullback()`**：筛战法真正的买点——**涨停回踩企稳**（规则59/26：牛股第一次回调到5日线缩量/温和量企稳）和**横盘突破前夜**（规则40/85：整理后第一根放量、尚未涨停）。
> - **专门排除"当日涨停、偏离5日线过远"的追高位**（规则27/28）。此前 `stock_diagnosis` 的"强势买入候选"几乎全是当天涨停股，与"不追高"规则自相矛盾；本函数提供可低吸的位置，是对它的补充。
>
> **V1.2.0（新增人工规则提示）：**
> - **新增 `manual_reminders()`**：100 条里约 30 条是靠经验的"人工纪律"（波段计数、追高、暴跌后第一反弹、洗盘判断等），无法自动打分。本函数按形态特征**主动提醒**该结合哪条人工规则思考（如：近期跌停+今日反弹→提示规则93；涨停偏离5日线过远→提示规则27/28不追高）。
> - `stock_diagnosis` 输出新增 `需人工判断` 字段。让使用者不漏掉那些工具算不了、但战法很看重的经验铁律。
>
> **V1.1.0（新增基本面避雷层）：**
> - **新增 `risk_check()`**：纯技术战法的避雷补充——亏损（mootdx净利润）+ 估值异常（PE负/>200）+ ST + **妖股顶部识别**（近10日涨停≥3次/20日涨≥50%/天量≥4倍，规则09/44/76/88）+ 个股新闻（尽力而为）。
> - `stock_diagnosis` 集成避雷：**妖股顶部风险会把"买入候选"自动降级为"高风险观望"**，并输出 `风险提示` + `基本面` 字段。解决了纯技术把"爆炒亏损股"误判为强势的盲点。
>
> **V1.0.0（短线战法工具包）：**
> - **新增「战法计算层」**：`compute_ma`（均线/死叉）· `compute_vol_ma`（均量线金叉死叉/持续放量）· `detect_patterns`（涨停/光头大阳/红三兵/跳空/十字星）· `compute_indicators`（MACD/威廉/RSI）· `market_regime`（大盘多空）· `weekly_view`（周线确认）· `limit_up_quality`（涨停板质量）。
> - **新增一键体检 `stock_diagnosis(code)`**：整合上述计算，按 100 条战法可量化规则自动打分，输出 **买入候选 / 持有 / 卖出 / 回避** 及命中的规则号。
> - **移除价值投资层**：研报/一致预期EPS/iwencai/财报三表/融资融券/大宗交易/股东户数/分红/PEG估值——与短线战法冲突，已删除。
> - 保留：行情（mootdx/腾讯/百度K线）· 同花顺热点题材 · 概念板块 · 行业排名 · 个股资金流 · 龙虎榜 · 解禁 · 个股新闻 · F10/个股信息 · 巨潮公告。

**使用方式：** 将本文件放入 `~/.claude/skills/a-stock-tactics/SKILL.md`，Claude Code 会自动识别并在 A 股短线相关对话中激活。

```
Layer 1 行情层（不封IP，战法数据源）
├── mootdx        → K线 + 五档盘口 + 逐笔成交 (TCP 7709)
├── 腾讯财经 API   → 实时价/换手率/量比/振幅/涨跌停价/指数/ETF (HTTP)
└── 百度股市通     → K线带MA5/10/20 (HTTP)

Layer 2 战法计算层（核心·本工具包的"大脑"）
├── compute_ma         → MA5/10/20/24生命线/60季线 + 斜率 + 5日线死叉生命线
├── compute_vol_ma     → VMA5/10/15 + 金叉死叉 + 持续放量
├── detect_patterns    → 涨停/一字板/光头大阳/红三兵/跳空/十字星/长下影/高位放量收阴
├── compute_indicators → MACD / 威廉WR / RSI (stockstats)
├── market_regime      → 上证/深成/创业板 多空环境
├── weekly_view        → 周线方向 + 三线合一
├── limit_up_quality   → 封单/换手（涨停板质量）
└── stock_diagnosis    → ★一键体检：按战法打分 → 买入候选/持有/卖出/回避

Layer 3 信号/题材层
├── 同花顺热点     → 当日强势股 + 题材归因 reason tags (零鉴权 73ms)
├── 百度股市通     → 概念板块归属 (HTTP)
├── 东财 push2     → 个股资金流向 分钟级
├── 龙虎榜席位     → 上榜记录 + 买卖席位 TOP5 + 机构动向
├── 全市场龙虎榜   → 每日全市场上榜股票 + 净买额排名
├── 限售解禁日历   → 历史解禁 + 未来90天待解禁
└── 行业板块排名   → 东财行业涨跌/上涨下跌家数

Layer 4 资金流向 ── 个股资金流120日（主力/大单/中单/小单 日级净流入）
Layer 5 新闻层   ── 东财个股新闻 (search-api-web JSONP)
Layer 6 基础数据 ── mootdx F10(9大类) + 东财个股信息(行业/股本/流通/市值/上市日期)
Layer 7 公告层   ── 巨潮 cninfo 公告全文 + mootdx F10 公告摘要
```

## 数据源优先级 & 东财防封（重要，先读）

### 优先级原则：能用通达信/腾讯，就别用东财

| 优先级 | 数据源 | 协议 | 封 IP 风险 | 覆盖 |
|--------|--------|------|-----------|------|
| **1（首选）** | **mootdx（通达信）** | TCP 7709 二进制 | **不封 IP** | K线、五档盘口、逐笔成交、财务快照、F10 |
| **2** | **腾讯财经** | HTTP GBK | **不封 IP** | 实时价、PE/PB/市值/换手率/涨跌停、指数、ETF |
| **3** | 新浪 / 巨潮 / 同花顺 | HTTP | 低 | 财报三表、公告、一致预期/热点 |
| **4（仅独有数据才用）** | **东财 eastmoney** | HTTP | **有风控，会封 IP** | 见下 |

**凡是行情 / K线 / 实时价 / 市值 / 财务三表能从 mootdx 或腾讯拿到的，一律走它们**——TCP 协议和腾讯接口实测不封 IP，可放心高频调用。

### 东财只用于它「独有、别处拿不到」的数据

下列数据**只有东财有**，通达信/腾讯/新浪都没有，必须用东财（但要限流）：

> 龙虎榜席位 · 全市场龙虎榜 · 限售解禁日历 · 个股资金流向（分钟/日级）· 行业板块排名 · 个股新闻

### 东财风控阈值（社区实测，2026-05）

| 行为 | 触发封禁的阈值 | 风险 |
|------|---------------|------|
| 每秒请求数 | > 5 次/秒 | 高 |
| 单 IP 并发连接 | ≥ 10 | 高 |
| 1 分钟请求总数 | ≥ 200 次 | 中高 |
| 5 分钟请求总数 | ≥ 300 次 | 触发封禁 |
| User-Agent | 空 UA / 无浏览器特征 | 中 |

被封表现：连续请求后 `403` / `429` / 连接超时 / 返回空数据。临时封禁通常几分钟到几小时。

### 防封铁律（调用东财时必须遵守）

1. **串行，不并发**——绝不对东财开多线程/协程并发请求
2. **每次间隔 ≥ 1 秒 + 随机抖动**（QPS ≤ 2），批量筛选时调大到 1.5~2 秒
3. **复用 HTTP 会话**（Keep-Alive），不要每次新建连接
4. **带正常 UA + Referer**（本 SKILL 各端点已配好）
5. **批量场景每只股票之间 sleep**——AI 跑批量循环（如筛选 100 只股逐个拉龙虎榜/资金流）是被封的头号元凶

### 已内置限流：所有东财请求走 `em_get()`

本 SKILL 提供统一的节流入口 `em_get()`（定义见下方「东财数据中心统一查询（共用 helper）」），它自动做到：串行限流（最小间隔 `EM_MIN_INTERVAL=1.0s` + 随机抖动）+ 复用 `EM_SESSION`（Keep-Alive）+ 默认 UA。**所有 `eastmoney.com` 端点的代码块都已改用 `em_get` 而非裸 `requests.get`**，AI 直接抄代码即自带防封。批量任务把 `EM_MIN_INTERVAL` 调大即可进一步降速。

> 注：`em_get` / `EM_SESSION` / `EM_MIN_INTERVAL` 是所有东财代码块共用的前置定义，使用任一东财端点前需先执行「共用 helper」代码块。

---

## When to Activate

- 用户要**给个股做短线体检 / 战法打分**（能不能买、该不该卖、是否回避）→ `stock_diagnosis()`
- 用户问**均线/生命线/季线**（站没站上 5 日线、有没有跌破生命线、5日线死叉生命线）
- 用户问**量能**（放量没、量能金叉死叉、是不是缩量涨停、持续放量）
- 用户问**K线形态**（涨停、一字板、光头大阳、红三兵、跳空缺口、十字星、高位放量收阴）
- 用户问**技术指标**（MACD、威廉指标 WR、RSI）
- 用户要看**大盘多空环境**（先看大盘再选股）→ `market_regime()`
- 用户要做**周线确认**（周线方向、三线合一）
- 用户要拉实时行情（价格 / 五档盘口 / K线 / 涨跌停价 / 量比 / 换手率）
- 用户要看**当日强势股 / 题材归因 / 概念热点** → 选股
- 用户要看**概念板块归属**（题材联想，规则23）
- 用户要看**个股资金流向**（主力/超大单/大单，验证放量真假）
- 用户要看**龙虎榜席位 / 全市场龙虎榜**（游资席位、净买额排名）
- 用户要看**限售解禁日历**（短线避雷）
- 用户要做**行业板块强弱 / 轮动**（板块联动，规则14）
- 用户要看**指数/ETF行情**
- 用户要看个股新闻 / 查公告（消息面避雷）
- 关键词：**体检、打分、能不能买、该不该卖、回避、强势股、打板、涨停、一字板、光头大阳、红三兵、跳空、十字星、均线、五日线、生命线、24日线、季线、60日线、死叉、金叉、均量线、放量、缩量、持续放量、量比、换手率、MACD、威廉、WR、RSI、大盘、周线、三线合一、题材、热点、概念归因、概念板块、资金流向、主力、超大单、龙虎榜、席位、营业部、净买入、解禁、限售、行业轮动、板块联动、指数、ETF**

---

## Prerequisites

```bash
pip install mootdx requests pandas stockstats
```

| 依赖 | 版本要求 | 用途 |
|------|---------|------|
| mootdx | >= 0.10 | TCP行情+财务+F10（唯一非HTTP依赖） |
| requests | any | 所有HTTP API直连 |
| pandas | any | 数据处理+HTML表格解析 |
| stockstats | any | 技术指标计算（RSI/MACD/BOLL等） |

> **架构：** 除 mootdx（TCP 二进制协议）外，所有数据源均为直连 HTTP API，零第三方数据封装依赖。战法计算层（均线/量能/形态/指标）基于 mootdx 数据用 pandas/stockstats 本地计算。

### 市场前缀规则（全局通用）

```python
def get_prefix(code: str) -> str:
    """6位代码 → 市场前缀"""
    if code.startswith(("6", "9")):
        return "sh"
    elif code.startswith("8"):
        return "bj"
    else:
        return "sz"
```

### Ticker 格式归一化

所有接口统一支持多种输入格式，内部归一化为纯 6 位数字：

| 输入 | 归一化结果 |
|------|-----------|
| `688017` | `688017` |
| `SH688017` / `sh688017` | `688017` |
| `688017.SH` / `688017.sh` | `688017` |
| `SZ000001` | `000001` |
| `BJ832000` | `832000` |

### 东财数据中心统一查询（共用 helper）

龙虎榜 / 全市场龙虎榜 / 限售解禁 共用同一 base URL：

```python
import time
import random
import requests

UA = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36"
DATACENTER_URL = "https://datacenter-web.eastmoney.com/api/data/v1/get"

# ── 东财防封：全局节流 + 会话复用 ────────────────────────────────────
# 东财系 HTTP 接口（push2 / datacenter / reportapi / search / np-weblist）有风控：
#   每秒 >5 次 / 单 IP 并发 ≥10 / 1 分钟 ≥200 次  →  临时封 IP。
# 所有 eastmoney.com 请求一律走 em_get()：串行限流（最小间隔 + 随机抖动）+ 复用
# Keep-Alive 会话，批量调用时自动降速，避免被封。详见「数据源优先级 & 东财防封」章节。
EM_SESSION = requests.Session()
EM_SESSION.headers.update({"User-Agent": UA})
EM_MIN_INTERVAL = 1.0          # 两次东财请求最小间隔(秒)；批量筛选建议调大到 1.5~2
_em_last_call = [0.0]          # 模块级上次请求时间戳

def em_get(url: str, params: dict | None = None, headers: dict | None = None,
           timeout: int = 15, **kwargs):
    """东财统一请求入口：自动节流 + 复用 session + 默认 UA。
    所有 eastmoney.com 接口都应通过它请求，避免高频被封 IP。"""
    wait = EM_MIN_INTERVAL - (time.time() - _em_last_call[0])
    if wait > 0:
        time.sleep(wait + random.uniform(0.1, 0.5))
    try:
        return EM_SESSION.get(url, params=params, headers=headers, timeout=timeout, **kwargs)
    finally:
        _em_last_call[0] = time.time()

def eastmoney_datacenter(report_name: str, columns: str = "ALL",
                          filter_str: str = "", page_size: int = 50,
                          sort_columns: str = "", sort_types: str = "-1") -> list[dict]:
    """东财数据中心统一查询 — 龙虎榜/全市场龙虎榜/限售解禁 共用（已内置限流）"""
    params = {
        "reportName": report_name, "columns": columns,
        "filter": filter_str, "pageNumber": "1", "pageSize": str(page_size),
        "sortColumns": sort_columns, "sortTypes": sort_types,
        "source": "WEB", "client": "WEB",
    }
    r = em_get(DATACENTER_URL, params=params, timeout=15)
    d = r.json()
    if d.get("result") and d["result"].get("data"):
        return d["result"]["data"]
    return []
```

---

## Layer 1: 行情层（实时，不封IP）

### 1.1 mootdx — K线 + 五档盘口 + 逐笔成交

TCP 二进制协议，连通达信服务器(7709)，无需注册，不封IP。

```python
from mootdx.quotes import Quotes

client = Quotes.factory(market='std')

# === K线数据 ===
# market: 0=深圳, 1=上海
# category: 4=日线, 5=周线, 6=月线, 7=1分钟, 8=5分钟, 9=15分钟, 10=30分钟, 11=60分钟
klines = client.bars(symbol='688017', category=4, offset=10)
# 返回: open, close, high, low, vol, amount, datetime

# === 实时报价 ===
quotes = client.quotes(symbol=['688017', '300476'])
# 返回 46 个字段:
#   price(现价), open, high, low, last_close(昨收)
#   bid1~bid5, ask1~ask5, bid_vol1~bid_vol5, ask_vol1~ask_vol5
#   vol(成交量), amount(成交额), servertime

# === 逐笔成交（非交易时间返回空）===
trades = client.transaction(symbol='688017', date='20260502')
# 返回: time, price, vol, num, buyorsell(0买/1卖/2中性)
```

**mootdx 不提供 PE / PB / 市值 / 换手率 / 涨跌停价** — 这些走腾讯财经。

### 1.2 腾讯财经 API — PE/PB/市值/换手率/涨跌停/指数/ETF

HTTP GET，GBK 编码，`~` 分隔 88 个字段，不封IP。

```python
import urllib.request

def tencent_quote(codes: list[str]) -> dict[str, dict]:
    """
    批量拉取腾讯财经实时行情。
    codes: ["688017", "300476", "002463"]
    也支持指数: ["000001", "000300", "399006"]
    也支持ETF: ["510050", "510300"]
    返回: {code: {name, price, pe_ttm, pb, mcap, ...}}
    """
    prefixed = []
    for c in codes:
        if c.startswith(("6", "9")):
            prefixed.append(f"sh{c}")
        elif c.startswith("8"):
            prefixed.append(f"bj{c}")
        else:
            prefixed.append(f"sz{c}")

    url = "https://qt.gtimg.cn/q=" + ",".join(prefixed)
    req = urllib.request.Request(url)
    req.add_header("User-Agent", "Mozilla/5.0")
    resp = urllib.request.urlopen(req, timeout=10)
    data = resp.read().decode("gbk")

    result = {}
    for line in data.strip().split(";"):
        if not line.strip() or "=" not in line or '"' not in line:
            continue
        key = line.split("=")[0].split("_")[-1]
        vals = line.split('"')[1].split("~")
        if len(vals) < 53:
            continue
        code = key[2:]
        result[code] = {
            "name":         vals[1],
            "price":        float(vals[3]) if vals[3] else 0,
            "last_close":   float(vals[4]) if vals[4] else 0,
            "open":         float(vals[5]) if vals[5] else 0,
            "change_amt":   float(vals[31]) if vals[31] else 0,
            "change_pct":   float(vals[32]) if vals[32] else 0,
            "high":         float(vals[33]) if vals[33] else 0,
            "low":          float(vals[34]) if vals[34] else 0,
            "amount_wan":   float(vals[37]) if vals[37] else 0,
            "turnover_pct": float(vals[38]) if vals[38] else 0,
            "pe_ttm":       float(vals[39]) if vals[39] else 0,
            "amplitude_pct":float(vals[43]) if vals[43] else 0,
            "mcap_yi":      float(vals[44]) if vals[44] else 0,
            "float_mcap_yi":float(vals[45]) if vals[45] else 0,
            "pb":           float(vals[46]) if vals[46] else 0,
            "limit_up":     float(vals[47]) if vals[47] else 0,
            "limit_down":   float(vals[48]) if vals[48] else 0,
            "vol_ratio":    float(vals[49]) if vals[49] else 0,
            "pe_static":    float(vals[52]) if vals[52] else 0,
        }
    return result

# 用法: 个股
quotes = tencent_quote(["688017", "300476", "002463"])
for code, q in quotes.items():
    print(f"{q['name']}({code}): {q['price']}元 PE={q['pe_ttm']} PB={q['pb']} 市值={q['mcap_yi']}亿")

# 用法: 指数 — sh000001=上证指数, sh000300=沪深300, sz399006=创业板指
index_quotes = tencent_quote(["000001", "000300", "399006"])

# 用法: ETF — sh510050=上证50ETF, sh510300=沪深300ETF
etf_quotes = tencent_quote(["510050", "510300"])
```

#### 腾讯财经字段索引速查（实测校准 2026-05-03）

| 索引 | 含义 | 示例 |
|------|------|------|
| 1 | 名称 | 绿的谐波 |
| 3 | 当前价 | 224.12 |
| 4 | 昨收 | 215.01 |
| 5 | 今开 | 214.10 |
| 9-18 | 买一~买五(价+量) | |
| 19-28 | 卖一~卖五(价+量) | |
| 31 | 涨跌额 | 9.11 |
| 32 | 涨跌幅% | 4.24 |
| 33 | 最高 | 229.62 |
| 34 | 最低 | 214.10 |
| 37 | 成交额(万) | 187040 |
| 38 | 换手率% | 4.55 |
| **39** | **PE(TTM)** | 300.45 |
| **43** | **振幅%（不是PB！）** | 7.22 |
| **44** | **总市值(亿)** | 410.88 |
| **45** | **流通市值(亿)** | 410.88 |
| **46** | **PB(市净率)** | 11.51 |
| **47** | **涨停价** | 258.01 |
| **48** | **跌停价** | 172.01 |
| 49 | 量比 | 1.20 |
| **52** | **PE(静)** | 314.76 |

> **踩坑提醒：** 网上很多教程把索引 43 写成 PB，实测是振幅%。PB 在索引 46。

### 1.3 百度股市通 K线 — 带MA5/MA10/MA20（V3.0 新增）

**核心价值：** 返回时自带均线数据，无需本地计算。

```python
import requests

def baidu_kline_with_ma(code: str, start_time: str = "") -> dict:
    """百度股市通K线 — 独有能力: 返回时自带 ma5/ma10/ma20 均价"""
    url = "https://finance.pae.baidu.com/selfselect/getstockquotation"
    params = {
        "all": "1", "isIndex": "false", "isBk": "false", "isBlock": "false",
        "isFutures": "false", "isStock": "true", "newFormat": "1",
        "group": "quotation_kline_ab", "finClientType": "pc",
        "code": code, "start_time": start_time, "ktype": "1",
    }
    headers = {
        "User-Agent": "Mozilla/5.0",
        "Accept": "application/vnd.finance-web.v1+json",
        "Origin": "https://gushitong.baidu.com",
        "Referer": "https://gushitong.baidu.com/",
    }
    r = requests.get(url, params=params, headers=headers, timeout=10)
    d = r.json()
    result = d.get("Result", {})
    md = result.get("newMarketData", {})
    keys = md.get("keys", [])  # includes: ma5avgprice, ma10avgprice, ma20avgprice
    rows = md.get("marketData", "").split(";")
    return {"keys": keys, "rows": rows}

# 用法
data = baidu_kline_with_ma("600519")
print("字段:", data["keys"][:10])
print("最近5根K线:", data["rows"][-5:])
# keys 包含: time, open, close, high, low, volume, amount, ma5avgprice, ma10avgprice, ma20avgprice 等
```

---
---

## Layer 2: 战法计算层（核心 — 短线战法的"大脑"）

> 行情层只提供**原始 K 线**，本层把它转成战法真正要看的东西：均线位置、量能金叉死叉、K线形态、技术指标、大盘环境、周线确认，并最终由 `stock_diagnosis()` 按 100 条战法逐条打分，输出 **买入候选 / 持有 / 卖出 / 回避**。
>
> 依赖：`pandas`（rolling 算均线均量）+ `stockstats`（MACD/威廉/RSI）。数据全部来自 mootdx（TCP，不封 IP）。

```python
# -*- coding: utf-8 -*-
import pandas as pd
from mootdx.quotes import Quotes

_CLIENT = None
def _client():
    global _CLIENT
    if _CLIENT is None:
        _CLIENT = Quotes.factory(market='std')
    return _CLIENT

def normalize(code: str) -> str:
    """SH600879 / 600879.SH / sz000001 → 纯6位"""
    c = str(code).upper().replace("SH", "").replace("SZ", "").replace("BJ", "")
    return c.split(".")[0].strip().zfill(6)

def limit_pct(code: str, name: str = "") -> float:
    """涨停阈值%：ST=5，科创/创业(688/300/301)=20，其余=10"""
    if "ST" in (name or ""):
        return 5.0
    if code.startswith(("688", "300", "301")):
        return 20.0
    return 10.0

def get_daily(code: str, n: int = 70) -> pd.DataFrame:
    """日线 + 均线(MA5/10/20/24/60) + 均量线(VMA5/10/15)。MA24=生命线，MA60=季线。"""
    code = normalize(code)
    df = _client().bars(symbol=code, category=4, offset=n)
    if df is None or len(df) == 0:
        return pd.DataFrame()
    df = df.reset_index(drop=True).copy()
    df['date'] = df['datetime'].astype(str).str[:10]
    for m in (5, 10, 20, 24, 60):
        df[f'MA{m}'] = df['close'].rolling(m).mean()
    for m in (5, 10, 15):
        df[f'VMA{m}'] = df['vol'].rolling(m).mean()
    df['chg'] = df['close'].pct_change() * 100
    return df

def _slope(df, col, look=3):
    """最近 look 日首尾比较 → up/down/flat"""
    if len(df) <= look or pd.isna(df.iloc[-1-look][col]) or pd.isna(df.iloc[-1][col]):
        return "flat"
    a, b = df.iloc[-1-look][col], df.iloc[-1][col]
    if b > a * 1.002: return "up"
    if b < a * 0.998: return "down"
    return "flat"

# ── 缺1：均线分析（规则18生命线 / 26·51五日线 / 32死叉 / 88季线）──
def compute_ma(df: pd.DataFrame) -> dict:
    last = df.iloc[-1]; price = last['close']
    out = {
        "price": round(price, 2),
        "MA5": round(last['MA5'], 2), "MA10": round(last['MA10'], 2),
        "MA24_lifeline": round(last['MA24'], 2), "MA60_season": round(last['MA60'], 2),
        "above_MA5": bool(price >= last['MA5']),
        "above_MA24": bool(price >= last['MA24']),      # 生命线：多空总分界
        "above_MA60": bool(price >= last['MA60']),      # 季线
        "MA5_slope": _slope(df, 'MA5'), "MA24_slope": _slope(df, 'MA24'),
        "MA5_above_MA24": bool(last['MA5'] >= last['MA24']),
        "bull_align": bool(last['MA5'] >= last['MA10'] >= last['MA24']),
        "deviate_MA5_pct": round((price / last['MA5'] - 1) * 100, 2),  # 偏离5日线 规则31
    }
    cross = None  # 近10日内 5日线与生命线的金叉/死叉
    for i in range(max(1, len(df)-10), len(df)):
        p, c = df.iloc[i-1], df.iloc[i]
        if pd.isna(p['MA24']) or pd.isna(c['MA24']): continue
        if p['MA5'] >= p['MA24'] and c['MA5'] < c['MA24']:
            cross = ("dead_cross", c['date'])
        elif p['MA5'] <= p['MA24'] and c['MA5'] > c['MA24']:
            cross = ("golden_cross", c['date'])
    out["MA5_x_MA24"] = cross
    return out

# ── 缺2：量能分析（规则04金叉 / 07·62死叉 / 30平行 / 37·63·84持续放量）──
def compute_vol_ma(df: pd.DataFrame) -> dict:
    last = df.iloc[-1]
    vcross = None
    for i in range(max(1, len(df)-10), len(df)):
        p, c = df.iloc[i-1], df.iloc[i]
        if pd.isna(p['VMA10']) or pd.isna(c['VMA10']): continue
        if p['VMA5'] <= p['VMA10'] and c['VMA5'] > c['VMA10']:
            vcross = ("golden_cross", c['date'])
        elif p['VMA5'] >= p['VMA10'] and c['VMA5'] < c['VMA10']:
            vcross = ("dead_cross", c['date'])
    recent3 = df['vol'].iloc[-3:].mean(); prev3 = df['vol'].iloc[-6:-3].mean()
    return {
        "vol": round(last['vol']/1e4, 0), "VMA5": round(last['VMA5']/1e4, 0),
        "VMA10": round(last['VMA10']/1e4, 0),
        "VMA5_above_VMA10": bool(last['VMA5'] >= last['VMA10']),
        "VMA5_slope": _slope(df, 'VMA5'),
        "VMA5_x_VMA10": vcross,
        "sustained_volume": bool(recent3 > prev3 * 1.05),  # 持续放量
        "vol_ratio_3d": round(recent3/prev3, 2) if prev3 else 0,
    }

# ── 缺3：K线形态（规则20·21·22·41·61·70·96）──
def detect_patterns(df: pd.DataFrame, code: str, name: str = "") -> dict:
    lp = limit_pct(code, name); pats = []
    last = df.iloc[-1]
    o, c, h, l = last['open'], last['close'], last['high'], last['low']
    chg = last['chg']; rng = (h - l) or 1e-9
    if chg >= lp - 0.3:
        pats.append("涨停")
        if abs(h - l) / (o or 1) < 0.005: pats.append("一字板")
    if c > o and (h - c) / (c or 1) < 0.003 and chg > 5: pats.append("光头大阳")  # 规则22
    if abs(c - o) / (o or 1) < 0.003: pats.append("十字星")                       # 规则41
    if (min(o, c) - l) / rng > 0.5: pats.append("长下影线")                       # 规则70
    if len(df) >= 2 and l > df.iloc[-2]['high']: pats.append("向上跳空缺口")       # 规则21/51
    if len(df) >= 3:                                                              # 规则61/96 红三兵
        a, b, cc = df.iloc[-3], df.iloc[-2], df.iloc[-1]
        if all(x['close'] > x['open'] for x in (a, b, cc)) and a['close'] < b['close'] < cc['close']:
            pats.append("红三兵")
    if c < o and last['vol'] > last['VMA5'] * 1.3:                               # 规则12 高位放量收阴
        if last['close'] > df['close'].iloc[-40:].max() * 0.85:
            pats.append("高位放量收阴")
    return {"patterns": pats, "limit_pct": lp, "today_chg": round(chg, 2)}

# ── 缺4：技术指标（规则35 MACD / 99 威廉<50强势）──
def compute_indicators(df: pd.DataFrame) -> dict:
    try:
        from stockstats import wrap
        sdf = wrap(df[['open','close','high','low','vol']].rename(columns={'vol':'volume'}).copy())
        macd = float(sdf['macd'].iloc[-1]); macds = float(sdf['macds'].iloc[-1])
        return {
            "MACD": round(macd, 3), "MACD_signal": round(macds, 3),
            "MACD_above_signal": bool(macd > macds),
            "WR6": round(float(sdf['wr_6'].iloc[-1]), 1),
            "WR_strong": bool(float(sdf['wr_6'].iloc[-1]) < 50),
            "RSI6": round(float(sdf['rsi_6'].iloc[-1]), 1),
        }
    except Exception as e:
        return {"error": f"{type(e).__name__}: {e}"}

# ── 缺5b：主力/散户资金流（规则64/76 资金验证）──
# 注意：东财资金流历史接口 push2his.eastmoney.com 在部分网络被阻断；
# 改用 push2.eastmoney.com 的 fflow/kline/get + klt=101（日级），同样给主力/超大/大/中/小单净额。
def fund_flow_em(code: str, lmt: int = 5) -> list:
    """当日及近 lmt 日主力/散户资金流（万元）。走 push2（连通性比 push2his 好）。
    返回 [{date, 主力净, 超大单, 大单, 中单, 小单}]，单位万元。失败返回 []。"""
    import requests
    _ua = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    code = normalize(code)
    secid = f"1.{code}" if code.startswith(("6", "9")) else f"0.{code}"
    url = "https://push2.eastmoney.com/api/qt/stock/fflow/kline/get"
    params = {"secid": secid, "klt": "101",
              "fields1": "f1,f2,f3,f7", "fields2": "f51,f52,f53,f54,f55,f56,f57"}
    # 东财资金流接口在部分网络间歇可达，重试若干次提高成功率；全失败则返回 []（不影响体检主流程）
    import time as _t
    sess = requests.Session(); sess.trust_env = False   # 绕过系统坏代理
    kl = []
    for _i in range(4):
        try:
            r = sess.get(url, params=params, timeout=10,
                         headers={"User-Agent": _ua, "Referer": "https://quote.eastmoney.com/"},
                         proxies={"http": None, "https": None})
            kl = r.json().get("data", {}).get("klines", []) or []
            if kl:
                break
        except Exception:
            _t.sleep(0.6 + _i * 0.4)
    if not kl:
        return []
    rows = []
    for ln in kl[-lmt:]:
        p = ln.split(",")
        if len(p) >= 6:
            rows.append({"date": p[0],
                         "主力净万": round(float(p[1])/1e4, 0), "超大单万": round(float(p[5])/1e4, 0),
                         "大单万": round(float(p[4])/1e4, 0), "中单万": round(float(p[3])/1e4, 0),
                         "小单万": round(float(p[2])/1e4, 0)})
    return rows

# ── 缺5c：量价异动 + 连板语义（吸收自 TradingAgents 游资追踪框架）──
def volume_anomaly(df: pd.DataFrame, code: str, name: str = "") -> dict:
    """量价异动识别 + 连板放缩量语义。
    判据（游资框架）：日量>20日均量2倍=放量异动；换手率>10%=异常活跃（需腾讯，缺则跳过）；
    连板：首板放量=分歧/缩量=一致；三板以上=妖股模式。"""
    last = df.iloc[-1]
    lp = limit_pct(code, name)
    vol20 = df['vol'].iloc[-20:].mean()
    vol_x = round(last['vol'] / vol20, 2) if vol20 else 0
    out = {"量比20日": vol_x, "放量异动": bool(vol_x >= 2.0)}
    # 连续涨停板数（从最后一根往前数连续涨停）
    boards = 0
    for i in range(len(df)-1, 0, -1):
        chg = (df.iloc[i]['close']/df.iloc[i-1]['close'] - 1) * 100
        if chg >= lp - 0.3:
            boards += 1
        else:
            break
    out["连板数"] = boards
    # 连板语义
    if boards >= 1:
        # 最近一个涨停日的量 vs 前一日量：放量=分歧，缩量=一致
        lim_vol = df.iloc[-1]['vol']; prev_vol = df.iloc[-2]['vol'] if len(df) >= 2 else lim_vol
        seal = "放量(分歧)" if lim_vol > prev_vol * 1.1 else ("缩量(一致)" if lim_vol < prev_vol * 0.9 else "平量")
        if boards >= 3:
            out["连板语义"] = f"{boards}连板·三板以上进入妖股模式需特别谨慎（规则09/76）"
        elif boards == 1:
            out["连板语义"] = f"首板·{seal}（缩量更安全，规则24/50）"
        else:
            out["连板语义"] = f"{boards}连板·{seal}"
    else:
        out["连板语义"] = ""
    return out

# ── 缺5d：资金博弈四态（吸收自 TradingAgents：主力吸筹/出货/游资接力/散户主导）──
def capital_battle(code: str, df: pd.DataFrame = None, name: str = "") -> dict:
    """综合主力净/散户净 + 量价，判定当前资金博弈格局四态之一。
    依赖 fund_flow_em（间歇可达）；拿不到资金流则返回 {态:'[数据缺失:资金流]'}。"""
    code = normalize(code)
    if df is None:
        df = get_daily(code, 70)
    ff = fund_flow_em(code, lmt=3)
    if not ff:
        return {"态": "[数据缺失:资金流]", "说明": "push2 资金流接口未取到，无法判定主力/散户格局"}
    f0 = ff[-1]
    main = f0["主力净万"]; retail = f0["中单万"] + f0["小单万"]
    last = df.iloc[-1]; chg = last['chg'] if 'chg' in df.columns else 0
    up = chg > 0
    # 四态判定
    if main > 0 and not up:
        state = "主力吸筹"; note = "主力净流入但股价未涨——疑似低位吸筹"
    elif main < 0 and up:
        state = "散户主导(诱多)"; note = "股价涨但主力净流出——散户接盘/拉高出货嫌疑"
    elif main < 0 and not up:
        state = "主力出货"; note = "主力净流出且股价下跌——出货特征"
    elif main > 0 and up:
        # 主力流入+上涨：看超大单是否主导，区分机构 vs 游资接力
        state = "主力推动" if f0["超大单万"] > 0 else "游资接力"
        note = "主力净流入推动上涨" if f0["超大单万"] > 0 else "大单为主、超大单不足——游资接力特征"
    else:
        state = "资金博弈"; note = "多空均衡"
    return {"态": state, "说明": note, "主力净万": main, "散户净万": round(retail, 0), "超大单万": f0["超大单万"]}

# ── 缺5：大盘环境（规则33先看大盘 / 62空头配合出货 / 97暴跌找逆势强势）──
def market_regime() -> dict:
    res = {}
    for idx, nm in [("999999","上证"), ("399001","深成"), ("399006","创业板")]:
        try:
            df = _client().index(symbol=idx, category=4, offset=30)  # 指数走 index() 不是 bars()
            df = df.reset_index(drop=True)
            for m in (5,10,20): df[f'MA{m}'] = df['close'].rolling(m).mean()
            last = df.iloc[-1]
            if last['close']>last['MA5']>last['MA10']>last['MA20']: res[nm]="bull"
            elif last['close']<last['MA5']<last['MA10']: res[nm]="bear"
            else: res[nm]="neutral"
        except Exception as e:
            res[nm] = f"err:{type(e).__name__}"
    bulls = sum(1 for v in res.values() if v=="bull")
    bears = sum(1 for v in res.values() if v=="bear")
    res["overall"] = "bull" if bulls>=2 else ("bear" if bears>=2 else "neutral")
    return res

# ── 缺6：周线确认（规则70·71三线合一 / 95买入前看周线）──
def weekly_view(code: str) -> dict:
    code = normalize(code)
    try:
        df = _client().bars(symbol=code, category=5, offset=30)  # 周线
        df = df.reset_index(drop=True)
        for m in (5,10,20): df[f'MA{m}'] = df['close'].rolling(m).mean()
        last = df.iloc[-1]
        slope = "up" if df.iloc[-1]['MA5'] > df.iloc[-3]['MA5'] else "down"
        spread = abs(last['MA5']-last['MA10'])/last['close'] + abs(last['MA10']-last['MA20'])/last['close']
        return {"weekly_MA5_slope": slope, "three_line_converge": bool(spread < 0.03)}
    except Exception as e:
        return {"error": f"{type(e).__name__}"}

# ── 缺7：涨停板质量（规则05·17·52，盘中尽力而为）──
def limit_up_quality(code: str, name: str = "") -> dict:
    code = normalize(code)
    try:
        q = _client().quotes(symbol=[code])
        if q is None or len(q) == 0: return {"note": "无盘口"}
        r = q.iloc[0]
        return {"price": float(r.get('price',0)), "bid1_vol": float(r.get('bid_vol1',0)),
                "note": "封单量=买一挂单；封停时长需逐笔，盘后不可得"}
    except Exception as e:
        return {"error": f"{type(e).__name__}"}

# ── 缺9：基本面/消息面避雷（纯技术战法的补充：盈利+估值+妖股顶部+新闻）──
def risk_check(code: str, name: str = "", df=None) -> dict:
    """避雷层：只提示风险，不改技术结论。
    - 亏损：mootdx 财务快照净利润(jinglirun)为负
    - 估值异常：腾讯 PE(负或>200)
    - ST：名称含 ST
    - 妖股顶部：近10日涨停≥3次 或 近20日涨幅≥50% 或 天量倍数≥4（规则09/44/76/88）
    - 新闻：东财个股新闻（本机网络不稳，失败标注未取到，不影响结论）
    """
    import urllib.request, json as _json, re as _re
    code = normalize(code)
    warns = []
    info = {}

    # 1) 亏损 — mootdx 财务快照
    try:
        fin = _client().finance(symbol=code)
        if fin is not None and len(fin):
            np_ = float(fin.iloc[0].get("jinglirun", 0) or 0)      # 净利润
            rev = float(fin.iloc[0].get("zhuyingshouru", 0) or 0)  # 主营收入
            info["net_profit_yi"] = round(np_/1e8, 2)
            info["revenue_yi"] = round(rev/1e8, 2)
            if np_ < 0:
                warns.append(f"⚠️亏损：最新季报净利润 {np_/1e8:.2f}亿（爆炒亏损股风险）")
    except Exception:
        info["finance"] = "未取到"

    # 2) 估值 — 腾讯 PE/PB
    try:
        pre = "sh" if code.startswith(("6","9")) else ("bj" if code.startswith("8") else "sz")
        u = f"https://qt.gtimg.cn/q={pre}{code}"
        rq = urllib.request.Request(u); rq.add_header("User-Agent", "Mozilla/5.0")
        v = urllib.request.urlopen(rq, timeout=8).read().decode("gbk").split('"')[1].split("~")
        pe = float(v[39]) if v[39] else 0; pb = float(v[46]) if len(v) > 46 and v[46] else 0
        info["PE_TTM"] = pe; info["PB"] = pb
        if pe < 0:
            warns.append(f"⚠️估值：PE为负（{pe}），公司当前亏损")
        elif pe > 200:
            warns.append(f"⚠️估值：PE高达 {pe:.0f} 倍，估值极高")
    except Exception:
        info["valuation"] = "未取到"

    # 3) ST
    if "ST" in (name or ""):
        warns.append("⚠️ST股：有退市/风险警示，波动剧烈")

    # 4) 妖股顶部（规则09/44/76/88）
    try:
        if df is None:
            df = get_daily(code, 70)
        lp = limit_pct(code, name)
        chg10 = [(df.iloc[i]['close']/df.iloc[i-1]['close']-1)*100 for i in range(len(df)-10, len(df))]
        limitups = sum(1 for c in chg10 if c >= lp - 0.3)
        rise20 = (df.iloc[-1]['close']/df.iloc[-21]['close'] - 1) * 100
        volmax = df['vol'].iloc[-10:].max(); volbase = df['vol'].iloc[-30:-10].mean()
        spike = (volmax/volbase) if volbase else 0
        # 连板语义（游资框架）：连续涨停板数
        va = volume_anomaly(df, code, name)
        boards = va.get("连板数", 0)
        info.update({"近10日涨停": limitups, "近20日涨幅%": round(rise20, 0),
                     "天量倍数": round(spike, 1), "连板数": boards, "连板语义": va.get("连板语义", "")})
        # 三板以上=妖股模式（游资框架，比单看涨停次数更准）
        if boards >= 3:
            warns.append(f"🔥妖股模式：{boards}连板·三板以上爆炒——崩塌风险极高，坚决回避（规则09/76）")
        elif limitups >= 3 or rise20 >= 50 or spike >= 4:
            warns.append(f"🔥妖股顶部风险：近10日涨停{limitups}次/20日涨{rise20:.0f}%/天量{spike:.1f}倍——警惕爆炒后崩塌（规则09/44/76/88）")
    except Exception:
        pass

    # 5) 新闻（尽力而为，本机网络不稳）
    try:
        inner = _json.dumps({"uid":"","keyword":code,"type":["cmsArticleWebOld"],"client":"web",
            "clientType":"web","clientVersion":"curr","param":{"cmsArticleWebOld":{"searchScope":"default",
            "sort":"default","pageIndex":1,"pageSize":3,"preTag":"","postTag":""}}}, separators=(',',':'))
        r = em_get("https://search-api-web.eastmoney.com/search/jsonp",
                   params={"cb":"jq","param":inner},
                   headers={"User-Agent": UA, "Referer":"https://so.eastmoney.com/"}, timeout=8)
        txt = r.text; dj = _json.loads(txt[txt.index("(")+1:txt.rindex(")")])
        arts = dj.get("result", {}).get("cmsArticleWebOld", []) or []
        info["最近新闻"] = [f"{a.get('date','')[:10]} {_re.sub(r'<[^>]+>','',a.get('title',''))}" for a in arts[:3]]
    except Exception:
        info["最近新闻"] = "未取到（本机网络对东财不稳，建议手动查看）"

    return {"风险提示": warns, "基本面": info}

# ── 缺10：人工规则提示（约30条靠经验的规则无法自动判定，按形态特征触发提醒）──
def manual_reminders(df, ma: dict, vol: dict, pat: dict, drawdown_pct: float, verdict: str) -> list:
    """根据形态特征，提示该结合哪些"人工经验类"战法规则思考。
    只提醒、不替用户下结论。drawdown_pct 为距40日高点回撤(负数)。"""
    tips = []
    # 规则93：暴跌后第一次反弹不轻易做 —— 近期出现过跌停 + 今日反弹
    try:
        lp = pat.get("limit_pct", 10)
        recent_chg = [(df.iloc[i]['close']/df.iloc[i-1]['close']-1)*100 for i in range(len(df)-8, len(df))]
        had_limitdown = any(c <= -(lp-0.3) for c in recent_chg[:-1])  # 之前几天有跌停
        today_up = recent_chg[-1] > 3
        if had_limitdown and today_up:
            tips.append("规则93：该股近期出现过跌停，今日为暴跌后的反弹——战法主张【不轻易做第一次反弹】，需人工确认暴跌原因")
    except Exception:
        pass
    # 规则27/28：涨停且偏离5日线过远 → 不追高
    if "涨停" in pat['patterns'] and ma.get('deviate_MA5_pct', 0) > 5:
        tips.append(f"规则27/28：今日涨停且偏离5日线 {ma['deviate_MA5_pct']}%，【不宜追高】；若追，次日高开3%即走")
    # 规则09/44：距高点不远且仍在涨 → 留意第几波，三波后出局
    if drawdown_pct > -5 and ma.get('MA5_slope') == "up":
        tips.append("规则09/44：股价接近阶段高位且仍在上行，需人工判断这是第几波上涨——【三波明显上涨后坚决出局】")
    # 规则41：高位十字星 → 顶部预警
    if "十字星" in pat['patterns'] and drawdown_pct > -8:
        tips.append("规则41：高位出现十字星，留意是否【顶部来临】（连续十字星更需警惕）")
    # 规则82/83：洗盘震仓判断（站上生命线但震荡大）
    if ma.get('above_MA24') and abs(ma.get('deviate_MA5_pct', 0)) > 4:
        tips.append("规则82/83：股价偏离均线较大、波动剧烈，需人工分辨是【洗盘震仓】还是【主升/出货】")
    # 买入候选类 → 提醒先看大盘、买后多看少动
    if verdict in ("强势买入候选", "买入候选"):
        tips.append("规则33：买入前先确认【大盘环境】；规则36：趋势确立后【多看少动】，规则34：短线持有2周内/超短3天内")
    # 规则92：弱势区反弹不抄底
    if not ma.get('above_MA24') and ma.get('MA5_slope') != "down":
        tips.append("规则92：生命线下方的反弹属弱势反抽，【不等反弹、不随便抄底】")
    return tips

# ── 缺11：一键战法体检（整合①–⑩，技术打分 + 基本面避雷 + 人工规则提示）──
def stock_diagnosis(code: str, name: str = "", with_market: bool = True) -> dict:
    code = normalize(code)
    df = get_daily(code, 70)
    if df.empty:
        return {"error": "无K线数据"}
    ma = compute_ma(df); vol = compute_vol_ma(df)
    pat = detect_patterns(df, code, name); ind = compute_indicators(df)
    wk = weekly_view(code)
    hi40 = df['close'].iloc[-40:].max()
    drawdown = round((ma['price']/hi40 - 1)*100, 1)

    buy, sell = [], []
    above5, above24 = ma['above_MA5'], ma['above_MA24']
    # 买入条款（多以"站上5日线"为前提，规则05·26·46·51）
    if above5: buy.append("26/51:站上5日线")
    if ma['MA5_above_MA24'] and ma['bull_align']: buy.append("18:多头排列")
    if above5 and vol['VMA5_above_VMA10'] and vol['VMA5_slope']=="up": buy.append("04/30:量金叉且均量向上")
    if above5 and vol['sustained_volume']: buy.append("37/63/84:持续放量配合上涨")
    if "涨停" in pat['patterns']: buy.append("05:涨停")
    if "向上跳空缺口" in pat['patterns']: buy.append("21/51:向上跳空")
    if "红三兵" in pat['patterns']: buy.append("61/96:红三兵")
    if above5 and ind.get("WR_strong"): buy.append("99:威廉<50强势")
    # 卖出/回避条款（规则07·12·32·39·88·90·92）
    if (ma['MA5_x_MA24'] and ma['MA5_x_MA24'][0]=="dead_cross"
            and not above24 and not ma['MA5_above_MA24']):
        sell.append(f"32:5日线死叉生命线({ma['MA5_x_MA24'][1]})")   # 仅当未收复生命线才算
    if not above24: sell.append("18/88:跌破生命线")
    if not ma['above_MA60']: sell.append("88:跌破季线")
    if ma['MA5_slope']=="down" and ma['MA24_slope']=="down": sell.append("90:均线空头向下发散")
    if "高位放量收阴" in pat['patterns']: sell.append("12/76:高位放量收阴")
    if ma['deviate_MA5_pct'] < -5: sell.append("31:大幅偏离5日线下方")
    if not above5 and vol['VMA5_x_VMA10'] and vol['VMA5_x_VMA10'][0]=="dead_cross":
        sell.append("07/62:均量线死叉(弱势)")
    # 结论（规则18：站上生命线是多空总分界）
    has_limit = "涨停" in pat['patterns']
    if not above24:
        verdict = "回避/卖出" if len(sell) >= 2 else "回避"
    elif has_limit and not sell:
        verdict = "强势买入候选"
    elif len(buy) >= 3 and not sell:
        verdict = "买入候选"
    elif above5 and ma['MA5_above_MA24'] and not sell:
        verdict = "持有/观察"
    else:
        verdict = "观望"

    # 避雷层：基本面/消息面/妖股顶部（不改技术信号，但可下调结论）
    risk = risk_check(code, name, df=df)
    is_yaogu = any(("妖股顶部" in w) or ("妖股模式" in w) for w in risk["风险提示"])
    is_loss = any("亏损" in w for w in risk["风险提示"])
    # 妖股顶部/妖股模式风险 → 买入候选一律降级为"高风险观望"（规则09/28/76/92）
    if is_yaogu and verdict in ("强势买入候选", "买入候选"):
        verdict = "高风险观望（妖股顶部）"

    # 人工规则提示：约30条靠经验的规则无法自动判定，按形态特征提醒用户结合思考
    reminders = manual_reminders(df, ma, vol, pat, drawdown, verdict)

    # 量价异动 + 连板语义（游资框架）
    va = volume_anomaly(df, code, name)
    # 资金面：主力/散户博弈四态（capital_battle，区分吸筹/出货/接力/散户主导）
    battle = capital_battle(code, df=df, name=name)

    return {
        "code": code, "name": name, "结论": verdict,
        "大盘": market_regime()["overall"] if with_market else None,
        "位置": {"距40日高点": f"{drawdown}%", "周线方向": wk.get("weekly_MA5_slope")},
        "均线": ma, "量能": vol, "K线形态": pat['patterns'],
        "量价异动": {"量比20日": va["量比20日"], "放量异动": va["放量异动"],
                   "连板数": va["连板数"], "连板语义": va["连板语义"]},
        "指标": {"MACD多头": ind.get("MACD_above_signal"), "威廉强势": ind.get("WR_strong"), "WR6": ind.get("WR6")},
        "买入信号": buy, "卖出回避信号": sell,
        "风险提示": risk["风险提示"], "基本面": risk["基本面"],
        "资金面": battle,
        "需人工判断": reminders,
    }

# ── 缺12：低吸选股 —— 战法真正的买点（涨停回踩 / 横盘突破），不追已涨停 ──
def find_pullback_buy(code: str, name: str = "") -> dict:
    """筛"可低吸位置"，而非追已涨停的票。对应战法规则：
    - 规则59/26：牛股第一次回调到5日线/生命线不破、缩量企稳 = 介入良机
    - 规则40/85：横盘整理(3~8天)后第一根放量、尚未涨停 = 突破前夜
    返回 {类型, 命中, 理由}；不命中返回 类型=None。
    专门排除"今日涨停、偏离5日线过远"的追高位置（规则27/28）。
    """
    code = normalize(code)
    df = get_daily(code, 70)
    if df.empty or len(df) < 30:
        return {"类型": None, "理由": "数据不足"}
    ma = compute_ma(df); vol = compute_vol_ma(df)
    lp = limit_pct(code, name)
    last = df.iloc[-1]
    chg = last['chg']
    today_limit = chg >= lp - 0.3

    # 前置硬条件：必须在生命线上方（多头土壤，规则18）+ 不亏损
    if not ma['above_MA24']:
        return {"类型": None, "理由": "生命线下方，不具备低吸土壤"}

    # 类型A：涨停回踩企稳 —— 近15日有过涨停，今日未涨停，回踩到5日线附近(±3%)缩量
    chg15 = [(df.iloc[i]['close']/df.iloc[i-1]['close']-1)*100 for i in range(len(df)-15, len(df))]
    had_limit = any(c >= lp - 0.3 for c in chg15[:-1])
    near_ma5 = abs(ma['deviate_MA5_pct']) <= 4            # 贴近5日线（±4%）
    not_dump = last['vol'] < vol['VMA5'] * 1.2            # 量能温和（不放巨量出货即可，回踩位健康标志）
    if (not today_limit) and had_limit and near_ma5 and not_dump and ma['MA5_above_MA24']:
        vt = "缩量" if last['vol'] < vol['VMA5'] * 0.9 else "温和量"
        return {"类型": "涨停回踩企稳", "命中": "规则59/26",
                "理由": f"近15日涨停过、今回踩5日线({ma['deviate_MA5_pct']:+.1f}%)且{vt}，牛股第一次回调介入位"}

    # 类型B：横盘突破前夜 —— 前期窄幅整理(波动<12%)，今日放量上涨但未涨停
    win = df['close'].iloc[-9:-1]                          # 前8日
    box_range = (win.max() - win.min()) / win.mean() * 100
    today_up = 2 < chg < (lp - 1)                          # 上涨但非涨停
    vol_up = last['vol'] > vol['VMA5'] * 1.3               # 放量
    if box_range < 12 and today_up and vol_up and ma['above_MA5']:
        return {"类型": "横盘突破前夜", "命中": "规则40/85",
                "理由": f"前8日窄幅整理({box_range:.1f}%)、今日放量+{chg:.1f}%突破未涨停，启动信号"}

    # 追高排除说明
    if today_limit:
        return {"类型": None, "理由": f"今日涨停(偏离5日线{ma['deviate_MA5_pct']:+.1f}%)，属追高位，非低吸（规则28）"}
    return {"类型": None, "理由": "当前非低吸位置"}

# 批量低吸扫描：传入候选代码列表，返回命中的低吸标的
def scan_pullback(codes: list) -> list:
    """对一批代码筛低吸位置。codes: [(code, name), ...] 或 [code, ...]"""
    out = []
    for item in codes:
        code, nm = (item if isinstance(item, (tuple, list)) else (item, ""))
        r = find_pullback_buy(code, nm)
        if r["类型"]:
            out.append({"code": normalize(code), "name": nm, **r})
    return out

```

### 用法

```python
# 一键体检（最常用）
d = stock_diagnosis("600879", "航天电子")
print(d["结论"])          # → 回避/卖出
print(d["卖出回避信号"])   # → ['18/88:跌破生命线', '88:跌破季线', '90:均线空头向下发散', ...]

d = stock_diagnosis("605277", "新亚电子")
print(d["结论"])          # → 强势买入候选（站上生命线+涨停+跳空+持续放量）

# 也可单独调用各计算函数
df = get_daily("600519")
print(compute_ma(df))         # 均线位置/斜率/死叉
print(compute_vol_ma(df))     # 均量线金叉死叉/持续放量
print(detect_patterns(df, "600519"))  # K线形态
print(market_regime())        # 大盘多空环境
```

---

## Layer 3: 信号层

### 3.1 同花顺热点 — 当日强势股 + 题材归因 reason tags（独家）

**核心价值：** 不只告诉你"哪些走强"，还告诉你**"为什么走强"** —— 同花顺编辑部人工运营的题材标签。

```python
import requests
import pandas as pd

def ths_hot_reason(date: str = None) -> pd.DataFrame:
    """
    同花顺当日强势股归因。
    date: 'YYYY-MM-DD' 格式，None=今天
    返回 DataFrame，含每只股票的题材标签 (reason)。

    实测: 73ms 拿到 ~125 只 + 完整字段
    """
    from datetime import date as _date
    if date is None:
        date = _date.today().strftime("%Y-%m-%d")

    url = (
        f"http://zx.10jqka.com.cn/event/api/getharden/"
        f"date/{date}/orderby/date/orderway/desc/charset/GBK/"
    )
    headers = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
            "Chrome/117.0.0.0 Safari/537.36"
        )
    }
    r = requests.get(url, headers=headers, timeout=10)
    data = r.json()
    if data.get("errocode", 0) != 0:
        raise RuntimeError(f"同花顺热点错误: {data.get('errormsg', '')}")

    rows = data.get("data") or []
    df = pd.DataFrame(rows)
    if df.empty:
        return df

    # 字段重命名（中文友好）
    rename_map = {
        "name": "名称", "code": "代码", "reason": "题材归因",
        "close": "收盘价", "zhangdie": "涨跌额", "zhangfu": "涨幅%",
        "huanshou": "换手率%", "chengjiaoe": "成交额",
        "chengjiaoliang": "成交量", "ddejingliang": "大单净量",
        "market": "市场",
    }
    df = df.rename(columns=rename_map)
    return df

# 用法
df = ths_hot_reason("2026-05-09")
print(f"当日强势股: {len(df)} 只")
print(df[["代码", "名称", "涨幅%", "题材归因"]].head(10))
```

#### 同花顺热点字段速查

| 原字段 | 中文 | 说明 |
|---|---|---|
| code | 代码 | 6 位股票代码 |
| name | 名称 | 简称 |
| **reason** | **题材归因** | **核心字段，人工运营 tags，如"算力租赁+Token工厂+AI政务"** |
| zhangfu | 涨幅% | 当日涨幅 |
| huanshou | 换手率% | 当日换手 |
| chengjiaoe | 成交额 | 元 |
| chengjiaoliang | 成交量 | 股 |
| ddejingliang | 大单净量 | 主力净流入指标 |
| close | 收盘价 | 元 |
| zhangdie | 涨跌额 | 元 |
| market | 市场 | 沪/深/北 |

### 3.3 百度股市通 — 概念板块归属

**核心价值：** 一次调用拿到个股所属的行业（申万一级/二级）、概念（多个）、地域三维分类，含当日涨跌幅。

```python
import requests

_BAIDU_PAE_HEADERS = {
    "Host": "finance.pae.baidu.com",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/117.0.0.0",
    "Accept": "application/vnd.finance-web.v1+json",
    "Origin": "https://gushitong.baidu.com",
    "Referer": "https://gushitong.baidu.com/",
}

def baidu_concept_blocks(code: str) -> dict:
    """
    百度股市通概念板块归属。
    返回: {industry: [...], concept: [...], region: [...], concept_tags: [...]}
    """
    url = (
        f"https://finance.pae.baidu.com/api/getrelatedblock"
        f"?code={code}&market=ab"
        f"&typeCode=all&finClientType=pc"
    )
    r = requests.get(url, headers=_BAIDU_PAE_HEADERS, timeout=10)
    d = r.json()
    if str(d.get("ResultCode", -1)) != "0":
        raise RuntimeError(f"百度PAE错误: {d}")

    result = {"industry": [], "concept": [], "region": [], "concept_tags": []}
    for block in d.get("Result", []):
        block_type = block.get("type", "")
        for item in block.get("list", []):
            entry = {
                "name": item.get("name", ""),
                "change_pct": item.get("increase", ""),
                "desc": item.get("desc", ""),
            }
            if "行业" in block_type:
                result["industry"].append(entry)
            elif "概念" in block_type:
                result["concept"].append(entry)
                result["concept_tags"].append(entry["name"])
            elif "地域" in block_type:
                result["region"].append(entry)
    return result

# 用法
blocks = baidu_concept_blocks("688017")
print("行业:", [b["name"] for b in blocks["industry"]])
print("概念:", blocks["concept_tags"])
print("地域:", [b["name"] for b in blocks["region"]])
```

> **踩坑：** `ResultCode` 返回类型不稳定——有时 int `0`，有时 string `"0"`。必须用 `str()` 统一比较。

### 3.4 东财 push2 — 个股资金流向（分钟级）

盘中实时分钟级资金流（主力/大单/中单/小单/超大单净流入）。

> **V3.1 替换说明：** 百度 PAE `fundflow` 和 `fundsortlist` 接口已于 2026-05 下线（返回 null），改用东财 push2 资金流 API。日级资金流见 Layer 4.5 `stock_fund_flow_120d()`。

```python
import requests

def eastmoney_fund_flow_minute(code: str) -> list[dict]:
    """
    个股资金流向（分钟级，当日盘中）。
    code: 6位股票代码
    返回: [{time, main_net, small_net, mid_net, large_net, super_net}, ...]
    单位: 元
    """
    secid = f"1.{code}" if code.startswith("6") else f"0.{code}"
    url = "https://push2.eastmoney.com/api/qt/stock/fflow/kline/get"
    params = {
        "secid": secid, "klt": 1,
        "fields1": "f1,f2,f3,f7",
        "fields2": "f51,f52,f53,f54,f55,f56,f57",
    }
    headers = {
        "User-Agent": UA,
        "Referer": "https://quote.eastmoney.com/",
        "Origin": "https://quote.eastmoney.com",
    }
    try:
        r = em_get(url, params=params, headers=headers, timeout=10)
        d = r.json()
    except Exception as e:
        print(f"[WARN] push2 资金流请求失败: {e}")
        return []

    rows = []
    for line in d.get("data", {}).get("klines", []):
        parts = line.split(",")
        if len(parts) >= 6:
            rows.append({
                "time": parts[0],
                "main_net": float(parts[1]),
                "small_net": float(parts[2]),
                "mid_net": float(parts[3]),
                "large_net": float(parts[4]),
                "super_net": float(parts[5]),
            })
    return rows

# 用法: 分钟级实时资金流
realtime = eastmoney_fund_flow_minute("000858")
if realtime:
    last = realtime[-1]
    signal = "bullish" if last["main_net"] > 0 else "bearish"
    print(f"主力净流入: {last['main_net']:.0f}元 → {signal}")
    # 统计全天主力净流入
    total = sum(r["main_net"] for r in realtime)
    print(f"全天主力累计: {total/1e4:.0f}万元")
```

> **注意：** push2 资金流金额单位是**元**（非万元），使用时注意换算。`klt=1` 分钟级，`klt=101` 日级。

### 3.5 龙虎榜席位 — 个股上榜记录 + 买卖席位 TOP5 + 机构动向

直连东财 datacenter API，不依赖第三方封装。

```python
import requests
from datetime import datetime, timedelta

def dragon_tiger_board(code: str, trade_date: str, look_back: int = 30) -> dict:
    """
    龙虎榜数据聚合。
    trade_date: YYYY-MM-DD
    look_back: 回看天数
    返回: {records: [...], seats: {buy: [...], sell: [...]}, institution: {...}}
    """
    start = datetime.strptime(trade_date, "%Y-%m-%d") - timedelta(days=look_back)
    start_str = start.strftime("%Y-%m-%d")

    # 1. 上榜记录
    records = []
    data = eastmoney_datacenter(
        "RPT_DAILYBILLBOARD_DETAILSNEW",
        filter_str=f"(TRADE_DATE>='{start_str}')(TRADE_DATE<='{trade_date}')(SECURITY_CODE=\"{code}\")",
        page_size=50,
        sort_columns="TRADE_DATE", sort_types="-1",
    )
    for row in data:
        records.append({
            "date": str(row.get("TRADE_DATE", ""))[:10],
            "reason": row.get("EXPLANATION", ""),
            "net_buy": round((row.get("BILLBOARD_NET_AMT") or 0) / 10000, 1),
            "turnover": round(float(row.get("TURNOVERRATE") or 0), 2),
        })

    # 2. 最近上榜的买卖席位
    seats = {"buy": [], "sell": []}
    if records:
        latest_date = records[0]["date"]
        # 买入席位
        buy_data = eastmoney_datacenter(
            "RPT_BILLBOARD_DAILYDETAILSBUY",
            filter_str=f"(TRADE_DATE='{latest_date}')(SECURITY_CODE=\"{code}\")",
            page_size=10,
            sort_columns="BUY", sort_types="-1",
        )
        for row in buy_data[:5]:
            seats["buy"].append({
                "name": row.get("OPERATEDEPT_NAME", ""),
                "buy_amt": round((row.get("BUY") or 0) / 10000, 1),
                "sell_amt": round((row.get("SELL") or 0) / 10000, 1),
                "net": round((row.get("NET") or 0) / 10000, 1),
            })
        # 卖出席位
        sell_data = eastmoney_datacenter(
            "RPT_BILLBOARD_DAILYDETAILSSELL",
            filter_str=f"(TRADE_DATE='{latest_date}')(SECURITY_CODE=\"{code}\")",
            page_size=10,
            sort_columns="SELL", sort_types="-1",
        )
        for row in sell_data[:5]:
            seats["sell"].append({
                "name": row.get("OPERATEDEPT_NAME", ""),
                "buy_amt": round((row.get("BUY") or 0) / 10000, 1),
                "sell_amt": round((row.get("SELL") or 0) / 10000, 1),
                "net": round((row.get("NET") or 0) / 10000, 1),
            })

    # 3. 机构买卖统计（从买卖席位明细中筛选 OPERATEDEPT_CODE="0" 即机构专用席位）
    institution = {"buy_amt": 0, "sell_amt": 0, "net_amt": 0}
    for detail_data, side in [(buy_data, "buy"), (sell_data, "sell")]:
        for row in detail_data:
            if str(row.get("OPERATEDEPT_CODE", "")) == "0":
                amt = (row.get("BUY") or 0) if side == "buy" else (row.get("SELL") or 0)
                if side == "buy":
                    institution["buy_amt"] += amt
                else:
                    institution["sell_amt"] += amt
    institution["buy_amt"] = round(institution["buy_amt"] / 10000, 1)
    institution["sell_amt"] = round(institution["sell_amt"] / 10000, 1)
    institution["net_amt"] = round(institution["buy_amt"] - institution["sell_amt"], 1)

    return {"records": records, "seats": seats, "institution": institution}

# 用法
data = dragon_tiger_board("002475", "2026-05-17")
print(f"近30日上榜 {len(data['records'])} 次")
for r in data["records"]:
    print(f"  {r['date']}: {r['reason']}")
if data["seats"]["buy"]:
    print("买入席位 TOP5:")
    for s in data["seats"]["buy"]:
        print(f"  {s['name']}: 买{s['buy_amt']}万 卖{s['sell_amt']}万 净{s['net']}万")
```

> **ST 股注意：** 5% 涨跌停更容易触发龙虎榜（"连续三日偏离值累计达12%"），科创板 20% 涨跌停则较少触发。

### 3.6 限售解禁日历 — 历史解禁 + 未来 90 天待解禁

```python
from datetime import datetime, timedelta

def lockup_expiry(code: str, trade_date: str, forward_days: int = 90) -> dict:
    """
    限售解禁日历。
    返回: {history: [...], upcoming: [...]}
    """
    # 1. 历史解禁记录
    history_data = eastmoney_datacenter(
        "RPT_LIFT_STAGE",
        filter_str=f"(SECURITY_CODE=\"{code}\")",
        page_size=15,
        sort_columns="FREE_DATE", sort_types="-1",
    )
    history = []
    for row in history_data:
        history.append({
            "date": str(row.get("FREE_DATE", ""))[:10],
            "type": row.get("LIMITED_STOCK_TYPE", ""),
            "shares": row.get("FREE_SHARES_NUM", 0),
            "ratio": row.get("FREE_RATIO", 0),
        })

    # 2. 未来待解禁
    end_date = datetime.strptime(trade_date, "%Y-%m-%d") + timedelta(days=forward_days)
    end_str = end_date.strftime("%Y-%m-%d")
    upcoming_data = eastmoney_datacenter(
        "RPT_LIFT_STAGE",
        filter_str=f"(SECURITY_CODE=\"{code}\")(FREE_DATE>='{trade_date}')(FREE_DATE<='{end_str}')",
        page_size=20,
        sort_columns="FREE_DATE", sort_types="1",
    )
    upcoming = []
    for row in upcoming_data:
        upcoming.append({
            "date": str(row.get("FREE_DATE", ""))[:10],
            "type": row.get("LIMITED_STOCK_TYPE", ""),
            "shares": row.get("FREE_SHARES_NUM", 0),
            "ratio": row.get("FREE_RATIO", 0),
        })

    return {"history": history, "upcoming": upcoming}

# 用法
data = lockup_expiry("002475", "2026-05-17")
print(f"历史解禁 {len(data['history'])} 批")
for h in data["history"][:5]:
    print(f"  {h['date']}: {h['type']} 数量={h['shares']}")
if data["upcoming"]:
    print(f"未来90天待解禁 {len(data['upcoming'])} 批")
else:
    print("未来90天无待解禁")
```

**限售股类型参考：**
- 首发原股东限售股份（IPO 后 1-3 年）
- 首发机构配售股份（IPO 战略配售）
- 定向增发机构配售股份（6-18 个月）
- 股权激励限售股份

### 3.7 行业板块排名（V3.0 改用东财 — 同花顺加了反爬401）

东财行业板块涨跌幅排名，一次调用看全市场行业轮动。

```python
import requests

def industry_comparison(top_n: int = 20) -> dict:
    """
    全行业涨跌幅排名（东财行业板块，~100 个行业）。
    返回: {top: [...], bottom: [...], total: int}
    """
    url = "https://push2.eastmoney.com/api/qt/clist/get"
    params = {
        "pn": "1", "pz": "100", "po": "1", "np": "1",
        "fltt": "2", "invt": "2",
        "fs": "m:90+t:2",
        "fields": "f2,f3,f4,f12,f13,f14,f104,f105,f128,f136,f140,f141,f207",
    }
    headers = {"User-Agent": UA}
    r = em_get(url, params=params, headers=headers, timeout=15)
    d = r.json()
    items = d.get("data", {}).get("diff", [])
    if not items:
        return {"top": [], "bottom": [], "total": 0}

    rows = []
    for i, item in enumerate(items):
        rows.append({
            "rank": i + 1,
            "name": item.get("f14", ""),
            "change_pct": item.get("f3", 0),
            "code": item.get("f12", ""),
            "up_count": item.get("f104", 0),
            "down_count": item.get("f105", 0),
            "leader": item.get("f140", ""),
            "leader_change": item.get("f136", 0),
        })

    return {
        "top": rows[:top_n],
        "bottom": rows[-top_n:],
        "total": len(rows),
    }

# 用法
data = industry_comparison(20)
print(f"共 {data['total']} 个行业")
print("\nTOP 10 涨幅:")
for r in data["top"][:10]:
    print(f"  {r['rank']}. {r['name']}: {r['change_pct']}% 涨{r['up_count']}跌{r['down_count']} 领涨{r['leader']}")
print("\nBOTTOM 5 跌幅:")
for r in data["bottom"][-5:]:
    print(f"  {r['rank']}. {r['name']}: {r['change_pct']}%")
```

### 3.8 全市场龙虎榜

每日全市场龙虎榜汇总——当日所有触发龙虎榜的股票 + 上榜原因 + 买卖净额 + 换手率。

```python
from datetime import datetime

def daily_dragon_tiger(trade_date: str = None, min_net_buy: float = None) -> dict:
    """
    全市场龙虎榜。
    trade_date: YYYY-MM-DD（默认当日）
    min_net_buy: 净买入下限（万元），None 不过滤
    返回: {date, total_records, stocks: [{code, name, reason, close, change_pct,
           net_buy_wan, buy_wan, sell_wan, turnover_pct}]}
    """
    if trade_date is None:
        trade_date = datetime.now().strftime("%Y-%m-%d")

    data = eastmoney_datacenter(
        "RPT_DAILYBILLBOARD_DETAILSNEW",
        filter_str=f"(TRADE_DATE>='{trade_date}')(TRADE_DATE<='{trade_date}')",
        page_size=500,
        sort_columns="BILLBOARD_NET_AMT", sort_types="-1",
    )
    if not data:
        return {"date": trade_date, "total_records": 0, "stocks": [],
                "note": "无数据（非交易日或盘后未更新）"}

    actual_date = str(data[0].get("TRADE_DATE", ""))[:10] if data else trade_date
    stocks = []
    for row in data:
        net_buy = (row.get("BILLBOARD_NET_AMT") or 0) / 10000
        if min_net_buy is not None and net_buy < min_net_buy:
            continue
        stocks.append({
            "code": row.get("SECURITY_CODE", ""),
            "name": row.get("SECURITY_NAME_ABBR", ""),
            "reason": row.get("EXPLANATION", ""),
            "close": row.get("CLOSE_PRICE") or 0,
            "change_pct": round(float(row.get("CHANGE_RATE") or 0), 2),
            "net_buy_wan": round(net_buy, 1),
            "buy_wan": round((row.get("BILLBOARD_BUY_AMT") or 0) / 10000, 1),
            "sell_wan": round((row.get("BILLBOARD_SELL_AMT") or 0) / 10000, 1),
            "turnover_pct": round(float(row.get("TURNOVERRATE") or 0), 2),
        })
    return {"date": actual_date, "total_records": len(stocks), "stocks": stocks}

# 用法
data = daily_dragon_tiger("2026-05-16")
print(f"{data['date']} 龙虎榜共 {data['total_records']} 条记录")
for s in data["stocks"][:10]:
    print(f"  {s['code']} {s['name']}: {s['reason']} | 净买{s['net_buy_wan']}万 涨跌{s['change_pct']}%")

# 只看净买入 > 5000 万的
data = daily_dragon_tiger("2026-05-16", min_net_buy=5000)
print(f"\n净买入 > 5000万: {data['total_records']} 条")
```

## Layer 4: 资金流向（个股资金验证）

### 4.5 个股资金流（120日，日级）

```python
import requests

def stock_fund_flow_120d(code: str) -> list[dict]:
    """
    个股资金流（日级，最近120个交易日）。
    返回: [{date, main_net(主力净流入), small_net, mid_net, large_net, super_net}]
    单位: 元
    """
    market_code = 1 if code.startswith("6") else 0
    url = "https://push2his.eastmoney.com/api/qt/stock/fflow/daykline/get"
    params = {
        "secid": f"{market_code}.{code}",
        "fields1": "f1,f2,f3,f7",
        "fields2": "f51,f52,f53,f54,f55,f56,f57,f58,f59,f60,f61,f62,f63,f64,f65",
        "lmt": "120",
    }
    headers = {
        "User-Agent": UA,
        "Referer": "https://quote.eastmoney.com/",
        "Origin": "https://quote.eastmoney.com",
    }
    try:
        r = em_get(url, params=params, headers=headers, timeout=15)
        d = r.json()
    except Exception as e:
        print(f"[WARN] push2 资金流请求失败: {e}")
        return []
    klines = d.get("data", {}).get("klines", [])

    rows = []
    for line in klines:
        parts = line.split(",")
        if len(parts) >= 7:
            rows.append({
                "date": parts[0],
                "main_net": float(parts[1]) if parts[1] != "-" else 0,
                "small_net": float(parts[2]) if parts[2] != "-" else 0,
                "mid_net": float(parts[3]) if parts[3] != "-" else 0,
                "large_net": float(parts[4]) if parts[4] != "-" else 0,
                "super_net": float(parts[5]) if parts[5] != "-" else 0,
            })
    return rows

# 用法
data = stock_fund_flow_120d("600519")
for d in data[-5:]:
    print(f"{d['date']}: 主力净流入={d['main_net']/1e4:.0f}万 超大单={d['super_net']/1e4:.0f}万")

# 统计近20日主力净流入
recent_20 = data[-20:]
total_main = sum(d["main_net"] for d in recent_20)
print(f"\n近20日主力累计净流入: {total_main/1e8:.2f}亿")
```

---

## Layer 5: 新闻层

### 5.1 东财个股新闻（直连 search-api-web）

```python
import requests
import re
import json

def eastmoney_stock_news(code: str, page_size: int = 20) -> list[dict]:
    """
    东财个股新闻（JSONP 接口）。
    返回: [{title, content, time, source, url}]
    """
    # 构造 JSONP 参数
    cb = "jQuery_news"
    url = "https://search-api-web.eastmoney.com/search/jsonp"
    inner_params = json.dumps({
        "uid": "",
        "keyword": code,
        "type": ["cmsArticleWebOld"],
        "client": "web",
        "clientType": "web",
        "clientVersion": "curr",
        "param": {"cmsArticleWebOld": {"searchScope": "default", "sort": "default",
                  "pageIndex": 1, "pageSize": page_size, "preTag": "", "postTag": ""}},
    }, separators=(',', ':'))
    params = {"cb": cb, "param": inner_params}
    headers = {"User-Agent": UA, "Referer": "https://so.eastmoney.com/"}
    r = em_get(url, params=params, headers=headers, timeout=15)

    # 解析 JSONP
    text = r.text
    json_str = text[text.index("(") + 1 : text.rindex(")")]
    d = json.loads(json_str)

    rows = []
    # 东财实际返回里 result.cmsArticleWebOld 直接就是文章列表（非 {list:[...]} 嵌套）
    articles = d.get("result", {}).get("cmsArticleWebOld", []) or []
    for a in articles:
        rows.append({
            "title": re.sub(r'<[^>]+>', '', a.get("title", "")),
            "content": re.sub(r'<[^>]+>', '', a.get("content", ""))[:200],
            "time": a.get("date", ""),
            "source": a.get("mediaName", ""),
            "url": a.get("url", ""),
        })
    return rows

# 用法
news = eastmoney_stock_news("688017")
for n in news[:5]:
    print(f"  {n['time']} | {n['source']} | {n['title']}")
```

---

## Layer 6: 基础数据层

### 6.2 mootdx F10（公司文本资料）

```python
from mootdx.quotes import Quotes

client = Quotes.factory(market='std')

# 9 大类文本数据:
categories = [
    "最新提示", "公司概况", "财务分析",
    "股东研究", "股本结构", "资本运作",
    "业内点评", "行业分析", "公司大事",
]
for cat in categories:
    text = client.F10(symbol='688017', name=cat)
    print(f"=== {cat} ===")
    print(text[:200] if text else "(空)")
```

> **优化提示：** "股东研究" 中的【4.股东变化】章节含大量历史十大股东列表，实测 16000+ chars。建议只保留最新一期（-70% token）。

### 6.3 东财个股基本面（直连 push2 API）

```python
import requests

def eastmoney_stock_info(code: str) -> dict:
    """
    东财个股基本面信息。
    返回: {code, name, industry, total_shares, float_shares, mcap, float_mcap, list_date}
    """
    market_code = 1 if code.startswith("6") else 0
    url = "https://push2.eastmoney.com/api/qt/stock/get"
    params = {
        "fltt": "2", "invt": "2",
        "fields": "f57,f58,f84,f85,f127,f116,f117,f189,f43",
        "secid": f"{market_code}.{code}",
    }
    headers = {"User-Agent": UA}
    r = em_get(url, params=params, headers=headers, timeout=10)
    d = r.json().get("data", {})
    return {
        "code": d.get("f57", ""),
        "name": d.get("f58", ""),
        "industry": d.get("f127", ""),
        "total_shares": d.get("f84", 0),     # 总股本(股)
        "float_shares": d.get("f85", 0),     # 流通股(股)
        "mcap": d.get("f116", 0),            # 总市值(元)
        "float_mcap": d.get("f117", 0),      # 流通市值(元)
        "list_date": str(d.get("f189", "")), # 上市日期 YYYYMMDD
        "price": d.get("f43", 0),
    }

# 用法
info = eastmoney_stock_info("688017")
print(f"{info['name']}({info['code']}): 行业={info['industry']} 总市值={info['mcap']/1e8:.0f}亿 上市={info['list_date']}")
```

## Layer 7: 公告层

### 7.1 巨潮公告（直连 cninfo.com.cn）

```python
import requests
from datetime import datetime

def _cninfo_ts_to_date(ts):
    """巨潮 announcementTime 返回 Unix 毫秒整数，需转换为日期字符串。"""
    if isinstance(ts, (int, float)):
        return datetime.fromtimestamp(ts / 1000).strftime("%Y-%m-%d")
    return str(ts)[:10] if ts else ""

def cninfo_announcements(code: str, page_size: int = 30) -> list[dict]:
    """
    巨潮公告全文检索。
    返回: [{title, type, date, url}]
    """
    url = "https://www.cninfo.com.cn/new/hisAnnouncement/query"
    # 构造 orgId（巨潮 2026 新格式）
    if code.startswith("6"):
        org_id = f"gssh0{code}"
    elif code.startswith("8") or code.startswith("4"):
        org_id = f"gsbj0{code}"
    else:
        org_id = f"gssz0{code}"

    payload = {
        "stock": f"{code},{org_id}",
        "tabName": "fulltext",
        "pageSize": str(page_size),
        "pageNum": "1",
        "column": "",
        "category": "",
        "plate": "",
        "seDate": "",
        "searchkey": "",
        "secid": "",
        "sortName": "",
        "sortType": "",
        "isHLtitle": "true",
    }
    headers = {
        "User-Agent": UA,
        "Content-Type": "application/x-www-form-urlencoded",
        "Referer": "https://www.cninfo.com.cn/new/disclosure",
        "Origin": "https://www.cninfo.com.cn",
    }
    r = requests.post(url, data=payload, headers=headers, timeout=15)
    d = r.json()

    rows = []
    for item in d.get("announcements", []) or []:
        rows.append({
            "title": item.get("announcementTitle", ""),
            "type": item.get("announcementTypeName", ""),
            "date": _cninfo_ts_to_date(item.get("announcementTime")),
            "url": f"https://www.cninfo.com.cn/new/disclosure/detail?annoId={item.get('announcementId', '')}",
        })
    return rows

# 用法
anns = cninfo_announcements("688017")
for a in anns[:10]:
    print(f"  {a['date']} | {a['type']} | {a['title']}")
```

### 7.2 mootdx F10 公告摘要

```python
from mootdx.quotes import Quotes
client = Quotes.factory(market='std')
text = client.F10(symbol='688017', name='最新提示')
# 包含最近的公告/分红/股东大会决议等摘要
```

---

## 战法决策框架（看盘九步 + 三档审核标准）

> 本工具包内置的 100 条短线战法，核心是三要素：**① 均线位置（MA5/MA24生命线/MA60季线）· ② 量能状态（均量线金叉死叉、持续放量）· ③ 所处高低位**。`stock_diagnosis()` 已把可量化规则编码为自动打分。
>
> 📖 **战法 100 条原文 + 每条对应的函数映射**见 `reference/战法100条.md`。判定逻辑有疑问时回查该文件。

### 看盘九步（规则33/95：先大盘后个股，先大级别后小级别）

```
① 大盘环境  → market_regime()        指数多空，决定能不能做       规则33/62/97
② 大级别位置→ weekly_view()           周线方向/三线合一，是不是高位 规则18/71/95
③ 均线形态  → compute_ma()            价 vs 5/24/60线、5日线vs生命线 规则18/26/32/51/88
④ 量能形态  → compute_vol_ma()        VMA5/VMA10金叉死叉、持续放量   规则04/30/37/63/84
⑤ K线形态  → detect_patterns()        涨停/光头大阳/红三兵/跳空/十字星 规则20/21/22/41/61/96
⑥ 涨停盘口  → limit_up_quality()       换手/量比/封单/封停时长        规则05/17/52
⑦ 板块联动  → ths_hot_reason()+概念板块 同题材强弱、当日热点          规则14/23/97
⑧ 资金验证  → 资金流(3.x)+龙虎榜       主力净流入、游资席位           规则64/76
⑨ 信号判定  → stock_diagnosis()        按规则打分 → 买/卖/回避         规则05/32/88/92
```

### 三档审核标准

**✅ 买入候选**（规则05/26/46/51，需多条同时成立）：
- 站上 5 日线 + 5 日线在生命线上方（多头排列）
- 温和/持续放量 + 量均线 VMA5 向上（规则46）
- 大盘非空头（规则33）
- 非高位、未大幅偏离均线（规则31）
- 加分项：今日涨停 / 向上跳空 / 红三兵 / 威廉<50

**🟡 持有/观察**：站上生命线、5日线多头，但无明确买点 → 多看少动（规则36）。

**⛔ 卖出/回避**（规则07/12/32/39/88/90/92，任一触发即警惕，≥2 条坚决回避）：
- **跌破生命线 MA24**（规则18：失去活力）/ 跌破季线 MA60（规则88：空仓休息）
- **5 日线死叉生命线**且未收复（规则32：极可靠卖出信号）
- 高位放量收阴、大幅偏离 5 日线（规则12/31）
- 均线空头向下发散（规则90）/ 均量线死叉（规则07/62）
- 弱势股不等反弹、不抄底（规则92）

> **生命线（MA24）是多空总分界**：股价在其上方才谈买点，下方一律回避——这是整套战法的总纲（规则18）。

---

## 完整战法流程

### 流程 A：单票战法体检（最常用）

```python
# 一行拿到完整诊断
d = stock_diagnosis("600879", "航天电子")
print(f"结论: {d['结论']}")
print(f"大盘: {d['大盘']}  位置: {d['位置']}")
print(f"均线: 站上5日线={d['均线']['above_MA5']} 站上生命线={d['均线']['above_MA24']}")
print(f"买入信号: {d['买入信号']}")
print(f"卖出回避: {d['卖出回避信号']}")
```

### 流程 B：当日强势股扫描 + 批量体检（选股）

```python
# 1. 拉当日强势股（同花顺热点，自带题材归因）
df_hot = ths_hot_reason()           # 见 Layer 3.1

# 2. 逐只体检，筛出"强势买入候选"
candidates = []
for _, row in df_hot.head(40).iterrows():
    code, name = str(row["代码"]), row["名称"]
    if "ST" in name:                # 规则92：弱势/风险股过滤
        continue
    d = stock_diagnosis(code, name, with_market=False)
    if d.get("结论") in ("强势买入候选", "买入候选"):
        candidates.append((code, name, row.get("题材归因",""), d["结论"], d["买入信号"]))

print(f"今日候选 {len(candidates)} 只:")
for code, name, reason, verdict, sig in candidates:
    print(f"  {code} {name} [{verdict}] {reason[:20]} | {sig}")
```

### 流程 C：题材联动确认（规则14/23）

```python
code = "605277"
# 1. 个股体检
d = stock_diagnosis(code, "新亚电子")
# 2. 概念板块归属（规则23：涨停联想同题材）
blocks = baidu_concept_blocks(code)        # 见 Layer 3.3
print("概念:", blocks["concept_tags"][:10])
# 3. 行业板块当日强弱（规则14：板块联动）
comp = industry_comparison(10)             # 见 Layer 3.7
print("强势行业 TOP5:", [(r["name"], r["change_pct"]) for r in comp["top"][:5]])
```

### 流程 D：持仓卖出预警（规则32/39/88）

```python
holdings = ["600879", "600519", "000858"]
for code in holdings:
    d = stock_diagnosis(code, with_market=False)
    if d["卖出回避信号"]:
        print(f"⛔ {code} {d['结论']}: {d['卖出回避信号']}")
    else:
        print(f"✅ {code} {d['结论']}")
```

---

## 数据源优先级

| 优先级 | 数据源 | 用途 | 可靠性 | 封IP风险 |
|--------|--------|------|--------|---------|
| 1 | **mootdx** (TCP) | K线+五档盘口+逐笔成交+周线+指数+F10（战法计算层数据源） | 极稳定 | 极低 |
| 2 | **腾讯财经** (HTTP) | 实时价/换手率/量比/振幅/涨跌停价/指数/ETF | 稳定 | 低 |
| 3 | **同花顺热点** (HTTP) | 当日强势股+题材归因 reason tags（选股） | 稳定 73ms | 极低（零鉴权） |
| 4 | **百度股市通** (HTTP) | 概念板块归属（题材联想）+K线带MA | 稳定 | 极低（零鉴权） |
| 5 | **东财 datacenter** (HTTP) | 龙虎榜/全市场龙虎榜/解禁/个股信息 | 稳定 | 中（走 em_get 限流） |
| 6 | **东财 push2/push2his** (HTTP) | 行业板块排名/个股资金流分钟级+120日 | 稳定 | 中（走 em_get 限流） |
| 7 | **东财 search-api** (HTTP) | 个股新闻 | 稳定 | 中（走 em_get 限流） |
| 8 | **巨潮 cninfo** (HTTP) | 公告全文检索+下载 | 稳定 | 低 |

**原则：** 战法计算（均线/量能/形态/指标/周线/大盘）全部走 mootdx+腾讯（不封IP，可高频）；题材选股走同花顺热点+百度概念；资金验证/龙虎榜/解禁/新闻走东财（统一经 `em_get()` 限流防封）。全部直连 HTTP/TCP，零第三方数据封装依赖。

---

## FAQ

### Q: mootdx 和腾讯有什么区别？
A: 互补。mootdx = K线/盘口/逐笔（战法计算层的数据源），腾讯 = 实时价/换手率/量比/涨跌停价。两者都不封 IP。

### Q: stock_diagnosis 判定准不准？
A: 它只把 100 条战法里**可量化**的均线/量能/形态规则编码自动打分，是"形态体检"，不含消息面/基本面/盘口博弈。务必结合龙虎榜、资金流、题材与大盘综合判断。**不构成投资建议。**

### Q: 为什么生命线用 MA24 而不是 MA20？
A: 战法原文（规则18）明确"二十四日均线为生命线"。MA20 也在 get_daily 里算了，可自行取用。

### Q: 涨停阈值怎么判断？
A: `limit_pct()` 按代码前缀+名称：ST=5%，科创/创业（688/300/301）=20%，主板=10%。

### Q: 大盘环境 market_regime 报错 / 指数取不到？
A: 指数必须走 `cli.index(symbol=...)`，不是 `cli.bars()`（bars 对指数会返回异常日期）。代码已处理。

### Q: 在海外服务器跑，mootdx 超时？
A: mootdx 走 TCP 直连通达信行情服务器，需国内 IP 才稳定。海外建议走代理。

### Q: 东财接口偶发连接中断？
A: push2/push2his 在部分网络下会被掐断，已统一走 `em_get()` 限流+会话复用；批量场景调大 `EM_MIN_INTERVAL`，必要时加重试。

### Q: 哪些数据源需要 API Key？
A: 全部免费无 key。mootdx / 腾讯 / 东财 / 同花顺 / 百度 / 巨潮均无需注册。

### Q: 不用 Claude Code，能用吗？
A: 能。SKILL.md 本质是 Markdown + 内嵌 Python。把代码段复制到自己的脚本里直接跑即可。

---

## 安装说明

```bash
# 1. 创建 skill 目录
mkdir -p ~/.claude/skills/a-stock-tactics

# 2. 将本文件复制为 SKILL.md
cp SKILL.md ~/.claude/skills/a-stock-tactics/SKILL.md

# 3. 安装 Python 依赖
pip install mootdx requests pandas stockstats

# 4. 启动 Claude Code，说"帮我体检一下 600879"即可自动激活
```
