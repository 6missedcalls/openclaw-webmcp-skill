# WebMCP Skill — Interact with WebMCP-Enabled Sites

## What This Does
Lets OpenClaw agents discover and invoke WebMCP tools exposed by websites via Chrome's `navigator.modelContext` API. Instead of clicking buttons and scraping DOM, the agent calls structured tools directly.

## Requirements
- Chrome Canary 146+ with "WebMCP for testing" flag enabled (`chrome://flags`)
- OpenClaw browser control connected (Chrome extension relay attached to the tab)
- Target site must implement WebMCP tool registration

## How It Works

### 1. Check WebMCP Support
```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => ({ supported: !!navigator.modelContext, methods: navigator.modelContext ? Object.getOwnPropertyNames(Object.getPrototypeOf(navigator.modelContext)) : [] })"
}
```

### 2. Discover Tools on a Page
The page registers tools via `registerTool()` or `provideContext()`. Since we're evaluating in the page context, we can access the internal registry if the site exposes it. For our sites (Proforge), tools are stored in a module-level Map:

```
browser action=navigate targetUrl="https://getproforge.com" profile=chrome
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => { if (!navigator.modelContext) return {error: 'WebMCP not available - enable flag in chrome://flags'}; /* Probe what methods exist */ const mc = navigator.modelContext; const info = {type: typeof mc}; for (const k of Object.getOwnPropertyNames(Object.getPrototypeOf(mc))) { info[k] = typeof mc[k]; } return info; }"
}
```

### 3. Invoke a Tool via Page's Registry
For sites where we control the code (Proforge, OpFlow), import the tool directly:

```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "async () => { /* Access the app's internal WebMCP registry */ const registry = window.__WEBMCP_TOOLS__; if (!registry) return {error: 'No tools exposed on window.__WEBMCP_TOOLS__'}; const tool = registry['TOOL_NAME']; if (!tool) return {error: 'Tool not found', available: Object.keys(registry)}; try { const result = await tool.execute(INPUT_OBJECT); return {success: true, result}; } catch(e) { return {error: e.message}; } }"
}
```

### 4. Quick Feature Detection
```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => !!navigator.modelContext"
}
```

## Workflow

1. **Attach tab** — User clicks OpenClaw Browser Relay icon on the target tab
2. **Navigate** — `browser action=navigate` to the WebMCP-enabled site
3. **Discover** — Run tool discovery to see what's available
4. **Invoke** — Call tools with structured JSON input
5. **Process** — Handle the structured response

## Known WebMCP Sites
| Site | Tools | Notes |
|------|-------|-------|
| getproforge.com | request_demo, get_pricing, check_compliance, calculate_roi | Our site |

## Notes
- WebMCP tools run in the page's JS context — they have access to the page's auth, state, and data
- Tools can be async (return Promises)
- The `agent` parameter in execute callbacks supports `requestUserInteraction()` for confirmation dialogs
- If `navigator.modelContext` is undefined, the flag isn't enabled or the browser doesn't support it
- Some sites may use `provideContext({tools: [...]})` instead of `registerTool()` — both expose tools the same way

## Alternative: Direct Evaluation (No WebMCP)
For sites without WebMCP, you can still evaluate JS directly:
```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => document.querySelector('form').submit()"
}
```
But WebMCP is preferred — structured input/output, no DOM scraping, no brittle selectors.

## Troubleshooting
- **"WebMCP not available"** → Enable "WebMCP for testing" in `chrome://flags`, restart Canary
- **"No tools found"** → Site hasn't registered tools yet, or they register on a specific page/route
- **Tool execution fails** → Check the tool's inputSchema for required fields
- **Browser relay not connected** → Click the OpenClaw toolbar icon on the tab
