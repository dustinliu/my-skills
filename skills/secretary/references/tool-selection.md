# Tool Selection Decision Tree for Secretary

## Golden Rule

**ALL file operations must use Obsidian MCP tools. Never use Write, Edit, or Bash for file I/O.**

```
┌─────────────────────────────────────────────────────────┐
│ Task: File Operation                                     │
└────────────────┬────────────────────────────────────────┘
                 │
        ┌────────▼─────────┐
        │ Is target in     │
        │ Obsidian vault?  │
        └────────┬─────────┘
                 │
        ┌────────┴────────┐
        │                 │
       YES               NO
        │                 │
        │                 └─► ❌ STOP - Cannot proceed
        │                    (Outside Obsidian vault)
        │
   ┌────▼───────────────────┐
   │ What's the operation?  │
   └────┬────────────────────┘
        │
   ┌────┴──────────────┬──────────────┬─────────────────┐
   │                   │              │                 │
 CREATE              READ           UPDATE            DELETE
   │                   │              │                 │
   │                   │              │                 │
  USE:                USE:            USE:              USE:
  ├─ obsidian_       ├─ obsidian_    ├─ obsidian_      ├─ obsidian_
  │  append_content   │  get_file_    │  patch_content   │  delete_file
  │                   │  contents     │                  │
  │  OR               │               │  OR              │
  │  obsidian_        │  obsidian_    │  obsidian_       │
  │  patch_content    │  simple_      │  append_content  │
  │  (prepend mode)   │  search       │  (append mode)   │
  │                   │               │                  │
  │  OR               │  obsidian_    │  obsidian_       │
  │  obsidian-        │  complex_     │  patch_content   │
  │  markdown skill   │  search       │  (replace mode)  │
  │  (for formatting) │               │                  │
  │                   │               │  obsidian-       │
  │  Then verify:     │  Then verify: │  markdown skill  │
  │  ✓ Path is in     │  ✓ Path is in │  (for formatting)│
  │    Obsidian       │    Obsidian   │                  │
  │  ✓ File created   │  ✓ Content    │  Then verify:    │
  │                   │    retrieved  │  ✓ Path is in    │
  │                   │               │    Obsidian      │
  │                   │               │  ✓ File updated  │
  │                   │               │    correctly     │
  └──────────────────┴───────────────┴──────────────────┴─────────────────┘
```

## Detailed Scenarios

### Scenario 1: Create New Obsidian File

**Task**: Create a new project note, meeting record, or daily note

**Decision Path**:
1. Determine file location (must be in Obsidian vault)
   - Project notes: `Work/Projects/[ProjectName].md`
   - Meeting records: `Work/Meetings/YYYY/MM/[title].md`
   - Daily notes: `DailyNote/YYYY/MM/YYYY-MM-DD.md`
2. Prepare content (use templates from `assets/`)
3. **Tool to use**: `obsidian_append_content`
4. **Verification**:
   - ✅ File path starts with vault root structure
   - ✅ File created in correct location
   - ✅ Content matches expected template

**Example**:
```python
obsidian_append_content(
    filepath="Work/Projects/MyProject.md",
    content="[frontmatter and content]"
)
```

### Scenario 2: Update Existing Obsidian File

**Task**: Add notes, update status, modify content in existing file

**Decision Path**:
1. Determine operation type
   - Append to end: use `obsidian_append_content`
   - Replace section: use `obsidian_patch_content` with target heading/block
   - Prepend to file: use `obsidian_patch_content` with prepend mode
2. **Tool to use**: `obsidian_append_content` or `obsidian_patch_content`
3. **Verification**:
   - ✅ Target file exists in Obsidian vault
   - ✅ Content inserted at correct location
   - ✅ File format preserved (frontmatter, markdown syntax)

**Example - Append**:
```python
obsidian_append_content(
    filepath="Work/Projects/MyProject.md",
    content="\n## New Section\nContent here"
)
```

**Example - Patch with heading target**:
```python
obsidian_patch_content(
    filepath="Work/Projects/MyProject.md",
    operation="append",
    target_type="heading",
    target="## Notes",
    content="- New note added"
)
```

### Scenario 3: Read from Obsidian File

**Task**: Retrieve content from project notes, meeting records, daily notes

**Decision Path**:
1. Determine which file to read
   - If searching for file: use `obsidian_simple_search` or `obsidian_complex_search`
   - If path is known: use `obsidian_get_file_contents` directly
   - For periodic notes (daily/weekly/etc): use `obsidian_get_periodic_note`
2. **Tool to use**: `obsidian_get_file_contents`, `obsidian_simple_search`, `obsidian_complex_search`, or `obsidian_get_periodic_note`
3. **Verification**:
   - ✅ File exists in vault
   - ✅ Content retrieved correctly
   - ✅ All required fields/sections found

**Example - Direct read**:
```python
obsidian_get_file_contents(filepath="Work/Projects/MyProject.md")
```

**Example - Search for file**:
```python
obsidian_simple_search(query="project:MyProject")
```

**Example - Get today's note**:
```python
obsidian_get_periodic_note(period="daily")
```

### Scenario 4: Delete from Obsidian

**Task**: Remove a file from vault

**Decision Path**:
1. Confirm file location in vault
2. Confirm this is intentional (dangerous operation)
3. **Tool to use**: `obsidian_delete_file` with `confirm: true`
4. **Verification**:
   - ✅ File was actually in Obsidian vault
   - ✅ File no longer exists after deletion

**Example**:
```python
obsidian_delete_file(
    filepath="Work/Projects/OldProject.md",
    confirm=True
)
```

## Forbidden Tools & Patterns

❌ **NEVER use these**:
- `Write` tool for any file operation
- `Edit` tool for any file operation
- `Bash` commands like `echo`, `cat`, `sed`, `awk` for file I/O
- Hardcoded local paths (e.g., `/Users/...`)
- Direct file system operations

**Why**: These bypass Obsidian's vault structure and create orphaned files outside the vault.

## Obsidian-Markdown Skill Integration

When creating or heavily formatting Obsidian files, use the `obsidian-markdown` skill:
- For complex formatting (wikilinks, callouts, properties)
- When creating files with special Obsidian syntax
- For frontmatter management

The `obsidian-markdown` skill internally uses Obsidian MCP tools, ensuring compliance.

## Operation Type Quick Reference

| Operation | Primary Tool | Secondary Tool | When to Use |
|-----------|--------------|-----------------|-------------|
| Create file | `obsidian_append_content` | `obsidian-markdown` | New project/meeting note |
| Append to file | `obsidian_append_content` | - | Add notes to existing file |
| Replace section | `obsidian_patch_content` | `obsidian-markdown` | Update specific heading block |
| Prepend to file | `obsidian_patch_content` (prepend) | - | Add to top of file |
| Read file | `obsidian_get_file_contents` | - | Get content for review/analysis |
| Search files | `obsidian_simple_search` | `obsidian_complex_search` | Find files by content/metadata |
| Get periodic note | `obsidian_get_periodic_note` | - | Retrieve daily/weekly/monthly notes |
| Delete file | `obsidian_delete_file` | - | Remove from vault (rare) |

## Verification Checklist

Before considering any file operation complete:

- [ ] Target file path verified to be in Obsidian vault
- [ ] Correct MCP tool used (not Write/Edit/Bash)
- [ ] Operation completed successfully (no errors)
- [ ] Content placed in correct location
- [ ] File format preserved (frontmatter, markdown, properties)
- [ ] No local directory paths used
