# Cron Manifest

更新日期：2026-07-16

## 台股存股鏈（4 組）

| 時間 | Job ID | 腳本/來源 | 輸出 | 說明 |
|------|--------|----------|------|------|
| 08:30 | investor-scout | `~/.hermes/scripts/investor_daily_report.py` | `/tmp/investor_daily_report.md` | 抓即時價（TWSE）+ 技術面（yfinance）+ 基本面（MOPS）+ 新聞（Google News RSS）+ 三大法人（TWSE T86） |
| 08:35 | investor-analyst | `~/.hermes/scripts/investor_scout.py` → 讀 `/tmp/investor_daily_report.md` | `/tmp/investor_analyst.md` | 結構化分析，產出 verdict |
| 09:00 | investor-brief | briefer-cron-prompt.md + `/tmp/investor_analyst.md` + `/home/k8sadm/wiki/stock-analysis/verdict_history.jsonl` + `/home/k8sadm/wiki/stock-analysis/insights/insights.jsonl` | Telegram DM | 每日研究簡報，含系統學習重點 |
| 09:15 | investor-deep-research | 條件觸發：2 個以上事件旗標命中時寫入 flag | Telegram DM | 用 MixRoute deepseek-v4-flash |

## 豁免規則

- `approvals.cron_mode: approve` — 此 4 組 cron 自動 bypass 同意框
- `approvals.mode: smart` — 使用者互動操作仍會提示

## 來源架構

| 資料 | 來源 |
|------|------|
| 即時收盤價 | TWSE `mis.twse.com.tw/stock/api/getStockInfo.jsp` |
| MA/RSI/MACD/布林 | yfinance 歷史 K 線（僅計算指標） |
| 基本面/月營收/EPS | MOPS `mopsov.twse.com.tw` Playwright |
| 新聞 | Google News RSS |
| 三大法人 | TWSE `rwd/zh/fund/T86?selectType=ALL` |
| 配息歷史 | `/home/k8sadm/wiki/stock-analysis/dividend_history.json` |
| 學習庫 | `/home/k8sadm/wiki/stock-analysis/insights/insights.jsonl` |
