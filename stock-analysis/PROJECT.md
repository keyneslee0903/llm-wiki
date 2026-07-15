# 台股存股研究專案

> 版本：1.0.0  
> 建立日期：2026-07-15  
> 狀態：Phase 1 + Phase 2 已完成；Tavily 已驗證可作為 Scout fallback

**重要提醒**：all market/financial data 目前由 yfinance 提供，已知限制包括資料可能不準確、過期、缺漏；凡是投資判斷應額外查證。系統會標註資料來源，但不擔保正確性。

---

## 3. Phase 1 完成說明（2026-07-15）

| 工作包 | 狀態 | 產出 |
|--------|------|------|
| Scout/Analyst/Brief 3 層骨架 | ✅ | `investor_scout.sh` / `investor_analyst.py` / briefer profile |
| Outcome tracking | ✅ | `investor_verdict_tracker.py`，每天寫入 `verdict_history.jsonl` |
| 配息歷史 DB | ✅ | `dividend_history.json`，7 檔 2021–2026 |
| Deep Research trigger | ✅ | `/tmp/investor_deep_research.flag` + cron `investor-deep-research` |
| E2E 驗證 | ✅ | scout → analyst → verdict tracker，exit 0 |
| 專案文件 | ✅ | `PROJECT.md`（本文件）|

---

## 4. Phase 2 完成說明（2026-07-15）

| 工作包 | 狀態 | 產出 |
|--------|------|------|
| 配息異常旗標 | ✅ | `investor_analyst.py` flag_signals 已加入配息檢查 |
| Brief 整合配息與 verdict 歷史 | 🟡 | prompt 已更新，待下次 briefer cron 實際輸出確認 |
| 本地新聞 / Tavily | ⏸️ | 免費 yfinance 新聞不完整，但穩定；本地 source 維護成本高，暫擱 |
| 月營收 / 法人預測 | ⏸️ | 同上 |
| MD&A / 年報摘錄 | ⏸️ | 排 Phase 4 |

---

## 5. 每日 cron 鏈（工作日）

| 時間 | Job | 層 | 模型 |
|------|-----|-----|------|
| 08:30 | `investor-scout` | Scout | 無 LLM |
| 08:35 | `investor-analyst` | Analyst | 無 LLM |
| 09:00 | `investor-brief` | Brief | 免費/預設 |
| 09:15 | `investor-deep-research` | Deep Research | `deepseek-v4-flash`（僅觸發時）|

---

## 6. 成果與學習點

---

## 1. 專案願景

建立一個**低成本、可累積、教投資的台股研究系統**，服務對象是一位長期存股族，關注 7 檔支持倉。

---

## 2. 核心目標

### P0（必須完成）
1. **每日 09:00 自動 Briefing** — 7 檔 verdict + 重點摘要
2. **Verdict 準確性追蹤** — 系統回頭看自己昨天說對了沒
3. **配息穩定性監控** — 連續配息中斷或大幅下降時報警
4. **歷史資料累積** — 每天歸檔，可回看趨勢

### P1（期望完成）
5. **事件驅動 Deep Research** — 觸發條件下用高階 LLM 做深度分析
6. **本地新聞補充** — yfinance 缺失時補抓台股公告
7. **教學機制** — Brief 內含「今日學習點」和指標解釋

### P2（未來願景）
8. **產業月營 revenuedriven 分析** — 當月營年 Revenue 月營 yfinance 太貴或太難維持，先擱置 yfinance 月營收
9. **TEJ/券商 API 整合** — 法人預測、EPS 共識、產業數據
10. **多用戶支援** — 延伸為通用台股研究平台

---

## 3. 範圍界定

### 在範圍內（In-Scope）
- 7 檔支持倉的技術面 + 基本面 + 配息分析
- 大盤 5 日表現摘要
- 事件旗標（ROE < 2%、FCF 為負、MA 全跌破、配息中斷...）
- Deep Research 觸發條件判定
- 歷史 verdict 準確性追蹤
- 台股本地新聞補充（鉅亨、經濟日報）
- 教學導向的 Brief

### 超出範圍（Out-of-Scope）
- 投資组合風險管理（beta、相關性、行业曝險）— 用戶偏好直接持有個股或 0050
- 短線交易建議、價差目標、進出場點
- 期貨/選擇權/權證
- 美股、港股、ETF（0050 除外）
- 即時 taping、盤中監控
- 法人買賣超等級數據（TEJ）

---

## 4. 系統架構

```
Phase 1: 骨架（已完成）
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Scout     │────▶│  Analyst    │────▶│   Brief     │
│ 08:30 抓資料 │     │ 08:35 結構化  │     │ 09:00 報告   │
└─────────────┘     └─────────────┘     └─────────────┘

Phase 2: 學習與回饋
┌─────────────────────────────────────────────────┐
│  Verdict Tracker  ──▶  Outcome Loop             │
│  （記錄每天 verdict + 實際價格）                │
│  → 寫入 verdict_history.jsonl                  │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│   Dividend DB  ──▶  Dividend Event Triggers      │
│   （配息歷史 + 異常檢測）                        │
└─────────────────────────────────────────────────┘

Phase 3: 事件與深度
┌──────────────────┐    ┌──────────────────────────┐
│  Event Trigger   │───▶│  Deep Research（按需）    │
│  （rule-based    │    │  用高階 LLM 做語意分析    │
│   條件判定）      │    └──────────────────────────┘
└──────────────────┘
```

---

## 5. 期程規劃

| 階段 | Milestone | 預計完成 | 負責 |
|------|-----------|---------|------|
| **Phase 0** | 3 層骨架 + 歷史歸檔 | ✅ 2026-07-14 | Hermes |
| **Phase 1** | Verdict Outcome Loop | 2026-07-15 | Hermes |
| **Phase 2** | 配息庫 + 事件旗標 + Brief 更新 | 2026-07-15 | Hermes |
| **Phase 3a** | 本地新聞補充 | 2026-07-16 | Hermes |
| **Phase 3b** | 完整 E2E 測試 | 2026-07-16 | Hermes |
| **Phase 4** | Deep Research Trigger | 2026-07-17 | Hermes |
| **Phase 5** | Brief Prompt 最終 polish + 教學內容 | 2026-07-18 | Hermes |
| **Phase 6** | 月營收分析 | 暫擱 | Hermes + TEJ |
| **Phase 7** | 法人預測 / EPS 共識 | 暫擱 | TEJ/券商 API |
| **Phase 8** | 公開資訊觀測站 MD&A 摘錄 | 2026-07-20 | Hermes |

---

## 6. 額外投資需求

### 已投資（Sunk Cost）
| 項目 | 金額 | 說明 |
|------|------|------|
| Hermes Agent | 免費 | 本平台 |
| yfinance | 免費 | 資料來源 |
| Raspberry Pi | 既有硬體 | 伺服器 |

### 未來可能投資
| 項目 | 預估金額 | 必要性 | 時程 |
|------|---------|--------|------|
| TEJ 台灣經濟新報 | NT$3,000–6,000/年 | 中 | Phase 7 |
| 鉅亨/富果付費 API | NT$1,000–3,000/月 | 低 | 視新聞品質決定 |
| 雲端運算（升級 Pi） | NT$500/月 | 低 | 目前夠用 |
| **Token 成本** | **NT$10–20/月** | 中 | Phase 4 起 |

### Token 成本說明
- 每日 Brief：免費 LLM，**NT$0**
- Deep Research（每月觸發 3–5 次）：**NT$1.5–7/月**
- 預估總額：**NT$10–20/月**（含緩衝）

---

## 7. 風險登記簿

| 風險 | 影響 | 緩解措施 | 狀態 |
|------|------|---------|------|
| yfinance 新聞不完整 | Brief 品質下降 | 補抓鉅亨網公告 | 規劃中 |
| 免費 LLM 品質不穩定 | Brief 邏輯錯誤 | 用 prompt engineering 補強 | 監控中 |
| 月營收 URL 變動 | 自動化失效 | 寫 fallback 邏輯，失敗時 skip | 已發生 |
| TEJ 成本偏高 | ROI 不足 | 先試免費方案，不夠再上 | 保留選項 |
| 系統過於複雜 | 維護困難 | 每階段 review，砍不必要的功能 | 持續 |

---

## 8. 完成標準（Definition of Done）

每個 Phase 完成時，必須滿足：

- [ ] 腳本自動化，手動可重複執行
- [ ] 產出檔案存在 wiki/stock-analysis/
- [ ] 更新本文件的 Progress Tracking
- [ ] 在 Brief 內可驗證功能正常
- [ ] 至少跑一次 end-to-end 測試

---

## 9. Progress Tracking

```
✅ Phase 0: 3 層骨架 + 歷史歸檔 — 2026-07-14 完成
   - Scout/Analyst/Brief cron jobs 上線
   - /home/k8sadm/wiki/stock-analysis/YYYY-MM-DD.md 歸檔
   - Outcome tracking 腳本完成（待整合）
   - 配息歷史 DB 完成

🔄 Phase 1: Verdict Outcome Loop — 今日完成
   - [x] 寫 verdict_tracker.py
   - [x] 寫 verdict_history.jsonl
   - [ ] 更新 Brief prompt 讀取 verdict_history
   - [ ] E2E 驗證 Brief 顯示前日 verdict 驗證

🔄 Phase 2: 配息旗標 — 今日完成
   - [x] dividend_history.py 抓取 7 檔歷史配息
   - [x] analyst.py 加配息異常旗標
   - [x] scout.py 加入近 3 期配息到 raw data
   - [ ] Brief prompt 提示 LLM 使用配息歷史
   - [ ] E2E 驗證配息異常出現在 Brief

⏳ Phase 3a: 本地新聞補充 — 明日
⏳ Phase 3b: 完整 E2E 測試 — 明日
⏳ Phase 4: Deep Research Trigger — 2026-07-17
⏳ Phase 5: Brief Polish + 教學 — 2026-07-18
⏸️ Phase 6: 月營收 — 暫擱
⏸️ Phase 7: 法人預測 — 暫擱
⏸️ Phase 8: MD&A — 2026-07-20
```

---

## 10. 下一步行動

1. 完成 Phase 1：整合 verdict history 到 Brief
2. 完成 Phase 2：Brief prompt 使用配息歷史做語意分析
3. E2E 完整測試一次
4. 準備 Phase 3：新聞 fallback

---

## 附錄：配息與財務指標字典

（教學用，逐步擴充）

### ROE（Return on Equity）
> 股東權益報酬率。公司用股東的錢每賺多少。10%↑ 優秀，5%↓ 注意。

### FCF（Free Cash Flow）
> 自由現金流 = 營運現金流 - 資本支出。負值 = 公司在花老本，不是賺錢。

### 營收 vs 淨利
> 營收是 top-line，淨利是 bottom-line。營收成長但淨利不增 = 成本侵蝕。

### MA（移動平均線）
> 過去 N 天的平均價格。MA5/20/60 全跌破 = 短中長期都輸給大盤。

### Verdict
> 系統當日結論：持有 / 等待 / 觀察 / 避開。
