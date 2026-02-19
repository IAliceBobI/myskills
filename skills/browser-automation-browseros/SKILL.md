---
name: browser-automation-browseros
description: Advanced browser automation using BrowserOS HTTP MCP interface with 31 tools for navigation, interaction, content extraction, and automation. Use when automating repetitive browser tasks, scraping authenticated pages, testing web applications, filling forms, scheduling periodic browser checks, or monitoring websites for changes. Optimized for BrowserOS with local model support, visual workflows, and privacy-first design.
---

# Browser Automation for BrowserOS

## Purpose

Automate browser tasks using BrowserOS HTTP MCP interface with 31 powerful tools for navigation, interaction, content extraction, and automation. Leverage your existing BrowserOS session with logged-in accounts, cookies, and browser history.

## Quick Start

### Prerequisites

1. **BrowserOS running**: Ensure BrowserOS application is open
2. **MCP configured**: BrowserOS MCP server at `http://127.0.0.1:9106/mcp`
3. **Check connection**:
   ```bash
   curl http://127.0.0.1:9106/mcp
   ```

### Core Pattern

```
Stable Browser Automation = Verify → Act → Confirm → Repeat

1. **Verify** - Always check load_status before interacting
2. **Act** - Use precise nodeId (not coordinates)
3. **Confirm** - Re-fetch elements to verify action succeeded
4. **Repeat** - Continue pattern for multi-step workflows
```

**Why this matters**:
- **Dynamic pages** - Content may load asynchronously
- **Interactive elements** - May change after navigation
- **Errors happen** - Popups, delays, network issues
- **Reliability** - Pattern works consistently across sites

### Basic Workflow

```
1. Navigate → 2. Verify Load → 3. Get Elements → 4. Interact → 5. Confirm
```

**Example**:
```bash
# 1. Navigate
mcp__browser-mcp__browser_navigate?url=https://example.com

# 2. Wait for page load
mcp__browser-mcp__browser_get_load_status?tabId=<from_navigation>

# 3. Get interactive elements
mcp__browser-mcp__browser_get_interactive_elements?tabId=<id>

# 4. Click element (using nodeId from step 3)
mcp__browser-mcp__browser_click_element?nodeId=<nodeId>&tabId=<id>

# 5. Verify result
mcp__browser-mcp__browser_get_interactive_elements?tabId=<id>
```

## Available BrowserOS MCP Tools

### Navigation (3 tools)
- `browser_navigate` - Open URL
- `browser_open_tab` - Open new tab
- `browser_close_tab` - Close tab

### Tab Management (6 tools)
- `browser_switch_tab` - Switch to tab
- `browser_list_tabs` - List all tabs
- `browser_group_tabs` - Group tabs
- `browser_ungroup_tabs` - Ungroup tabs
- `browser_get_active_tab` - Get active tab
- `browser_create_window` - Create new window

### Scrolling (3 tools)
- `browser_scroll_down` - Scroll down one viewport
- `browser_scroll_up` - Scroll up one viewport
- `browser_scroll_to_element` - Scroll element into view

### Interaction (4 tools)
- `browser_click_element` - Click element by nodeId
- `browser_type_text` - Type text into input
- `browser_clear_input` - Clear input field
- `browser_send_keys` - Send keyboard keys

### Content Extraction (3 tools)
- `browser_get_page_content` - Get page text/links
- `browser_get_interactive_elements` - Get all clickable elements
- `browser_grep_interactive_elements` - Search elements by regex

### Screenshots (2 tools)
- `browser_get_screenshot` - Capture page screenshot
- `browser_get_screenshot_pointer` - Screenshot with pointer overlay

### Data Management (10 tools)
- `browser_get_bookmarks` - List bookmarks
- `browser_create_bookmark` - Create bookmark
- `browser_remove_bookmark` - Remove bookmark
- `browser_get_recent_history` - Recent history
- `browser_search_history` - Search history
- And more...

**Total: 31 tools** - See [Tools List](./references/tools-list.md) for complete reference.

## Best Practices

### DO

- **Always verify load_status** before interacting
  ```
  mcp__browser-mcp__browser_get_load_status?tabId=1697399534
  ```
  Check: `isPageComplete == true`

- **Re-fetch elements** after navigation or actions
  ```
  mcp__browser-mcp__browser_get_interactive_elements?tabId=1697399534
  ```

- **Use nodeId** not coordinates for clicks
  ```
  mcp__browser-mcp__browser_click_element?nodeId=15
  ```

- **Screenshot** for debugging and evidence
  ```
  mcp__browser-mcp__browser_get_screenshot?tabId=1697399534
  ```

- **Use grep** for finding elements
  ```
  mcp__browser-mcp__browser_grep_interactive_elements?tabId=1697399534&pattern=submit|login|button
  ```

### DON'T

- **Don't skip load_status** check - always verify before acting
- **Don't assume elements exist** without fetching
- **Don't click coordinates** (fragile) - use nodeId instead
- **Don't forget to re-verify** after actions

## Example: Form Automation

```bash
# 1. Navigate to form
mcp__browser-mcp__browser_navigate?url=https://example.com/form

# 2. Wait for load
mcp__browser-mcp__browser_get_load_status?tabId=<id>

# 3. Get interactive elements
mcp__browser-mcp__browser_get_interactive_elements?tabId=<id>&simplified=true

# 4. Fill form fields (use nodeId from step 3)
mcp__browser-mcp__browser_type_text?nodeId=15&text=user@example.com
mcp__browser-mcp__browser_type_text?nodeId=18&text=secret123

# 5. Click submit
mcp__browser-mcp__browser_click_element?nodeId=22&tabId=<id>

# 6. Verify success
mcp__browser-mcp__browser_get_load_status?tabId=<id>
mcp__browser-mcp__browser_get_page_content?tabId=<id>
```

## Common Troubleshooting

### Element Not Found

```bash
# 1. Re-fetch elements
mcp__browser-mcp__browser_get_interactive_elements?tabId=<id>

# 2. Check if page loaded
mcp__browser-mcp__browser_get_load_status?tabId=<id>

# 3. Use grep to find element
mcp__browser-mcp__browser_grep_interactive_elements?tabId=<id>&pattern=<text>
```

### Click Not Working

```bash
# 1. Scroll element into view
mcp__browser-mcp__browser_scroll_to_element?nodeId=15

# 2. Take screenshot to see current state
mcp__browser-mcp__browser_get_screenshot?tabId=<id>

# 3. Check for blocking modals
mcp__browser-mcp__browser_grep_interactive_elements?tabId=<id>&pattern=Close|Dismiss
```

### Session Expired

```bash
# Detect login page in content
mcp__browser-mcp__browser_get_page_content?tabId=<id>

# Re-authenticate if needed
mcp__browser-mcp__browser_navigate?url=https://example.com/login
# ... fill credentials and submit
```

## Scripts

Utility scripts are available in `scripts/` directory:

| Script | Purpose |
|--------|---------|
| `check_connection.sh` | Verify BrowserOS MCP connection |
| `extract_form.py` | Extract form fields from HTML |
| `screenshot_on_error.sh` | Auto-capture screenshots on errors |

## References

For detailed information, see:

- **[Tools List](./references/tools-list.md)** - Complete list of 31 BrowserOS MCP tools
- **[Advanced Patterns](./references/advanced-patterns.md)** - Complex automation patterns, batch processing, error recovery
- **[Error Handling](./references/error-handling.md)** - Comprehensive troubleshooting guide

## Quick Reference

### BrowserOS MCP Server

- **Default URL**: `http://127.0.0.1:9106/mcp`
- **Connection check**: `curl http://127.0.0.1:9106/mcp`

### Tab ID Management

- **From navigation**: Save tabId from `browser_navigate` response
- **From active tab**: Use `browser_get_active_tab`
- **List all tabs**: Use `browser_list_tabs`

### Element Finding

```bash
# Get all interactive elements
mcp__browser-mcp__browser_get_interactive_elements?tabId=1697399534

# Search for specific elements
mcp__browser-mcp__browser_grep_interactive_elements?tabId=1697399534&pattern=login|submit
```
