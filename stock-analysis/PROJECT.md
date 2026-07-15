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

---



## 最終架構（2026-07-15）

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Scout         │────▶│  Analyst        │────▶│   Brief         │
│  08:30 多源抓取  │     │  08:35 rule-based│     │  09:00 LLM 摘要  │
│                 │     │  事件旗標判定     │     │  + 指標人語解說  │
└─────────────────┘     └────────┬────────┘     └─────────────────┘
                                 │
                   觸發時        │       觸發時
              ┌──────────────────┴──────────────────┐
              │   /tmp/investor_deep_research.flag   │
              └──────────────────┬──────────────────┘
                                 │ 09:15
                                 ▼
                      ┌─────────────────────┐
                      │ Deep Research        │
                      │ MixRoute deepseek-   │
                      │ v4-flash             │
                      └─────────────────────┘

資料來源分層：
  技術面 即時收盤價             → TWSE `mis.twse.com.tw/stock/api/getStockInfo.jsp`（穩定無 key）
  技術面 指標（MA/RSI/MACD/布林）→ yfinance 歷史 K 線（price 已棄用）
  基本面（營收/EPS/營業現金流）   → MOPS `mopsov.twse.com.tw` Playwright 抓取
  每股配息                      → MOPS > `dividend_history.json`
  個股新聞                      → Google News RSS，棄用 yfinance news
  verdict 歷史 + outcome         → `verdict_state.json` + `verdict_history.jsonl`
```

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



## 7 檔觀察名單

```
聯強 (2347.TW)  富邦金 (2881.TW)  國泰金 (2882.TW)
玉山金 (2884.TW)  中信金 (2885.TW)  緯創 (3231.TW)  合庫金 (5880.TW)
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
