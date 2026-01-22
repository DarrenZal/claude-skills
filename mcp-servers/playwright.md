# Playwright MCP Server

Browser automation for web testing and interaction.

## Installation

No installation needed - uses npx.

```bash
claude mcp add -s user playwright -- npx @playwright/mcp@latest
```

## Tools

| Tool | Description |
|------|-------------|
| `browser_navigate` | Navigate to URL |
| `browser_snapshot` | Get accessibility snapshot (better than screenshot) |
| `browser_click` | Click element |
| `browser_type` | Type into element |
| `browser_fill_form` | Fill multiple form fields |
| `browser_take_screenshot` | Take screenshot |
| `browser_evaluate` | Run JavaScript |
| `browser_wait_for` | Wait for text/element |
| `browser_tabs` | Manage browser tabs |

## Usage Examples

```
Navigate to page:
→ browser_navigate(url: "https://example.com")

Get page structure:
→ browser_snapshot()

Click button:
→ browser_click(element: "Submit button", ref: "button[0]")

Fill form:
→ browser_fill_form(fields: [
    {name: "Email", type: "textbox", ref: "input[0]", value: "test@example.com"}
  ])
```

## Best Practices

1. **Prefer `browser_snapshot` over `browser_take_screenshot`** - snapshots are actionable, screenshots are not
2. **Use ref values from snapshot** - the snapshot returns element references to use in actions
3. **Wait for content** - use `browser_wait_for` to ensure page is loaded

## Repository

https://github.com/playwright/mcp
