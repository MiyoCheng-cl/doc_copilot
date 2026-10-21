# 引入 Grill（mattpocock/skills）強化 Plan/Coding/Testing 溝通嚴謹度

## Context

目前 Director 系統的嚴謹度集中在「執行後審查」（code-reviewer、test-plan-reviewer、test-code-reviewer、Blocker 分級）。miyo 觀察到比較表裡 Superpowers 那種「執行前先被盤問」的高流程約束目前缺席——任務一開始理解有歧義時，容易帶著錯誤假設直接動工，事後才在 review 階段被抓出來，成本較高。

目標：引入 mattpocock/skills 的 **Grill**（執行前釐清需求/設計），讓「盤問」發生在動工前而不是審查階段；並讓這件事**自動**發生在處理 project task 時，不用每次手動提醒。同時 miyo 要求追加兩個延伸機制：

1. 用「囚徒機制」概念追蹤 TDD 相關犯錯類型與評分，滾動式強化 Grill 的使用。
2. 對「用 Grill 溝通的問題類型」做 Lv0–Lv5 分級（哪類問題用 Grill 淨收益大、哪類不需要）。

已確認的變體選擇（miyo 已決定）：

- **`grill-me`**（預設）：快速釐清需求、臨時討論、一次性任務，不寫入 repo。
- **`grill-with-docs`**（升級）：中大型功能、架構設計、重要 trade-off、測試策略、需團隊共享脈絡的任務。只把「穩定且具長期價值」的結論寫進 `CONTEXT.md` / ADR；探索中的猜測、已推翻方案、瑣碎實作細節不落檔。

已確認來源與精確路徑（已用 GitHub API 查證，非猜測）：

- `skills/productivity/grill-me`
- `skills/engineering/grill-with-docs`
- Repo：`https://github.com/mattpocock/skills`

---

## Part A — 安裝機制

### A1. Claude Code 側（this repo，project-scoped）

```bash
npx skills@latest add https://github.com/mattpocock/skills --skill grill-me,grill-with-docs --agent claude-code -y
```

- 落地位置：`.claude/skills/grill-me/`、`.claude/skills/grill-with-docs/`
- 不加 `-g`，維持 project-scoped（只影響這個 repo，不動全域 `~/.claude/skills`）
- 執行後用 `npx skills list` 確認兩個 skill 都列為已安裝

### A2. Codex 側（依 miyo 指示，交由 Codex 自己的 skill-installer 執行）

環境裡已確認有全域 `codex` CLI（`codex-cli 0.146.0`）。用非互動模式委派給 Codex 自己的 `skill-installer` skill（`C:\Users\miyo_cheng.CLT\.codex\skills\.system\skill-installer\SKILL.md`），讓它用自己的腳本安裝到 `$CODEX_HOME/skills/`：

```bash
codex exec --skip-git-repo-check "Use the skill-installer skill. Run: scripts/install-skill-from-github.py --repo mattpocock/skills --path skills/productivity/grill-me --path skills/engineering/grill-with-docs"
```

- 這是 Codex 自己的技能系統（獨立於 `.claude/skills`），照 miyo 的要求用它原生的安裝流程，不是我用 vercel `skills` CLI 幫 Codex 裝。
- `skill-installer` 的腳本本身需要 network，sandbox 下可能要求 escalation——執行時若被要求批准，會先跟你確認。
- 完成後請求 Codex 用 `list-skills.py` 確認 `grill-me`、`grill-with-docs` 出現在已安裝清單。

### A3. 同步進 `.agents/skills/`（跟 Codex 共用的目錄）

`.agents/skills/` 目前每個項目都是指向 `.claude/skills/<name>` 的 symlink（現有慣例，例如 `test-code-author`、`tdd-developer-for-unit-test` 皆是）。裝完 A1 後，新增兩個同樣的 symlink：

```
.agents/skills/grill-me         -> .claude/skills/grill-me
.agents/skills/grill-with-docs  -> .claude/skills/grill-with-docs
```

這樣任何讀 `.agents/skills/`（而非 `.claude/skills/`）的工具（包含 Codex 在跑這個 repo 內任務時）也能看到 Grill。

---

## Part B — 讓「處理 project task 自動採用 Grill」

**不修改全域檔案**：`~/.claude/commands/director.md`、`~/.claude/director/*.md` 是跨 repo 共用的可攜式 Director 定義（Director-System-Portable）。改這裡會影響你其他 repo 的 Director 行為，超出這次「這個 repo 的 project task」的要求。改用這個 repo 既有的、Director 已經會自動載入的兩個 repo-scoped 檔案來掛勾：

### B1. `docs/personal-rules.md`

在現有的「何時要讀、讀哪個檔」表格新增一行，並在檔案下方新增完整章節「Grill 溝通輔助」，內容包含：

- 預設 `grill-me` / 升級 `grill-with-docs` 的判斷標準（沿用 miyo 上面確認的文字）
- 對照下方 Part C 的 Lv0–Lv5 表，指出哪個等級以上「必須」先 Grill

### B2. `AGENTS.md`

在「一般原則」下加一小段（維持 AGENTS.md 一貫的精簡風格，只放 pointer，不放細節）：

```
### 溝通嚴謹度（Grill）
處理 project task 前，先依 `docs/personal-rules.md` 的「Grill 溝通輔助」判斷是否需要先用 grill-me / grill-with-docs 釐清需求，避免帶著錯誤假設動工。
```

### B3. `.agent/rules/director-reflection.md`

新增一個章節，記錄這次的設計決策（符合 director-log 的「設計決策：選了 A 放棄 B，且有明確理由」標準），內容：

- 為什麼引入 Grill（執行前盤問 vs 執行後審查的成本差）
- 預設/升級判斷標準
- Lv0–Lv5 表（見 Part C）
- 指向新的犯錯 ledger 檔案（見 Part C）

同時在 `.agent/rules/director-log.md` 補一筆 `[ ]` 條目記錄這個決策，並依 Director 既有規則跑一次 `/distill`（這是 director skill 本來就有的步驟，不是新機制）。

---

## Part C — 滾動式強化機制（miyo 追加的兩個要求）

> ⚠️ 這兩點是全新概念，下面是我對「囚徒機制」的具體解讀，若跟你想的不一樣，請在這裡糾正，我會照修正後的版本再落地。

### C1. Lv0–Lv5：「用 Grill 溝通的問題類型」分級

沿用這個 repo 既有的分級慣例（`director-reflection.md` 裡已有「信心度分級」「Blocker 分級」，格式一致）：

| Lv  | 定義                                                                | 是否用 Grill                               | 範例                                  |
| --- | ------------------------------------------------------------------- | ------------------------------------------ | ------------------------------------- |
| Lv0 | 純機械操作，無理解歧義                                              | 不需要                                     | 改字串、格式化、rename                |
| Lv1 | 單檔小改動，需求清楚                                                | 不需要（除非 user 主動要求）               | typo fix、單一數值調整                |
| Lv2 | 涉及既有邏輯理解，有輕微歧義空間                                    | 建議 `grill-me`                            | bug fix 涉及邊界條件                  |
| Lv3 | 跨檔案/跨模組修改，需求描述有多種合理解讀                           | **必須** `grill-me`                        | 新增功能到既有 flow                   |
| Lv4 | 架構決策、trade-off 明顯、影響多人協作或跨 task 共用知識            | **必須** `grill-with-docs`                 | 新模型整合、跨工具統一性修改          |
| Lv5 | 高風險/不可逆決策（動 CI gate、共用元件、資安、既有 gate 門檻變更） | **必須** `grill-with-docs` + Director 審核 | 修改 `coding-style-alignment.md` 門檻 |

判斷規則：Lv3 以上「跳過 Grill 直接動工」視為 Part C2 的「背叛」事件，會被記錄。

### C2. TDD 犯錯類型收集與評分（「囚徒機制」）

**我的解讀**：把「該走 Grill/TDD 卻抄捷徑」視為重複賽局裡的「背叛（defect）」——短期省時間，長期若造成 bug 逃出或返工，會被追蹤記錄，累積到該 skill/agent 的信任分數；多次背叛後，該 skill/agent 在對應任務類型上被**強制**要求走 Grill/TDD，收回自主判斷空間（懲罰）；若之後持續乾淨（N 次無犯錯），可解除強制、恢復自主判斷（和解路徑）。這是「合作 vs 抄捷徑」的長期信任賽局，不是一次性扣分。

具體落地（骨架，細節交給 `/director-develop` 依 project 微調）：

1. **新檔案** `.agent/rules/grill-tdd-mistake-ledger.md`——append-only 記錄格式：
   ```
   - [YYYY-MM-DD] {skill/agent} | 錯誤類型: {分類} | 對應 task | 是否曾經該走 Grill/TDD 卻跳過 | 影響
   ```
2. **初版錯誤類型分類**（可擴充）：需求誤解（未 grill 就開工）、測試假綠（mock 過度）、Scope creep、跳過靜態門檻、忽略既有 pattern。
3. **評分規則**：同一 skill/agent 在同一錯誤類型累積到閾值（初版建議 3 次，交給 `/director-develop` 調整）→ 觸發「強制模式」：下次同類任務，Lv3 以上一律必須先 Grill，不得跳過；連續 N 次（初版建議 5 次）任務無犯錯 → 解除強制。
4. **掛入既有戰力表**：`/team-update <成員> performance` 更新時，若涉及 TDD/Grill 相關犯錯，同步寫入 ledger（在 `.claude/commands/team-update.md` 補一句提示，不改變其現有欄位結構）。
5. **滾動改進 Grill 本身**：仿照現有 `test-curator` 的做法——當 ledger 累積出跨 task 的共性犯錯模式時，產出建議更新 `grill-me`/`grill-with-docs` 的 `SKILL.md` 提問清單（例如「常犯：沒問清楚 edge case」就把這個問題固定加進 Grill 的提問模板）。這一步不在本次範圍內自動化，先讓 ledger 累積數據，等有足夠案例再手動觸發。

---

## 落地清單（待核准後執行）

**新裝/新建：**

- `.claude/skills/grill-me/`、`.claude/skills/grill-with-docs/`（npx 安裝）
- Codex `$CODEX_HOME/skills/grill-me`、`$CODEX_HOME/skills/grill-with-docs`（codex exec 委派安裝）
- `.agents/skills/grill-me`、`.agents/skills/grill-with-docs`（symlink）
- `.agent/rules/grill-tdd-mistake-ledger.md`（新檔，空骨架）

**修改：**

- `docs/personal-rules.md`（新表格行 + 新章節）
- `AGENTS.md`（一般原則下加短段落）
- `.agent/rules/director-reflection.md`（新章節：Grill 決策 + Lv0-5 表 + ledger pointer）
- `.agent/rules/director-log.md`（補一筆待蒸餾條目）
- `.claude/commands/team-update.md`（補一句 ledger 同步提示）

**不動：**

- `~/.claude/commands/director.md`、`~/.claude/director/*.md`（全域可攜式 Director 定義，理由見 Part B 開頭）

## 驗證方式

1. `npx skills list`（或檢查 `.claude/skills/` 目錄）確認兩個 skill 存在
2. 檢查 `.agents/skills/grill-me`、`grill-with-docs` symlink 可正確解析
3. 請 Codex 執行 `scripts/list-skills.py` 確認兩個 skill 出現在其已安裝清單
4. 讀取修改後的 `docs/personal-rules.md`、`AGENTS.md`、`director-reflection.md`，確認格式與既有章節一致、沒有破壞原有內容
5. 跑一次 `/distill` 確認新條目被正確蒸餾
