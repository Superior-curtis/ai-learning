# ListMcpResourcesTool: Listing MCP Resources

> 📅 Mon Jul 27 2026 08:00:00 GMT+0800 (Taiwan Standard Time) · Tooling
> Understand how ListMcpResourcesTool gives the model MCP resource discovery capabilities.

---

## ListMcpResourcesTool: Listing MCP Resources

ListMcpResourcesTool's job is much like **directory browsing** in the remote world. If the model is already connected to an MCP server but doesn't know which resources are exposed over there, the first step should be to list them with this tool.

### One-Sentence Summary

> ListMcpResourcesTool solves the **resource discovery problem** — without it, the model can only guess at URIs.
