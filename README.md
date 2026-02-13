<p align="center">
  <img src="https://img.shields.io/badge/W3C-WebMCP-005A9C?style=for-the-badge&logo=w3c&logoColor=white" alt="W3C WebMCP" />
  <img src="https://img.shields.io/badge/Chrome-146%2B-4285F4?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Chrome 146+" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="MIT License" />
  <img src="https://img.shields.io/badge/OpenClaw-Skill-FF6B35?style=for-the-badge" alt="OpenClaw Skill" />
</p>

# OpenClaw WebMCP Skill

**Give your AI agents structured tool access to any WebMCP-enabled website.**

No more DOM scraping. No more brittle selectors. Just JSON in, JSON out.

---

## What is WebMCP?

[WebMCP](https://github.com/nicholasgriffintn/webmcp) is a proposed **W3C web standard** (Chrome 146+) that lets websites register callable tools with the browser. AI agents can discover and invoke these tools without any UI automation.

```
  Traditional Agent                    WebMCP Agent
  ─────────────────                    ────────────
  Load page                            Call function
  Parse DOM                            Get structured JSON
  Find button                          Done.
  Click button
  Read HTML response
  Parse response
  Hope nothing changed
```

## How It Works

Websites register tools via `navigator.modelContext`. This skill lets OpenClaw agents discover and call those tools directly:

```js
// Website registers a tool
navigator.modelContext.registerTool({
  name: "get_pricing",
  description: "Get current pricing tiers",
  inputSchema: {
    type: "object",
    properties: {
      tier: { type: "string", enum: ["basic", "pro", "enterprise"] }
    }
  },
  execute: async ({ tier }) => {
    return { tier, monthly: 29, annual: 290, currency: "USD" };
  }
});
```

```
// OpenClaw agent invokes it — structured JSON response
browser action=act request={
  "kind": "evaluate",
  "fn": "async () => await window.__WEBMCP_TOOLS__.get_pricing.execute({tier: 'pro'})"
}
// -> { tier: "pro", monthly: 29, annual: 290, currency: "USD" }
```

## Requirements

| Requirement | Details |
|-------------|---------|
| **Browser** | Chrome Canary 146+ with `chrome://flags` → "WebMCP for testing" enabled |
| **Agent Framework** | [OpenClaw](https://github.com/anthropics/claude-code) with browser control |
| **Target Site** | Must implement WebMCP tool registration |

## Quick Start

### 1. Install the Skill

```bash
# Copy to your OpenClaw skills directory
cp skills/SKILL.md ~/.openclaw/skills/webmcp/SKILL.md
```

### 2. Check WebMCP Support

```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => ({ supported: !!navigator.modelContext })"
}
```

### 3. Discover Available Tools

```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "() => window.__WEBMCP_TOOLS__
    ? Object.keys(window.__WEBMCP_TOOLS__)
    : 'No tools exposed'"
}
```

### 4. Invoke a Tool

```
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "async () => await window.__WEBMCP_TOOLS__.get_pricing.execute({
    tier: 'pro',
    billing: 'annual'
  })"
}
```

## For Website Developers

Make your site agent-accessible in three steps:

```js
// 1. Register tools with the WebMCP API
navigator.modelContext.registerTool({
  name: "search_products",
  description: "Search product catalog by keyword",
  inputSchema: {
    type: "object",
    properties: { query: { type: "string" } },
    required: ["query"]
  },
  execute: async ({ query }) => {
    const results = await fetch(`/api/products?q=${query}`).then(r => r.json());
    return results;
  }
});

// 2. Expose on window for agent discovery
window.__WEBMCP_TOOLS__ = window.__WEBMCP_TOOLS__ || {};
window.__WEBMCP_TOOLS__.search_products = { execute: searchProducts };

// 3. That's it. Agents can now discover and call your tools.
```

Each tool needs: `name`, `description`, `inputSchema`, and `execute`.

See the [W3C WebMCP Explainer](https://github.com/nicholasgriffintn/webmcp) for the full specification.

## Example: Mock ERP Application

Here's what a WebMCP-enabled business app might expose:

| Tool | Description |
|------|-------------|
| `request_demo` | Submit a demo request with contact info |
| `get_pricing` | Retrieve pricing tiers and billing options |
| `check_compliance` | Query available compliance and regulatory features |
| `calculate_roi` | Estimate ROI based on team size and current workflow |

```
// Agent discovers and calls tools — no UI automation needed
browser action=act profile=chrome request={
  "kind": "evaluate",
  "fn": "async () => await window.__WEBMCP_TOOLS__.calculate_roi.execute({
    teamSize: 50
  })"
}
// -> { monthlySavings: 2500, annualROI: 30000 }
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `WebMCP not available` | Enable "WebMCP for testing" in `chrome://flags`, restart Canary |
| `No tools found` | Site hasn't registered tools, or they load on a specific route |
| Tool execution fails | Check the tool's `inputSchema` for required fields |
| Browser relay not connected | Click the OpenClaw toolbar icon on the target tab |

## Links

- [W3C WebMCP Spec](https://github.com/nicholasgriffintn/webmcp)
- [Chrome Canary](https://www.google.com/chrome/canary/)

## Contributing

Contributions are welcome! If you've found a WebMCP-enabled site or have improvements to the skill:

1. Fork the repo
2. Create your feature branch (`git checkout -b feat/my-improvement`)
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

## License

[MIT](LICENSE)
