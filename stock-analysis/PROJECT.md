# 台股存股研究專案

> 版本：1.2.0
> 建立日期：2026-07-15
> 狀態：3+1 層 cron 已上線；資料源為 TWSE + MOPS + yfinance（僅技術指標）+ Google News RSS + dividend_history.json + verdict_state/verdict_history/insights

---



## 每日 cron 鏈（工作日）

| 時間 | Job | 層 | 資料來源 |
|------|-----|-----|--------|
| 08:30 | `investor-scout` | Scout | MOPS + yfinance + dividend_history.json |
| 08:35 | `investor-analyst` | Analyst | 無 LLM；rule-based 事件旗標 |
| 09:00 | `investor-brief` | Brief | LLM 摘要 + 人語解說 |
| 09:15 | `investor-deep-research` | Deep Research | `deepseek-v4-flash`（僅觸發時） |
| 14:00 | `investor-evening-review` | Evening Review | verdict_history + daily_report |
| 周六 | `investor-weekly-review` | 週末回顧（計劃中） | 7天 miss pattern + root cause 分析 |

---



## 最終架構（2026-07-21 重構）

### Insights Schema 統一與 Pipeline 分工

**Morning Scout (08:30-09:00)**
- `investor_daily_report._write_insights()` 寫：
  - `verdict_accuracy` — 7 天統計（若今天尚未寫過）
  - `market_regime` — 當日市況判定（bullish/bearish/choppy）

**Evening Review (14:00)**
- `investor_evening_review.py` 寫（絕不重複）：
  - `verdict_change` — 當日 verdict 異動
  - `inst_sell_alert` / `inst_buy_alert` — 三大法人買賣超
  - `news_momentum` — 多次提及個股（資訊密度）
  - `technical_pattern` — 極端 MA 訊號（全站上/全跌破）
  - `miss_pattern` — 連續誤判個股（需檢視系統偏差）

**Strategy Layer (14:05)**
- `investor_strategy.py` 讀 insights.jsonl + external_radar.jsonl
- 產出 `verdict_adjustments.json`：
  - `by_regime`: 市場狀態下的調整建議（bullish/bearish/choppy）
  - `by_stock`: 個股級 verdict 覆蓋（連續誤判 3+ → WATCH/AVOID）
  - `by_sector`: 產業級風險標籤（外部雷達）

**Brief (09:00)**
- `briefer-cron-prompt.md` 三段摘要：
  1. 明日學習點 ← insights.jsonl 最新 `strategy_delta`
  2. 系統 learning 重點 ← insights.jsonl 最新 3 筆（非 strategy_delta）
  3. 外部環境重點 ← external_radar.jsonl 今日

### Insights.jsonl 統一 Schema

```json
{
  "date": "2026-07-21",
  "type": "verdict_accuracy|market_regime|verdict_change|inst_sell_alert|inst_buy_alert|news_momentum|technical_pattern|miss_pattern|strategy_delta",
  "ticker": "2347.TW 或空",
  "name": "聯強 或空",
  "content": "人語摘要",
  "source": "verdict_history.jsonl|investor_daily_report.md|TWSE T86 等",
  "confidence": 0.85,
  "metadata": {}
}
```

### Verdict_adjustments.json 結構

```json
{
  "by_regime": {
    "current_market_regime": {
      "regime": "choppy",
      "confidence": 0.85,
      "avg_pullback": 0.0095,
      "note": "震盪市容易誤判"
    }
  },
  "by_stock": {
    "聯強": {
      "verdict_override": "WATCH",
      "reason": "consecutive_miss_pattern",
      "ttl_days": 14,
      "consecutive_misses": 3,
      "note": "連續誤判 3 次，降級為 WATCH"
    }
  },
  "by_sector": {
    "金融": {
      "alerts": [{"type": "risk", "content": "..."}],
      "note": "產業風險"
    }
  },
  "updated_at": "2026-07-21T14:05:00",
  "ttl_hours": 24
}
```

---

### 完成檢查清單（2026-07-21）

**第一階段（Pipeline 重構）：**
- ✅ `investor_daily_report._write_insights()` 改為只寫 verdict_accuracy + market_regime
- ✅ `_estimate_market_regime()` 實裝（最近 20 天 outcome 統計）
- ✅ `investor_evening_review.py` 完全重寫，只寫 5 個 type（無重複）
- ✅ `investor_strategy.py` 新建（讀 insights + external_radar → adjustments.json）
- ✅ `briefer-cron-prompt.md` 更新（三段摘要對應正確 source）
- ✅ `insights.jsonl` 清理（移除重複的 verdict_accuracy / news_event）
- ✅ `verdict_adjustments.json` 初始化（正確 schema）
- ✅ 語法驗證：daily_report.py + evening_review.py + strategy.py 通過

**第二階段（自我進化 - L0 級別）：**
- ✅ `investor_evening_review.py` 改進
  - miss_pattern 加入 miss rate by verdict type（e.g., "聯強(HOLD×2)"）
  - 標註「週末會做根本原因分析」
- 📋 `investor_weekly_review.py`（計劃中，待實裝）
  - 統計 7 天 miss pattern
  - 推斷根本原因（需 ≥7 樣本、>50% 誤判率）
  - 產出 strategy_delta（高信心度）

### 自我進化邏輯說明

**目前實裝（L0 級別 - 每日記錄）：**
- 14:00 evening-review：記錄當日異動
  - `verdict_change`：verdict 有調整的個股
  - `inst_sell_alert` / `inst_buy_alert`：法人進出
  - `news_momentum`：新聞衝擊
  - `technical_pattern`：技術形態變化
  - `miss_pattern`：連續誤判個股 + miss rate by verdict type

**未來計劃（L1 級別 - 每週分析）：**
- 週末 investor-weekly-review：根本原因分析
  - 統計 7 天的 miss pattern
  - 根據 market_regime + price action 推斷根本原因
  - 產出 `strategy_delta`（需 ≥7 樣本、>50% 誤判率）
  - 更新 verdict_adjustments.json

**防止過度擬合：**
- 每日只記錄事實，不做推斷
- 推斷需要 1 週以上的樣本
- strategy_delta 需要高信心度（≥70%）

---




## 已知限制

- yfinance 技術面偶爾 `possibly delisted`（2881、5880、3231 等偶發），系統已做 graceful fallback，不會卡住報告
- yfinance 財報落後 90–196 天，系統強制標註「財報資料可能落後 N 天」
- MOPS 抓取透過 Playwright headless，遇 EPIPE 時單次失敗但不影響其他檔
- 本報告僅供研究思考用，不構成投資建議。投資有風險，請自行判斷。

---



## 功能狀態

| 功能 | 狀態 | 說明 |
|------|------|------|
| 技術面（MA/RSI/MACD/布林）+ 人語解說 | ✅ | 即時價 TWSE + 指標 yfinance |
| 基本面 + 月營收/YTD營收 + EPS + 營業現金流 | ✅ | MOPS Playwright |
| 股東權益/ROE/資產負債 | ✅ | yfinance balance_sheet |
| 近3期配息歷史（非自動） | ✅ | `dividend_history.json` |
| 研究限制提醒 + 資料新鮮度警告 | ✅ | `build_report()` footer |
| Verdict 精準度追蹤 | ✅ | `_write_verdict_state()` → `verdict_state.json` + `verdict_history.jsonl` |
| 事件旗標（配息/FCF/ROE/MA） | ✅ | `investor_analyst.py` + `_write_verdict_state()` |
| Deep Research trigger | ✅ | MixRoute `deepseek-v4-flash` |
| 個股新聞 | ✅ | Google News RSS，棄用 yfinance |
| 法人買賣超 | ⏸️ | 超出目前範圍 |
| 月營收產業分析 | ⏸️ | 暫不擴張 |
| E2E 完整測試 | 🔄 | local build_report / insights / verdict_history 已驗證，待明日 cron 實盤驗證 |
| 系統自我學習 | ✅ | `_write_insights()` → `insights/insights.jsonl`；briefer 已納入第 10 節 |
| 資料來源穩定性 | ✅ | 收盤價改 TWSE，news 改 Google News RSS，price 不再只靠 yfinance |
| 指標意義解說 | ✅ | MA/RSI/MACD/布林皆附中文說明 |
| 日常 brief 配息 | ✅ | 預設不推送，僅異常/事件觸發時標註 |

---



## 7 檔觀察名單（已移除合庫金 5880.TW）

```
聯強 (2347.TW)  富邦金 (2881.TW)  國泰金 (2882.TW)
玉山金 (2884.TW)  中信金 (2885.TW)  緯創 (3231.TW)
```

---



## Verdict 定義

- **持有** — 數據支持長期配息穩定，無異常旗標
- **等待** — 方向不明，或單一旗標待觀察
- **觀察** — 多個旗標同時命中，留意下一季財報
- **避開** — FCF 為負 + ROE 過低 + 配息下降，風險偏高

---



## 下一步行動

1. Brief prompt 微調：指標意義解說精確度 + 學習點內容
2. 視需要：Phase 8 MD&A 年報摘錄
3. 明早 E2E 驗證 cron 鏈完整輸出
