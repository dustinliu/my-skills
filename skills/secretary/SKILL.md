---
name: secretary
description: "Manage Obsidian documents (project notes, daily notes, meeting records), Jira tickets, and Todo tasks. Optionally generate daily reports. Integrates with Jira MCP, Things MCP, and Obsidian MCP. Use when the user asks to: (1) Create, update, or organize project notes, (2) Update or review daily notes, (3) Create, update, or manage meeting records, (4) Manage Jira tickets or Todo tasks, (5) Manage Obsidian documents, or (6) Generate a daily report (only when explicitly requested). Triggers include phrases like 'update my project note', 'create meeting minutes', 'add to my daily note', 'record meeting', 'update meeting notes', 'organize my Obsidian', 'manage documents', 'check my tickets', 'manage my todos', or 'generate daily report'."
---

# Secretary

助理職責：管理 Jira 工單、Todo 任務、維護 Obsidian 文件，產生日常工作報告。

## Jira 工單管理

所有工單都通過 **Jira** 管理。使用 Jira MCP 工具來：

- 搜尋和檢索議題
- 建立新工單
- 更新工單狀態和轉換
- 新增評論和工時日誌

## Todo 任務管理

所有待辦事項都通過 **Things MCP** 管理。使用 Things MCP 工具來：

- 檢索現有任務
- 建立新任務到收件匣
- 更新任務狀態
- 按專案和區域組織（`get_projects`、`get_areas`, `get_anytime`）

## Obsidian 文件管理

所有文件都通過 **Obsidian** 維護。支援的文件類型：

### 專案筆記
- **位置**：`Work/Projects/[ProjectName].md`
- **範本**：`assets/project.md`
- **屬性**：`jira`（工單鍵值）、`space`（Jira 專案碼）、`created`、`updated`
- **用途**：追蹤專案概況、進度和相關筆記

### 每日筆記
- **位置**：`DailyNote/YYYY/MM/YYYY-MM-DD.md`
- **取得方法**：`mcp__mcp-obsidian__obsidian_get_periodic_note` with `period: "daily"`
- **用途**：記錄每日工作、進度和想法

### 會議記錄
- **位置**：`Work/Meetings/YYYY/MM/[meeting title].md`
- **範本**：`assets/meeting.md`
- **屬性**：
  - `title` — 會議標題
  - `date` — 會議日期
  - `attendees` — 參與人員列表
  - `project` — 相關專案
  - `status` — 狀態 (`Draft` 或 `Reviewed`)
  - `tags` — 標籤（如 `meeting`、`project:ProjectName`）
- **包含**：背景、討論、決策、行動項目、後續追蹤
- **用途**：記錄會議重點、決策和責任分配

#### 會議記錄工作流程

建立或編輯會議記錄時：

1. **收集資訊**：詢問使用者會議相關信息（日期、參與者、專案、主題）
2. **建立或更新文件**：使用範本建立新筆記（預設 `status: Draft`），或更新現有筆記
3. **建議標籤**：根據會議內容建議標籤（例如 `project:ProjectName`、`type:planning`）
4. **驗證行動項目**：確認所有行動項目有明確的責任人和截止日期
5. **更新狀態**：
   - **Draft** — 初始建立後的狀態，表示筆記還在編輯中
   - **Reviewed** — 會議內容已確認和審核，可供參考

**編輯所有 Obsidian 文件時，始終使用 `obsidian-markdown` skill 確保正確的語法。**

## 產生每日報告（可選）

**只有在使用者明確要求時，才執行此工作流程。** 例如："Generate today's daily report" 或 "Create a daily report"。

### 每日報告工作流程

按序依次完成以下步驟，每個步驟完成後等待使用者確認：

#### 第一步：檢視未結 Jira 工單

使用 `mcp__atlassian__searchJiraIssuesUsingJql` 搭配 JQL：
```
assignee = currentUser() AND statusCategory not in (Done) ORDER BY updated DESC
```

Fields：`["summary","status","issuetype","priority","created","updated","duedate","project"]`，maxResults：`50`

展示工單列表供使用者 review、更新工單狀態或備註。

#### 第二步：檢視今日待辦清單

使用 `mcp__things__get_today` 檢索今日應完成的任務。

展示任務列表並請使用者 review、完成或更新任務狀態。

#### 第三步：檢視 Obsidian 每日筆記

使用 `mcp__mcp-obsidian__obsidian_get_periodic_note` 並設定 `period: "daily"` 來檢索今日筆記。

根據筆記內容協助整理或更新。

#### 第四步：產生每日報告

根據 Jira 工單、Todo 任務和 Obsidian 筆記，產生結構化的日報：

```markdown
# 每日報告 - {date}

## Jira 工單
| 鍵值 | 摘要 | 狀態 | 優先度 | 截止日期 |
|-----|------|------|--------|---------|
| {ticket data} |

## 今日任務
| 任務 | 備註 | 專案/區域 |
|------|------|---------|
| 標題 | 備註摘要 | 專案或區域 |

## Obsidian 筆記
{每日筆記的摘要內容，如可用}
```

## **重要規則**

- 編輯 Obsidian 筆記時，必須使用但不限於 `obsidian-markdown` skill 確保正確的語法。
- 當提到專案時，一律以 Obsidian 底下的專案筆記為 Source of Truth，除非有特別其他的要求。
