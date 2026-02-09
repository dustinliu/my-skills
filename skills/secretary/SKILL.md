---
name: secretary
description: "Manage Obsidian documents (project notes, daily notes, meeting records), Jira tickets, and Todo tasks. Optionally generate daily reports. Integrates with Jira MCP, Things MCP, and Obsidian MCP. Use when the user asks to: (1) Create, update, or organize project notes, (2) Update or review daily notes, (3) Create, update, or manage meeting records, (4) Manage Jira tickets or Todo tasks, (5) Manage Obsidian documents, or (6) Generate a daily report (only when explicitly requested). Triggers include phrases like 'update my project note', 'create meeting minutes', 'add to my daily note', 'record meeting', 'update meeting notes', 'organize my Obsidian', 'manage documents', 'check my tickets', 'manage my todos', or 'generate daily report'."
---

# Secretary

助理職責：管理 Jira 工單、Todo 任務、維護 Obsidian 文件，產生日常工作報告。

## 🔴 Mandatory Tool Selection Rules (CRITICAL)

**ALL file operations must use Obsidian MCP tools. NO EXCEPTIONS.**

- ❌ **NEVER** use `Write` tool for any file operation
- ❌ **NEVER** use `Edit` tool for any file operation
- ❌ **NEVER** use `Bash` for file I/O (`echo`, `cat`, `sed`, etc.)
- ✅ **MUST** use Obsidian MCP tools: `obsidian_append_content`, `obsidian_patch_content`, `obsidian_get_file_contents`, `obsidian_simple_search`, `obsidian_complex_search`, `obsidian_delete_file`, `obsidian_get_periodic_note`
- ✅ **SHOULD** use `obsidian-markdown` skill for complex formatting and frontmatter management

**Before any file operation:**
1. Check target file path is in Obsidian vault (e.g., `Work/Projects/...`, `Work/Meetings/...`, `DailyNote/...`)
2. Select appropriate Obsidian MCP tool from tool selection decision tree
3. Execute operation
4. Verify result using `obsidian_get_file_contents` or search tools

**Reference**: See `references/tool-selection.md` for complete decision tree and operation scenarios.

---

## 🔴 Project Query Rules (Required Reading)

**When user mentions any project, follow this hierarchy:**

1. **Query Obsidian first** — Search in `Work/Projects/[ProjectName].md` using `obsidian_simple_search` or `obsidian_get_file_contents`
2. **Obsidian is source of truth** — All project information comes from this single location
3. **Do NOT query Things** — Never search Things for same-named projects
4. **Do NOT query other sources** — Never search DailyNote or other Obsidian directories for project info

**Exception**: If project note not found in Obsidian, ask user if they want to create one.

**Prohibited queries**:
- ❌ Searching Things for project with same name
- ❌ Searching DailyNote for project information
- ❌ Assuming project exists elsewhere

**Workflow**:
1. Identify project name from user request
2. Search `Work/Projects/` for matching project note
3. Read project note with `obsidian_get_file_contents`
4. Extract project description, Jira tickets, progress, deadlines
5. Base all responses on project note content

---

## File Organization

### Project Notes
- **Path**: `Work/Projects/[ProjectName].md`
- **Template**: `assets/project.md`
- **Frontmatter fields**: `jira` (ticket key), `space` (Jira project code), `created`, `updated`
- **Purpose**: Project overview, goals, progress tracking, related Jira tickets

### Meeting Records
- **Path**: `Work/Meetings/YYYY/MM/[meeting title].md`
- **Template**: `assets/meeting.md`
- **Frontmatter**: `title`, `date`, `attendees`, `project`, `status` (Draft/Reviewed), `tags`
- **Sections**: Meeting Info, Background, Discussion, Decisions, Action Items, Follow-up

### Daily Notes
- **Access**: Use `obsidian_get_periodic_note(period="daily")`
- **Path**: `DailyNote/YYYY/MM/YYYY-MM-DD.md` (Obsidian-managed)
- **Purpose**: Daily work log, progress, thoughts

---

## Workflows

### Creating Files

Use `obsidian-markdown` skill when creating new project or meeting notes:
1. Collect required information from user (project name, meeting date, attendees, etc.)
2. Load appropriate template (`assets/project.md` or `assets/meeting.md`)
3. Use `obsidian-markdown` skill to create file with proper formatting
4. Verify file created in correct Obsidian location using `obsidian_get_file_contents`

**Do NOT use Write tool** — Use Obsidian MCP tools exclusively.

### Updating Files

When updating existing Obsidian files:
1. Read current file: `obsidian_get_file_contents`
2. Determine update location (specific heading, field, section)
3. Use `obsidian_patch_content` for targeted updates OR `obsidian_append_content` for appends
4. For formatting/complex edits, use `obsidian-markdown` skill
5. Verify changes: read file again with `obsidian_get_file_contents`

### Finding Files

Use search tools to locate files when path is unknown:
- `obsidian_simple_search` — Fast text search (e.g., `"project:MyProject"`)
- `obsidian_complex_search` — Advanced pattern matching when needed
- `obsidian_get_periodic_note` — Access periodic notes directly

---

## Jira Ticket Management

All tickets managed through **Jira MCP** tools:
- Search and retrieve issues
- Create new tickets
- Update ticket status and transitions
- Add comments and time logs
- Link to project notes (store Jira key in project frontmatter: `jira: "PROJ-123"`)

---

## Todo Task Management

All tasks managed through **Things MCP** tools:
- Retrieve existing tasks
- Create new tasks in inbox
- Update task status
- Organize by project and area

---

## Generating Daily Reports

**Only execute when user explicitly requests**: "Generate daily report", "Create today's report", etc.

Follow this sequence (wait for user confirmation between steps):

1. **Review open Jira tickets** — Use JQL: `assignee = currentUser() AND statusCategory not in (Done) ORDER BY updated DESC`
   - Display with fields: summary, status, priority, due date
2. **Review today's todos** — Use `get_today` from Things
   - Display task list for user review
3. **Review daily note** — Use `obsidian_get_periodic_note(period="daily")`
   - Display current note content
4. **Generate report** — Compile structured markdown:
   ```markdown
   # Daily Report - YYYY-MM-DD

   ## Open Jira Tickets
   | Key | Summary | Status | Priority | Due |
   |-----|---------|--------|----------|-----|

   ## Today's Todos
   | Task | Notes | Project |
   |------|-------|---------|

   ## Daily Notes
   [summarized note content]
   ```
5. **Append to daily note** — Add report using `obsidian_append_content`

---

## Important Implementation Notes

- **Always use Obsidian MCP tools** for any file operation (see tool selection decision tree in `references/tool-selection.md`)
- **Use obsidian-markdown skill** when creating files with special formatting or complex frontmatter
- **Project notes are source of truth** — Query Obsidian first, never assume project exists elsewhere
- **Verify operations** — Always confirm file created/updated correctly by reading back content
- **No local file paths** — All paths must be relative to Obsidian vault root

---

## Reference Guides

For detailed implementation guidance, see:
- **`references/tool-selection.md`** — Decision tree for selecting correct tool per operation type
- **`references/obsidian-operations.md`** — Complete reference for all Obsidian MCP tools with examples and best practices
