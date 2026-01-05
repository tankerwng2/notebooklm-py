# Source List Command Improvements

**Date:** 2026-01-05
**Status:** Design Approved

## Overview

Improve the `source list` command to display source types and creation timestamps, matching the pattern established by the artifact list improvements.

## Current State

The `source list` command currently shows only:
- ID
- Title

## Design

### 1. Source Type Detection

Implement a multi-indicator detection system to identify source types from API data:

```python
def detect_source_type(src: list) -> str:
    """Detect source type from API data structure.

    Detection logic:
    - Check src[2][7] for YouTube/URL indicators
    - Check src[3][1] for type code
    - Check file size indicators at src[2][1]
    - Use title extension as fallback (.pdf, .txt, etc.)

    Returns:
        Display string with emoji (e.g., "🎥 YouTube")
    """
    type_code = src[3][1] if len(src) > 3 and isinstance(src[3], list) and len(src[3]) > 1 else 0

    # Check for URL at position [2][7] (YouTube/URL indicator)
    if len(src) > 2 and isinstance(src[2], list) and len(src[2]) > 7:
        url_field = src[2][7]
        if url_field and isinstance(url_field, list) and len(url_field) > 0:
            url = url_field[0]
            if 'youtube.com' in url or 'youtu.be' in url:
                return '🎥 YouTube'
            return '🔗 Web URL'

    # Check title for file extension
    title = src[1] if len(src) > 1 else ''
    if title:
        if title.endswith('.pdf'):
            return '📄 PDF'
        elif title.endswith(('.txt', '.md', '.doc', '.docx')):
            return '📝 Text File'
        elif title.endswith(('.xls', '.xlsx', '.csv')):
            return '📊 Spreadsheet'

    # Check for file size indicator (uploaded files have src[2][1] as size)
    if len(src) > 2 and isinstance(src[2], list) and len(src[2]) > 1:
        if isinstance(src[2][1], int) and src[2][1] > 0:
            return '📎 Upload'

    # Default to pasted text
    return '📝 Pasted Text'
```

### 2. Source Type Categories

**Supported types with emojis:**
- 🔗 Web URL - Regular web pages
- 🎥 YouTube - YouTube videos
- 📄 PDF - PDF files
- 📝 Text File - Text documents (.txt, .md, .doc, .docx)
- 📊 Spreadsheet - Excel/CSV files
- 📎 Upload - Generic uploaded files
- 📝 Pasted Text - Text pasted directly into NotebookLM
- ☁️ Google Drive - Google Drive files (if detected)

### 3. Table Structure

**New Column Order:**
- ID (existing)
- Title (existing)
- **Type** (new)
- **Created** (new)

**Implementation:**
```python
table = Table(title=f"Sources in {nb_id}")
table.add_column("ID", style="cyan")
table.add_column("Title", style="green")
table.add_column("Type")
table.add_column("Created", style="dim")

for src in sources_list:
    if isinstance(src, list) and len(src) > 0:
        # Extract ID
        src_id = src[0][0] if isinstance(src[0], list) else src[0]

        # Extract title
        title = src[1] if len(src) > 1 else "-"

        # Detect type
        type_display = detect_source_type(src)

        # Extract timestamp from src[2][2] - [seconds, nanoseconds]
        created = "-"
        if len(src) > 2 and isinstance(src[2], list) and len(src[2]) > 2:
            timestamp_list = src[2][2]
            if isinstance(timestamp_list, list) and len(timestamp_list) > 0:
                created = datetime.fromtimestamp(timestamp_list[0]).strftime("%Y-%m-%d %H:%M")

        table.add_row(str(src_id), title, type_display, created)
```

### 4. Edge Cases

1. **Missing timestamp**: Use "-" if src[2][2] is not a valid timestamp list
2. **Unknown type**: Display "❓ Unknown" if detection fails completely
3. **Empty sources list**: Keep existing behavior (no error, just empty table)
4. **Malformed data**: Gracefully handle missing indices with "-" placeholders
5. **Google Drive files**: Detect if Drive indicators are present in data

### 5. Example Output

```
Sources in nb_abc123
┌──────────┬────────────────────┬──────────────┬─────────────────┐
│ ID       │ Title              │ Type         │ Created         │
├──────────┼────────────────────┼──────────────┼─────────────────┤
│ src_123  │ Research Paper.pdf │ 📄 PDF       │ 2026-01-05 14:30│
│ src_124  │ Tutorial Video     │ 🎥 YouTube   │ 2026-01-05 15:15│
│ src_125  │ API Documentation  │ 🔗 Web URL   │ 2026-01-04 09:20│
│ src_126  │ My Notes          │ 📝 Pasted Text│ 2026-01-03 11:45│
└──────────┴────────────────────┴──────────────┴─────────────────┘
```

## Implementation Notes

- **Location**: `src/notebooklm/notebooklm_cli.py` lines 702-740
- **New function**: Add `detect_source_type()` near line 60-70 (near `ARTIFACT_TYPE_DISPLAY`)
- **Import**: Ensure `datetime` is imported (already present from artifact list work)
- **Consistency**: Matches the pattern established by artifact list improvements

## Benefits

1. **Visibility**: Users can see what type of content each source is
2. **Organization**: Easier to identify and manage different source types
3. **Timestamps**: Shows when sources were added for tracking
4. **Consistency**: Matches the improved artifact list design pattern
