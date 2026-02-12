# OpenClaw WebMCP Skill

> First-mover integration: OpenClaw + Chrome WebMCP (W3C Standard)

Lets OpenClaw agents discover and invoke [WebMCP](https://github.com/webmachinelearning/webmcp) tools exposed by websites via Chrome's `navigator.modelContext` API.

**Instead of clicking buttons and scraping DOM, agents call structured tools directly. JSON in, JSON out.**

## What is WebMCP?

[WebMCP](https://github.com/webmachinelearning/webmcp) is a proposed W3C web standard (Chrome 146+) that enables websites to register callable tools with the browser. AI agents can discover and invoke these tools without UI automation.

```js
// Website registers a tool
navigator.modelContext.registerTool({
  name: "calculate_roi",
  description: "Calculate ROI for switching to our platform",
  inputSchema: { type: "object", properties: { employeeCount: { type: "number" } } },
  execute: async ({ employeeCount }) => {
    return { monthlySavings: employeeCount * 43.2, annualROI: employeeCount * 518.4 };
  }
});
```

```
// OpenClaw agent invokes it
browser action=act request={
  "kind": "evaluate",
  "fn": "async () => await window.__WEBMCP_TOOLS__.calculate_roi.execute({employeeCount: 50})"
}
// → { monthlySavings: 2160, annualROI: 25920 }
```

## Requirements

- Chrome Canary 146+ with **"WebMCP for testing"** flag enabled (`chrome://flags`)
- [OpenClaw](https://github.com/openclaw/openclaw) with browser control
- Target site must implement WebMCP tool registration

## Installation

Copy `SKILL.md` to your OpenClaw skills directory:

```bash
cp SKILL.md ~/clawd/skills/webmcp/SKILL.md
```

## Usage

### 1. Check WebMCP Support
```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => ({ supported: !!navigator.modelContext })"
}
```

### 2. Discover Tools
```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => window.__WEBMCP_TOOLS__ ? Object.keys(window.__WEBMCP_TOOLS__) : 'No tools exposed'"
}
```

### 3. Invoke a Tool
```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "async () => await window.__WEBMCP_TOOLS__.get_pricing.execute({tier: 'all', billing: 'annual'})"
}
```

## For Website Developers

To make your site WebMCP-compatible and agent-accessible:

1. **Register tools** with `navigator.modelContext.registerTool()` or `provideContext()`
2. **Expose on window** via `window.__WEBMCP_TOOLS__` for agent discovery
3. **Follow the W3C spec**: each tool needs `name`, `description`, `inputSchema`, `execute`

See the [W3C WebMCP Explainer](https://github.com/webmachinelearning/webmcp) for the full specification.

## Example: ProForge ERP

[getproforge.com](https://getproforge.com) implements 4 WebMCP tools:
- `request_demo` — Submit demo request with lead info
- `get_pricing` — Get tier pricing (Foundation/Professional/Enterprise)
- `check_compliance` — Check Davis-Bacon, prevailing wage, certified payroll features
- `calculate_roi` — Calculate ROI based on company size and current processes

## How It Works

```
Before WebMCP:
Agent → loads page → parses DOM → finds button → clicks → reads HTML response

After WebMCP:  
Agent → calls function → gets structured JSON
```

Same difference as scraping a website vs calling its API.

## Links

- [W3C WebMCP Spec](https://github.com/webmachinelearning/webmcp)
- [MCP-B Polyfill](https://github.com/WebMCP-org)
- [OpenClaw](https://github.com/openclaw/openclaw)
- [Chrome Canary](https://www.google.com/chrome/canary/)

## License

MIT
