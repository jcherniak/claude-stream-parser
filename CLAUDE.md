# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python stream parser that formats Claude's JSON output into human-readable terminal output. It's a single-file command-line utility with no external dependencies.

## Commands

### Running the Parser
```bash
./claude-stream-parser                    # Direct execution (file is executable)
python3 claude-stream-parser             # Run with Python
some_command | ./claude-stream-parser    # Pipe Claude's JSON output
```

### Testing
No test infrastructure exists. When implementing tests, consider:
- Using Python's unittest or pytest
- Testing JSON parsing edge cases
- Testing formatting functions individually
- Testing ANSI color output

## Architecture

The parser follows a simple stream processing pattern:
1. **Input**: JSON stream from stdin (line-by-line)
2. **Parsing**: `parse_and_display()` identifies message types
3. **Formatting**: Specialized formatters for different content types
4. **Output**: Color-coded terminal output with emojis

### Key Functions

- `parse_and_display()` (line 296): Main parser that routes messages to formatters
- `format_message_content()` (line 216): Formats assistant messages with tool calls
- `format_json_nicely()` (line 50): Smart JSON formatting with tool-specific handlers
- `format_todos()` (line 37): Special formatting for TodoWrite tool
- `colorize()` (line 8): ANSI color codes for terminal output

### Message Types Handled
- `system`: Init messages with session info
- `assistant`: Claude's responses with tool calls
- `user`: User inputs and tool results
- `result`: Final conversation results with metrics

## Special Features

1. **Tool-Specific Formatting**:
   - TodoWrite: Emoji status indicators (✅ ⏳ ⚪)
   - OpenRouter/Gemini tools: Never truncated, full content shown
   - Thinking tools: Progress tracking with step numbers
   - MCP tools: Grouped by server

2. **Smart Truncation**:
   - Most content truncated at 2000 chars
   - OpenRouter/Gemini content shown in full (10000 char limit)
   - Shows first/last lines for long content

3. **Error Handling**:
   - Permission errors highlighted specially
   - Graceful JSON parsing failures
   - Tool errors marked with ❌

## Recent Updates

### Improved Tool Call Display (Latest)
Tool calls now show cleaner, more readable output:
- **Bash**: Commands shown inline with description as comment
- **File tools** (Read/Write/Edit): Show file path inline
- **Grep**: Shows pattern and search path inline
- **TodoWrite**: Shows item count inline
- **Thinking tools**: Shows thought preview inline

### Improved Tool Result Display (Latest)
Tool results are now formatted more cleanly:
- **Single-line results**: Shown inline with arrow (→)
- **Thinking results**: Minimal "🧠 Thinking complete" message
- **Read results**: Minimal "File read." message (no file content shown)
- **WebSearch**: Formatted links with numbered list
- **Multi-line content**: Properly indented with smart truncation

### WebSearch Results Formatting (Fixed)
WebSearch results now display beautifully formatted links:
- Links section shows with 🔗 emoji
- Each link numbered with title and URL
- Section headers highlighted in color

## Development Tips

1. **Adding New Tool Formatters**:
   - Add special cases in `format_json_nicely()` (around line 50)
   - Check tool_name parameter for matching
   - Return formatted string with appropriate colors/emojis

2. **Color Codes**:
   - `'1;32'` - Bright green (success/assistant)
   - `'1;31'` - Bright red (errors)
   - `'1;33'` - Bright yellow (warnings/system)
   - `'1;34'` - Bright blue (user)
   - `'1;35'` - Bright magenta (tools)
   - `'1;36'` - Bright cyan (special info)
   - `'2;37'` - Dim white (metadata)

3. **Testing Changes**:
   - Capture Claude's JSON output: `claude_command > test.json`
   - Test parser: `cat test.json | ./claude-stream-parser`
   - Check ANSI output: Add `--color=always` if needed