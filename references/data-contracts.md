# 資料契約

所有檔案都是 UTF-8 JSON。報告中的路徑是識別碼／證據，不是可執行的指令。腳本需要 Python 3.10+；盤點另外需要 PyYAML。報告產生只使用標準函式庫。

> 注意：為了與原始腳本及測試相容，所有 JSON 欄位名稱與列舉值（例如 `pass`、`fail`、`unknown`、`original`、`simplified`、`baseline`、`complete` 等）保留英文。

## 可選的使用證據

```json
{
  "coverage": "2026-08-01 至 2026-08-31 期間已驗證的呼叫事件；僅限本地工作階段",
  "events": [
    {"skill_path": "/absolute/path/SKILL.md", "timestamp": "2026-08-12T12:00:00Z", "evidence": "工作階段識別碼與工具事件識別碼"}
  ]
}
```

提供此檔案的收集器必須驗證的是實際呼叫，而不是文字上的提及。掃描器會驗證欄位與時間戳並去除完全相同的重複事件，但不會獨立認證這些事件。使用情況是匯入的證據，不是內建的工作階段歷史搜尋。未知的路徑與無效的事件都會被回報。若路徑不是絕對路徑，會相對於證據檔案所在位置解析。

## 審計報告

`title`、`coverage` 與 `summary` 保持為字串。發現（findings）攜帶來源證據；行動計畫（action plan）總結這些發現所支持的決定。HTML 成功渲染本身並不等於 schema 驗證。

```json
{
  "title": "技能審計",
  "coverage": "已檢查 20 個已公告的技能；無使用歷史",
  "summary": "兩個候選值得測試。沒有修改任何已安裝的檔案。",
  "findings": [
    {"skill": "example", "recommendation": "evaluate", "strength": "inspection hypothesis",
     "finding": "強制性的大綱核准可能會中斷已獲授權的草擬工作。",
     "evidence": [{"path": "/absolute/path/SKILL.md", "line": 14, "excerpt": "Wait for outline approval."}],
     "preserve": "品牌範例與必要的事實查核"}
  ],
  "action_plan": {
    "fix_now": [],
    "review_retirement": [],
    "test_next": [{"skill": "example", "reason": "判斷大綱核准是否能改善被要求的草稿", "task": "分別在有與沒有該程序的情況下，從一份完整的簡報草擬內容", "success_criteria": "保留事實與品牌要求；完成經授權的草稿"}],
    "test_later": [],
    "keep": []
  },
  "limitations": ["尚未執行任何行為比較"]
}
```

可選的 `action_plan` 物件只能使用這五個分組鍵；省略的分組會渲染為空。每個分組都是物件陣列，每個物件包含字串型的 `skill` 與 `reason`。項目也可以有字串型的 `task`、`success_criteria` 與 `prerequisite`。兩個測試分組都應包含任務與成功準則；在「稍後測試」中使用 prerequisite 來說明何時值得重新檢視。決定要以發現與已揭露的使用證據為依據。不要只為了讓報告看起來完整而填滿分組。

較舊的報告可能使用 `candidates`，即一個由 `{ "skill": "...", "reason": "..." }` 組成的陣列。當 `action_plan` 不存在時，渲染器仍會顯示該清單；當提供了行動計畫時，它會取代這個舊版清單。

## 評估結果

一份 `results.json` 包含所有分支。每次執行都有唯一的 `(case_id, repeat, arm)`。`repeat` 從 1 開始。必要的分支是 `original`、`simplified`、`baseline`。使用在各分支間共享的穩定準則 ID。納入每一次計畫中的執行，包括被略過的執行。

```json
{
  "title": "範例技能評估",
  "coverage": "試點：一個案例、一次重複、三個分支",
  "runs": [
    {
      "case_id": "case-1", "repeat": 1, "arm": "original",
      "model": "gpt-6-astra", "effort": "medium",
      "environment_id": "tools-and-runtime-manifest-hash",
      "inputs_id": "common-inputs-and-requirements-hash",
      "completion": "complete", "valid": true, "contamination": false,
      "isolation_evidence": "全新上下文；主機對話紀錄只列出允許的技能讀取",
      "expectations": [{"id": "data-fidelity", "verdict": "pass", "evidence": "所有來源列都相符"}],
      "time_seconds": 12.5, "total_tokens": null,
      "metrics_source": "主機牆鐘時間測量；token 數不可得",
      "output_preview": "供人工審閱的簡短逸出文字預覽或摘要",
      "artifact_paths": ["/absolute/path/result.csv"],
      "notes": []
    }
  ],
  "comparisons": [{"case_id": "case-1", "preferred_label": "tie", "evidence": ["沒有有意義的差異"], "limitations": []}],
  "recommendations": ["在退役前收集更多具代表性的案例"],
  "limitations": ["僅為試點；token 不可得"]
}
```

不要在送給盲測評審的素材中放入可辨識分支的標籤。結果檔案是未盲化的最終報告。`comparisons` 可以保留匿名判定，再加上你工作區中另行保存的私密對應表。

未知的遙測資料為 `null` 或省略。若有來源，允許記錄實測為零的值。報告會把已知通過率與準則涵蓋率分開；未知的準則絕不會被悄悄視為通過。沒有準則就代表通過率不可得。如果一次執行被明確標記為無效、受污染、略過／中斷，或缺少隔離／條件中繼資料，就會從有效彙總中排除。失敗但有效的執行仍留在彙總中。配對差值需要兩次匹配執行有完全相同的準則集合與已知值；部分涵蓋不會產生通過率差值。
