# 致謝與變更記錄

## 本中文翻譯版本

本倉庫是 [cth9191/skill-audit](https://github.com/cth9191/skill-audit) 的中文翻譯。翻譯依原專案的 Apache License 2.0 授權發布，原始版權歸原作者所有。翻譯範圍包括所有 Markdown 文件、腳本中的註解、docstring、CLI 說明與 HTML 報告標籤，以及範例資料中的說明文字；JSON 欄位名稱、狀態值與錯誤訊息中的契約字串保留英文，以維持與原始資料契約及測試相容。

## 上游來源

`references/grader.md`、`references/comparator.md`、`references/analyzer.md` 中的評估角色指令，以及 `references/evaluation.md` 中的工作流程，改編自 Anthropic 的 skill-creator。

- 上游：https://github.com/anthropics/skills/tree/41bbe19d1a1a7eaab5e7bb9050a417e5c6cffc8f/skills/skill-creator
- 上游版權：Copyright 2026 Anthropic, PBC.
- 授權：Apache License 2.0，全文收錄於 LICENSE.txt。
- 取得日期：2026-09-08。

為 Codex 所做的修改：以原生的乾淨上下文執行取代 Claude CLI 呼叫；三組比較分支；明確的污染檢查；凍結的結果準則；允許平手與「無法判定」的結果；把客觀要求與主觀偏好分開；未知的遙測資料保留為缺失；把檢查與測量區分開；不自動修改目標技能。

Python 盤點、彙總與 HTML 報告的實作是全新撰寫的。它遵循一般的評估方法，但沒有複製 Anthropic 的 Python 或 HTML 實作。

參考指引：

- https://developers.openai.com/api/docs/guides/latest-model#prompting-best-practices
- https://claude.com/blog/improving-skill-creator-test-measure-and-refine-agent-skills

這些參考資料支持審計與測試工作。它們並不能證明任何特定技能是多餘的，也不能證明 Astra 在沒有技能時總是表現更好。
