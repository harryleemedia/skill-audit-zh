# Skill Audit（技能審計）

> 本倉庫是 [cth9191/skill-audit](https://github.com/cth9191/skill-audit) 的中文翻譯版本，依 Apache License 2.0 授權發布。原始英文版本與所有版權歸原作者所有；程式碼中的 JSON 欄位名稱與狀態值保留英文，以維持與原始資料契約相容。

找出哪些 AI Agent 技能需要修復、哪些彼此重疊、哪些值得先測試再決定保留、簡化或退役。

Skill Audit 結合了三個部分：一套可供 Agent 讀取的工作流程、一個唯讀的 Python 盤點掃描器，以及可獨立開啟的 HTML 報告。它是為 Codex 設計的；檔案系統掃描器也可以檢查 Claude Code 以及其他使用 `SKILL.md` 檔案的安裝環境。

## 它能做什麼

| 階段 | 工作內容 | 產出 |
| --- | --- | --- |
| 審計（Audit） | 盤點技能檔案；檢查標頭、本地連結、完全重複、描述重疊，以及指令訊號 | 附路徑與行號的證據，加上一份簡短的排序審查清單 |
| 使用情況審查（經授權時） | 匯入已驗證的呼叫證據，並說明其涵蓋範圍 | 觀察到的活動，並保留未知項與限制 |
| 評估（Evaluate） | 在隔離試驗中比較原始指令、簡化版本與無技能基準 | 結果準則、產出物、實測遙測資料，以及在證據不足時的「無法判定」結果 |
| 套用（Apply，經要求時） | 備份原件、執行經授權的修復或退役、更新呼叫方，並重新檢查技能庫 | 可審閱的變更與還原記錄 |

腳本負責收集訊號並呈現證據。Agent 負責情境式審查、設計比較實驗，並套用經授權的變更。這裡沒有自動刪除按鈕、沒有通用品質分數，也不會宣稱「新模型讓技能變得不再必要」。

## 安裝

把本倉庫 clone 到你的 Agent 會載入的技能目錄下的一個空 `skill-audit` 資料夾。對於預設的 Codex 安裝：

```bash
git clone https://github.com/cth9191/skill-audit.git ~/.codex/skills/skill-audit
python -m pip install -r ~/.codex/skills/skill-audit/requirements.txt
```

PowerShell：

```powershell
git clone https://github.com/cth9191/skill-audit.git "$env:USERPROFILE/.codex/skills/skill-audit"
python -m pip install -r "$env:USERPROFILE/.codex/skills/skill-audit/requirements.txt"
```

如果你使用自訂的 `CODEX_HOME`，請安裝到它的 `skills` 目錄下。如果目標位置已存在，請保留它並有意識地更新，而不是直接 clone 覆蓋。需要 Python 3.10+；盤點功能需要 PyYAML，報告產生則只使用標準函式庫。私有倉庫需要 GitHub 存取權限才能 clone。

## 第一次使用？從這裡開始

安裝技能後，把下面這段貼給你的 Agent：

```text
Use $skill-audit for a first-pass audit of my installed skill library.
Scan the available library and inspect the flagged issues in context.
Generate an HTML report that separates verified fixes, retirement decisions
for me to review, skills worth testing next, and lower-priority tests for later.
For each proposed test, explain the question it would answer and suggest a
representative task. Include strengths worth keeping and disclose scan gaps.
Do not change skills, inspect conversation history, or run benchmarks yet.
```

中文提示詞版本（意思相同，可擇一使用）：

```text
使用 $skill-audit 對我已安裝的技能庫做第一輪審計。
掃描可用的技能庫，並在情境中檢查被標記的問題。
產生一份 HTML 報告，分別列出：已驗證的修復項、需要我審查的退役決定、
下一步值得測試的技能，以及優先度較低、可稍後再測的項目。
針對每個建議的測試，說明它要回答什麼問題，並建議一個具代表性的任務。
包含值得保留的優點，並揭露掃描的缺口。
先不要修改技能、不要檢查對話紀錄，也不要執行基準測試。
```

這個基本執行會產生一份盤點清單和一份有證據支撐的 HTML 報告。它會掃描發現的技能庫並審查相關標記；它不會執行每一個技能，也不會證明每個未被標記的技能都是健康的。如果你也想納入 Claude Code 安裝環境或其他技能目錄，請告訴 Agent。報告應該說明它能檢查什麼、不能檢查什麼。

| 報告分組 | 屬於這一組的內容 | 你的下一個決定 |
| --- | --- | --- |
| 立即修復（Fix now） | 已驗證的缺陷，例如缺少必要的輔助檔案或格式錯誤的中繼資料 | 選擇你想套用的修復 |
| 審查是否退役（Review for retirement） | 有具體相關性或重複疑慮的技能 | 確認你是否還需要這個工作流程 |
| 下一步測試（Test next） | 具代表性的任務可以解決的重大不確定性 | 選擇一小批，通常最多三個技能 |
| 稍後測試（Test later） | 優先度較低的問題，或等待測試夾具／依賴修復後才能測的項目 | 當所述條件改變時再回頭看 |
| 保留／保存（Keep / preserve） | 審查過程中發現的有用知識、腳本、範例與限制條件 | 在簡化或整併時保留它們 |

分組為空是正常的。「下一步測試」表示測試會有助於做決定，不代表該技能已被證明有缺陷。缺少使用證據、技能的存在時間、檔案大小，這些單獨都不足以判定一個技能應該退役。

先從報告開始，然後有選擇地測試。對 100 個技能、各以兩個任務、三種條件做測試，在還沒重複之前就需要 600 次執行；基本審計不應該悄悄啟動這種規模的工作。一個屬於已放棄專案的技能可能只需要你做相關性判斷，而一個常用但價值不明的技能可能值得做一次比較。

合成報告結構請見 [examples/audit.json](examples/audit.json)。Python 腳本負責產生盤點清單與 HTML；Agent 負責情境式審查並選擇建議。

## 依報告追蹤後續

從你的報告中選出名稱，替換下方方括號內的內容。這些是各自獨立的後續步驟，不是必須全部依序執行的流程。

```text
Use $skill-audit to test only [skill names] from the Test next group.
Use two representative tasks per skill and compare the current procedure,
a simpler version, and a baseline. Start with a small isolated pilot.
Report outcomes and limitations; do not change the installed skills yet.
```

```text
Apply only the verified repairs for [skill names] from the report.
Back up originals, preserve the listed constraints, and verify the changes.
```

```text
Retire only [skill names] from the report; I no longer need those workflows.
Back up the installed copies, update affected callers, and verify the result.
```

可選的使用情況審查：

```text
Review skill usage over the last 60 days using the local logs I authorize you
to inspect. Separate actual launches from file reads and ordinary mentions.
Explain any retention gaps before recommending retirement.
```

使用日誌的收集是由 Agent 協助完成的工作，**不是內建的 Codex/Claude 對話紀錄解析器**。內附的匯入器接受符合[文件規範格式](references/data-contracts.md)的已驗證呼叫事件。檔案讀取可能只是維護行為；目錄中的提及不算使用；缺少日誌不能證明某個技能從未被使用。

## 直接執行腳本

請在倉庫根目錄執行以下指令。輸出目錄要選在所有被掃描技能目錄之外。

```bash
python scripts/inventory.py --root /path/to/skills --out /path/to/local-audit/inventory.json
python scripts/inventory.py --root /path/to/skills --usage /path/to/usage.json --out /path/to/local-audit/with-usage.json
python scripts/report.py audit examples/audit.json --out /path/to/local-audit/audit.html
python scripts/report.py benchmark examples/results.json --out /path/to/local-audit/benchmark.html
```

重複使用 `--root` 可以掃描多個位置。若不指定任何 root，掃描器會使用 `$CODEX_HOME/skills`（或 `~/.codex/skills`）以及 `~/.agents/skills`。若要涵蓋 Claude Code，請明確加入 `~/.claude/skills`。`--catalog catalog.json` 可傳入一個 JSON 陣列，內含已公告的 `SKILL.md` 路徑；`--include-plugins` 會加入 Codex 外掛快取，其中可能包含未啟用的版本。

盤點 JSON 是掃描器的輸出。審計報告 JSON 則加上了審查後的發現與建議；兩者是不同格式。基準測試範例包含的是**合成的示範資料**，不是實測效能結果。

## 決策是如何做出的

1. 先確認實際安裝或公告了什麼。光有磁碟上的檔案，並不能確定哪個版本是啟用中的。
2. 在情境中檢查訊號。共用的詞彙可能代表有用的專門技能；強制核准可能是恰當的；很長的技能可能包含有價值的領域知識。
3. 先修復具體的失敗。追蹤被引用的檔案到它實際的安裝位置，檢查呼叫參數，並保留輸出與憑證的存放位置。
4. 使用經授權的使用證據來挑選審查候選，而不是用它來證明過時。
5. 當檢查無法回答「指令是否有幫助」時，在匹配的條件下比較有意義的任務結果。未知的遙測資料保持未知，失敗的執行也保留在記錄中。
6. 只套用被要求的決定，編輯前先備份，更新依賴，並驗證最終的安裝狀態。

請參閱 [SKILL.md](SKILL.md)、[評估規範](references/evaluation.md)以及[資料契約](references/data-contracts.md)。

## 限制與隱私

- 盤點只讀取技能文字。它不會執行目標腳本、不會連線到它們的服務、預設不讀取對話紀錄，也不會修改已安裝的技能。
- 重複雜湊值與詞彙相似度只是審查線索，不是判決。核准與固定步驟用語只是檢查訊號，本身不是缺陷。
- 連結偵測涵蓋正文中的 Markdown 連結。圍欄（fenced）程式碼範例會被略過，慣用的 URL 佔位符會被標記為未解析，而任意程式碼路徑或間接依賴則需要在情境中檢查。
- 符號連結／junction 目錄以及常見的依賴／快取資料夾會被略過。必要時請明確提供解析後的目標路徑。未知的目錄路徑與讀取／解析錯誤會保持可見。
- 光是開新的 Agent 對話並不能證明隔離。如果基準組能載入目標技能，請回報比較結果為「無法判定」，或為隔離環境準備測試計畫。
- 報告可能包含本地路徑與摘錄。分享前請先審閱。原始對話紀錄、憑證、私人輸出與退役封存請放在本倉庫之外。

## 開發

```bash
python -m unittest discover -s tests -v
```

測試涵蓋唯讀盤點行為、路徑與輸出防護、格式錯誤的中繼資料、重複與使用情況處理、範例連結過濾、HTML 逸出、缺少指標、無效／受污染的執行，以及匹配比較。它們不會證明任何特定技能能提升任務表現。

CLI 工作流程測試也會從一個不相關的工作目錄執行，使用帶有敵意的測試夾具文字、一個可執行的輔助腳本、格式錯誤的 YAML，以及不完整的使用證據。它會檢查產生的盤點結果，並驗證輸入保持不變。

關於實際 Agent 試驗的範圍與限制，請見[驗證記錄](docs/validation.md)。

## 授權與致謝

Apache License 2.0；見 [LICENSE.txt](LICENSE.txt)。評估角色指令改編自 Anthropic 的 skill-creator。[ATTRIBUTION.md](ATTRIBUTION.md) 記錄了上游版本與變更。盤點與報告的實作是全新撰寫的；它不依賴 Claude CLI。
