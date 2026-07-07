---
title: Briefer Profile
created: 2026-07-08
updated: 2026-07-08
type: entity
tags: [model, architecture]
sources: [raw/articles/mixroute-hermes-multi-agent-setup-2026-06-20.md]
confidence: high
---

# Briefer Profile

Briefer 是 [hermes-multi-agent-setup](./hermes-multi-agent-setup.md) 中的**晨報發送者**，只讀取 wiki，輸出五條重點到 Telegram。

## 職責
- 讀取過去 24 小時的 wiki 更新
- 與使用者本週專案交叉比對
- 發送五點式 prioritized brief 到 Telegram
- 附上本週累計 token 花費

## 輸出位置
Telegram chat

## 模型選擇
便宜、快速的模型即可。任務單一、輸出極短，不需要強推理。

## 排程
建議在每天 8:00 執行，比使用者喝咖啡早。

## 限制
- 不超過 5 條
- 不重複昨天內容，除非狀態有改變

## 相關頁面
- [hermes-multi-agent-setup](./hermes-multi-agent-setup.md)
- [analyst-profile](./analyst-profile.md)
