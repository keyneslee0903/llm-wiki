---
title: Single Agent vs Multi-Agent Setup
created: 2026-07-08
updated: 2026-07-08
type: comparison
tags: [model, architecture, comparison]
sources: [raw/articles/mixroute-hermes-multi-agent-setup-2026-06-20.md]
confidence: high
---

# Single Agent vs Multi-Agent Setup

| 維度 | 單一 Agent | 多 Agent（Scout / Analyst / Briefer） |
|------|-----------|-------------------------------------|
| Context 污染 | 高：歷次任務殘留於同一 context window | 低：各 profile context 互相隔離 |
| Skill 管理 | 單一身分強行承擔多種技能 | 每種角色只載入必要 skills |
| 判斷品質 | 搜尋與整理共用同一判斷回路 | 蒐集用寬鬆直覺、整理用嚴謹推理、回報用精準壓縮 |
| 錯誤定位 | 難：混合輸出難以追責 | 易： inspection 單一資料夾就能定位問題 |
| 成本 | 固定，即便無任務也要維持 context | wakeAgent gate 可讓空跑免費 |
| 複雜度 | 初期低，但任務一多就混亂 | 初期高（需建立 3 個 profile），長期可維護 |
| 複利效果 | 弱：知識與執行權重混在一起 | 強：共享 wiki 讓知識越級聯、越豐富 |
| 適用規模 | 輕度單一任務 | 持續性研究、多主題監控、每日彙整 |

## 結論

單一 agent 在**單一任務**場景依舊高效，但面對「搜尋 → 分析 → 回報」的管線式需求時，多 agent 分工在 **context 品質、錯誤定位、長期維護成本** 三方面顯著勝出。Wiku 模式則讓多個 agent 的輸出能互相疊加而非重複勞動。
