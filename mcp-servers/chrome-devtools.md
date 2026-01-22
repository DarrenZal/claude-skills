# Chrome DevTools MCP Server

Connect to Chrome DevTools for debugging and browser interaction.

## Installation

No installation needed - uses npx.

```bash
claude mcp add -s user chrome-devtools -- npx chrome-devtools-mcp@latest
```

## Prerequisites

Chrome must be running with remote debugging enabled:

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Or add to Chrome shortcut
```

## Tools

| Tool | Description |
|------|-------------|
| `list_pages` | List open browser tabs |
| `select_page` | Select a tab to work with |
| `navigate_page` | Navigate to URL |
| `take_snapshot` | Get accessibility snapshot |
| `take_screenshot` | Take screenshot |
| `click` | Click element |
| `fill` | Fill input field |
| `evaluate_script` | Run JavaScript |
| `list_console_messages` | Get console output |
| `list_network_requests` | Get network activity |
| `performance_start_trace` | Start performance trace |
| `performance_stop_trace` | Stop and analyze trace |

## Usage Examples

```
List open tabs:
→ list_pages()

Select a tab:
→ select_page(pageId: 0)

Get page content:
→ take_snapshot()

Check console errors:
→ list_console_messages(types: ["error"])

Check network requests:
→ list_network_requests()
```

## Performance Analysis

```
Start trace:
→ performance_start_trace(reload: true, autoStop: true)

Analyze insights:
→ performance_analyze_insight(insightSetId: "...", insightName: "LCPBreakdown")
```

## Repository

https://github.com/nicholasoxford/chrome-devtools-mcp
