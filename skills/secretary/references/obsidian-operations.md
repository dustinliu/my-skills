# Obsidian Operations Guide

Complete reference for all Obsidian MCP operations used by Secretary skill. Use this guide when implementing file operations.

## File Operations Overview

### obsidian_append_content

Append content to file (creating if it doesn't exist).

**Usage**:
```python
obsidian_append_content(
    filepath="Work/Projects/MyProject.md",
    content="\n## New Section\nContent here"
)
```

**Parameters**:
- `filepath` (required): Path relative to vault root (e.g., `Work/Projects/MyProject.md`)
- `content` (required): Content to append (include newline prefix if needed)

**Best practices**:
- Use for appending new content to existing files
- For new files, content becomes the initial file content
- Always include leading `\n` when appending to existing files
- Don't include trailing newline unless you want blank lines

**Common use cases**:
- Add notes to project file: `\n\n## Notes\n[content]`
- Add daily note entries: `\n- [timestamp] [note]`
- Record meeting minutes: append with proper formatting

---

### obsidian_patch_content

Insert, replace, or modify content at specific locations (heading, block, frontmatter).

**Usage - Append to heading**:
```python
obsidian_patch_content(
    filepath="Work/Projects/MyProject.md",
    operation="append",
    target_type="heading",
    target="## Notes",
    content="\n- New note"
)
```

**Usage - Replace frontmatter field**:
```python
obsidian_patch_content(
    filepath="Work/Projects/MyProject.md",
    operation="replace",
    target_type="frontmatter",
    target="updated",
    content="2026-02-09"
)
```

**Parameters**:
- `filepath` (required): Path relative to vault root
- `operation` (required): `append`, `prepend`, or `replace`
- `target_type` (required): `heading`, `block`, or `frontmatter`
- `target` (required): Heading path (e.g., `## Notes`), block reference ID, or frontmatter field name
- `content` (required): Content to insert/replace

**Target types explained**:

**heading**: Document section
- Use heading hierarchy: `## Section` or `### Subsection`
- Operation `append`: adds content after heading
- Operation `prepend`: adds content before heading's existing content
- Operation `replace`: replaces heading's content (not the heading itself)

**block**: Specific content block with reference
- Use block ID: `^block-id`
- Operation `append`: adds after block
- Operation `replace`: replaces block content
- Block references less common in Secretary (use heading instead)

**frontmatter**: YAML frontmatter fields
- Project notes: `jira`, `space`, `created`, `updated`
- Meeting records: `title`, `date`, `status`, `project`, `attendees`
- Operation `replace`: updates field value only

**Best practices**:
- For structured updates, use heading targets
- Always verify heading exists before patching
- Use for updating metadata/frontmatter
- Combine multiple patches for complex edits
- Remember: operation `replace` on heading replaces heading CONTENT, not the heading itself

---

### obsidian_get_file_contents

Read complete file content from vault.

**Usage**:
```python
obsidian_get_file_contents(filepath="Work/Projects/MyProject.md")
```

**Parameters**:
- `filepath` (required): Path relative to vault root
- `limit` (optional): Number of lines to read (default: all)
- `offset` (optional): Starting line number (default: 0)

**Best practices**:
- Use when you know exact file path
- For large files, use `limit` and `offset` to read in chunks
- Returns content with line numbers (format: `  1→[content]`)
- Use to verify file structure before patching

---

### obsidian_simple_search

Basic text search across all vault files.

**Usage**:
```python
obsidian_simple_search(
    query="project:MyProject",
    context_length=100
)
```

**Parameters**:
- `query` (required): Search term (simple text, no regex)
- `context_length` (optional): Characters of context around match (default: 100)

**Best practices**:
- Use for finding files by content/metadata
- Great for locating project notes: `project:ProjectName`
- Great for finding meetings: `meeting` or `meeting:ProjectName`
- Returns files with context around matches
- Slower than direct path lookup but useful for discovery

**Common patterns**:
- Find project: `Work/Projects/[ProjectName]`
- Find meeting: `meeting:ProjectName` or `Date`
- Find by tag: `tag:meeting` or `tag:project:ProjectName`

---

### obsidian_complex_search

Advanced search using JsonLogic with pattern matching.

**Usage - Find all project notes**:
```python
obsidian_complex_search(
    query={
        "glob": ["Work/Projects/*.md", {"var": "path"}]
    }
)
```

**Usage - Find by frontmatter field**:
```python
obsidian_complex_search(
    query={
        "and": [
            {"glob": ["Work/Projects/*.md", {"var": "path"}]},
            {"in": ["project:MyProject", {"var": "frontmatter"}]}
        ]
    }
)
```

**Parameters**:
- `query` (required): JsonLogic query object

**Best practices**:
- Use for complex multi-condition searches
- Supports glob patterns and regexp
- Returns file paths matching query
- More powerful but slower than simple_search
- Usually overkill for Secretary use cases (use simple_search first)

**Common patterns**:
- All project notes: `{"glob": ["Work/Projects/*.md", {"var": "path"}]}`
- All meeting notes: `{"glob": ["Work/Meetings/**/*.md", {"var": "path"}]}`
- Files with specific tag: Combine glob with content matching

---

### obsidian_get_periodic_note

Retrieve periodic notes (daily, weekly, monthly, etc).

**Usage**:
```python
obsidian_get_periodic_note(period="daily")
```

**Parameters**:
- `period` (required): `daily`, `weekly`, `monthly`, `quarterly`, or `yearly`

**Returns**:
- Content of current periodic note for that period
- If note doesn't exist, may return empty

**Best practices**:
- Use for accessing daily notes without knowing exact path
- Respects Obsidian's periodic note settings
- Great for daily report workflow
- Simpler than searching for date-based paths

**Common use cases**:
- Get today's note: `period="daily"`
- Get this week's note: `period="weekly"`
- Get this month's overview: `period="monthly"`

---

### obsidian_get_recent_changes

Get recently modified files.

**Usage**:
```python
obsidian_get_recent_changes(
    days=7,
    limit=10
)
```

**Parameters**:
- `days` (optional): Look back N days (default: 90)
- `limit` (optional): Max files to return (default: 10, max: 100)

**Best practices**:
- Use to find recently active projects
- Helpful for daily report generation
- Returns file list, not content
- Combine with `obsidian_get_file_contents` to read files

---

### obsidian_delete_file

Delete file from vault.

**Usage**:
```python
obsidian_delete_file(
    filepath="Work/Projects/OldProject.md",
    confirm=True
)
```

**Parameters**:
- `filepath` (required): Path relative to vault root
- `confirm` (required): Must be `True` to actually delete

**Best practices**:
- Rarely needed (archiving preferred over deletion)
- Always confirm file path before deletion
- Consider archiving instead of deleting
- This is destructive - use carefully

---

## File Structure Reference

### Project Notes

**Location**: `Work/Projects/[ProjectName].md`

**Template**:
```yaml
---
jira: "PROJ-123"
space: "PROJ"
created: "2026-02-09"
updated: "2026-02-09"
---

# Project Name

## Overview
Brief project description and goals.

## Notes
Additional notes and progress updates.
```

**Operations**:
- Create: `obsidian_append_content` with full template
- Update: `obsidian_patch_content` to target specific sections
- Read: `obsidian_get_file_contents` or search

---

### Meeting Records

**Location**: `Work/Meetings/YYYY/MM/[meeting title].md`

**Template**:
```yaml
---
title: "Meeting Title"
date: "2026-02-09"
attendees: []
project: "ProjectName"
status: "Draft"
tags: ["meeting", "project:ProjectName"]
---

# Meeting Title

## Meeting Info
- **Date**: 2026-02-09
- **Location/Platform**: [location]
- **Attendees**: [list]
- **Duration**: [duration]

## Background
Context and goals.

## Discussion
[Topics discussed]

## Decisions
[Decisions made]

## Action Items
[Table of action items]

## Follow-up
[Next steps]
```

**Operations**:
- Create: Use `obsidian-markdown` skill to ensure proper formatting
- Update: Patch specific sections (Discussion, Decisions, Action Items)
- Update status: Replace frontmatter `status` field

---

### Daily Notes

**Location**: Auto-managed by Obsidian (typically `DailyNote/YYYY/MM/YYYY-MM-DD.md`)

**Access**:
- Use `obsidian_get_periodic_note(period="daily")` to get today's note
- Use `obsidian_append_content` to add daily entries

**Operations**:
- Read: `obsidian_get_periodic_note`
- Append: `obsidian_append_content` with date-prefixed entries
- Search: `obsidian_simple_search` by date

---

## Common Workflows

### Workflow: Create New Project Note

```
1. obsidian_append_content(
     filepath="Work/Projects/ProjectName.md",
     content="""---
jira: ""
space: ""
created: "2026-02-09"
updated: "2026-02-09"
---

# Project Name

## Overview


## Notes
"""
   )

2. Verify file created: obsidian_get_file_contents
3. Confirm success
```

### Workflow: Update Project Status

```
1. Read project: obsidian_get_file_contents("Work/Projects/ProjectName.md")
2. Check current content
3. Patch updated date: obsidian_patch_content(
     target_type="frontmatter",
     target="updated",
     operation="replace",
     content="2026-02-09"
   )
4. Append status update: obsidian_patch_content(
     target_type="heading",
     target="## Notes",
     operation="append",
     content="\n- [2026-02-09] Status updated..."
   )
5. Verify changes
```

### Workflow: Record Meeting Notes

```
1. Create meeting file: obsidian_append_content(
     filepath="Work/Meetings/2026/02/Project Planning.md",
     content=[full meeting template]
   )
2. Use obsidian-markdown skill for formatting
3. Verify file created in correct location
4. Can later update status from Draft → Reviewed
```

### Workflow: Generate Daily Report

```
1. Get daily note: obsidian_get_periodic_note(period="daily")
2. Get recent changes: obsidian_get_recent_changes(days=1)
3. Get project notes: obsidian_simple_search(query="project:")
4. Compile into report format
5. Append to daily note or create report file
```

---

## Error Handling

**File not found**:
- `obsidian_get_file_contents` returns error if path wrong
- Use `obsidian_simple_search` to find correct path first
- Double-check path structure: `Work/Projects/[ProjectName].md`

**Path outside vault**:
- If path doesn't start with expected vault structure, file is not in Obsidian
- Never attempt to create/edit files outside vault
- Redirect to create in appropriate vault location

**Frontmatter errors**:
- Ensure frontmatter fields match expected types
- For arrays (like `attendees`), use proper YAML syntax
- Keep date format consistent: `YYYY-MM-DD`

**Encoding issues**:
- All Obsidian files should be UTF-8
- Special characters generally safe in markdown
- Test with emojis/unicode if concerns

---

## Performance Notes

- `obsidian_append_content`: Fast, good for new files
- `obsidian_patch_content`: Moderate, slower for large files
- `obsidian_get_file_contents`: Fast for reading
- `obsidian_simple_search`: Slower (full text search)
- `obsidian_get_periodic_note`: Fast (indexed access)

For efficiency:
- Use direct path if you know it
- Use `simple_search` for discovery
- Batch operations when possible (but keep files well-organized)
