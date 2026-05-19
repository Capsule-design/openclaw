---
name: stagehand-web
description: Interact with any website using natural language — act, extract content, or observe DOM structure. Powered by Stagehand + Playwright Chromium. Use this instead of peekaboo+curl for web automation tasks.
metadata:
  {
    "openclaw":
      { "emoji": "🌐", "os": ["darwin"], "requires": { "bins": ["node"] }, "install": [] },
  }
---

# stagehand-web

Stagehand-web wraps Stagehand v3 + Playwright Chromium to let you interact with web pages using plain English. It runs headless Chromium locally — no browser window appears.

**CLI:**

```
node ~/agents/stagehand-web/index.js <command> "<instruction>" --url <url> [--timeout <ms>] [--headless false]
```

Output is always a single JSON line: `{ "success": bool, "result": string, "duration_ms": number, "url": string }`

## Commands

### act — perform an action on the page

```bash
node ~/agents/stagehand-web/index.js act "click the Sign In button" --url https://example.com
node ~/agents/stagehand-web/index.js act "fill the email field with test@example.com" --url https://example.com
node ~/agents/stagehand-web/index.js act "select 'Monthly' from the billing dropdown" --url https://example.com
```

Returns `result: "action completed"` on success.

### extract — extract text or data from the page

```bash
node ~/agents/stagehand-web/index.js extract "get the main heading" --url https://example.com
node ~/agents/stagehand-web/index.js extract "what is the price shown?" --url https://example.com/pricing
node ~/agents/stagehand-web/index.js extract "list all navigation links" --url https://example.com
```

Returns the page accessibility tree as `pageText` — a hierarchical text representation of all visible elements. Parse the result to find the information requested.

### observe — inspect the DOM structure

```bash
node ~/agents/stagehand-web/index.js observe "what forms are on this page" --url https://example.com
node ~/agents/stagehand-web/index.js observe "list all buttons" --url https://example.com
```

Returns a JSON tree of DOM elements (tag, text, href, role) up to 4 levels deep. Use this to understand page structure before acting.

### navigate — go to a URL or follow a link

```bash
node ~/agents/stagehand-web/index.js navigate "go to the pricing page" --url https://example.com
node ~/agents/stagehand-web/index.js navigate "click the About link in the nav" --url https://example.com
```

Returns the URL of the page after navigation.

## Typical workflow

1. `observe` to understand the page structure
2. `act` to interact with elements
3. `extract` to pull data from the result

```bash
# 1. See what's on the page
node ~/agents/stagehand-web/index.js observe "what input fields exist" --url https://example.com/login

# 2. Fill and submit
node ~/agents/stagehand-web/index.js act "fill the email field with user@example.com" --url https://example.com/login
node ~/agents/stagehand-web/index.js act "click the Login button" --url https://example.com/login

# 3. Extract the result
node ~/agents/stagehand-web/index.js extract "was login successful?" --url https://example.com/dashboard
```

## Notes

- Always dispatch via `dispatch_to_vulcan` — don't run stagehand-web in the foreground.
- `--timeout` defaults to 60000ms (60s). For slow pages, increase to 90000.
- `--headless false` opens a visible Chrome window (useful for debugging; avoid in normal use).
- The CLI uses `claude-haiku-4-5-20251001` for `act` (cheapest capable model).
- `observe` and `extract` don't call the LLM — they return the raw DOM/AX tree.
- Requires `ANTHROPIC_API_KEY` in `~/mac-studio/env.sh` (already set on the Mac Studio).
