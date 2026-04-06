---
name: payweave
description: Discovers and invokes payment-enabled API resources on PayWeave. Use when you need to find MPP-compatible APIs, serverless functions, or file downloads with per-call USD pricing. Supports REST discovery, natural-language search via MCP, and fetching individual resource details with input schemas and example outputs.
metadata:
  author: payweave
  version: "1.0"
---

# PayWeave

[PayWeave](https://payweave.app/) is a payment-enabled discovery platform. Resources (gateways, functions, file collections) are monetised per-call via the Machine Payment Protocol (MPP). Use the Discovery API or MCP tools below to find and invoke them.

## Discovery API

The Skills API is the primary REST interface for discovering all payment-enabled resources on PayWeave.

### Endpoints

| Scope | URL |
|-------|-----|
| Global (all workspaces) | `GET https://api.payweave.app/api/discovery/skills` |
| Workspace-scoped | `GET https://api.payweave.app/api/discovery/:workspaceId/skills` |

`workspaceId` must be a valid UUID. Returns `404` if the workspace does not exist.

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `type` | `gateway` \| `function` \| `file` | — | Filter by skill type. Omit to return all types. |
| `search` | string | — | Case-insensitive search against name and description fields. |
| `limit` | integer (1–200) | `50` | Maximum number of skills to return per page. |
| `offset` | integer (≥ 0) | `0` | Number of skills to skip (for pagination). |

### Response Shape

#### `SkillSet` (top-level)

```json
{
  "platform": "PayWeave",
  "version": "0.1.0",
  "scope": "global",
  "workspaceId": "10000001-5eed-4000-a000-000000000001",
  "workspaceName": "Acme AI Labs",
  "skills": [],
  "pagination": {
    "limit": 50,
    "offset": 0,
    "total": 142
  },
  "generatedAt": "2026-04-02T10:30:00.000Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `platform` | string | Always `"PayWeave"` |
| `version` | string | API version |
| `scope` | `"global"` \| `"workspace"` | Whether results are global or workspace-scoped |
| `workspaceId` | string (UUID) | Present only when `scope` is `"workspace"` |
| `workspaceName` | string | Present only when `scope` is `"workspace"` |
| `skills` | `Skill[]` | Array of skill objects (see below) |
| `pagination.limit` | integer | Effective page size |
| `pagination.offset` | integer | Current offset |
| `pagination.total` | integer | Total matching skills before pagination |
| `generatedAt` | ISO 8601 | Timestamp of response generation |

#### `Skill`

```json
{
  "id": "gw_abc123",
  "type": "gateway",
  "name": "Sentiment Analysis API",
  "description": "Analyse text sentiment using a fine-tuned model.",
  "baseUrl": "https://api.payweave.app/gw/gw_abc123",
  "openApiUrl": "https://api.payweave.app/gw/gw_abc123/openapi.json",
  "endpoints": []
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique public ID (e.g. `gw_abc123`, `fn_xyz789`, `fc_def456`) |
| `type` | `"gateway"` \| `"function"` \| `"file"` | Skill type |
| `name` | string | Human-readable name |
| `description` | string \| null | Description of what the skill does |
| `baseUrl` | string | Base URL for all endpoints in this skill |
| `openApiUrl` | string | URL to the skill's OpenAPI 3.1 spec |
| `endpoints` | `SkillEndpoint[]` | Individual callable endpoints |

#### `SkillEndpoint`

```json
{
  "path": "/v1/analyse",
  "name": "Analyse Text",
  "description": "Returns a sentiment score between -1.0 and 1.0.",
  "methods": "POST",
  "priceUsd": "0.005",
  "inputSchema": { "text": { "type": "string" } },
  "exampleInput": { "text": "PayWeave is great!" },
  "exampleOutput": { "score": 0.92, "label": "positive" }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `path` | string | URL path appended to `baseUrl` to form the full endpoint URL |
| `name` | string | Human-readable endpoint name |
| `description` | string \| null | What the endpoint does |
| `methods` | string | Allowed HTTP methods (e.g. `"GET"`, `"POST"`) |
| `priceUsd` | string | Price per call in USD (decimal string, e.g. `"0.005"`) |
| `inputSchema` | object | JSON schema for the request body (optional) |
| `exampleInput` | object | Example request payload (optional) |
| `exampleOutput` | any | Example response value (optional) |

### Code Examples

#### Fetch all gateway skills

```javascript
const res = await fetch(
  'https://api.payweave.app/api/discovery/skills?type=gateway&limit=20'
);
const data = await res.json();

for (const skill of data.skills) {
  console.log(`${skill.name} — ${skill.openApiUrl}`);
  for (const ep of skill.endpoints) {
    console.log(`  ${ep.methods} ${ep.path}  $${ep.priceUsd}/call`);
  }
}
```

#### Search by keyword

```javascript
const res = await fetch(
  'https://api.payweave.app/api/discovery/skills?search=image+classification'
);
const { skills } = await res.json();
console.log(`Found ${skills.length} matching skills`);
```

#### Fetch all pages

```javascript
async function fetchAllSkills(baseUrl = 'https://api.payweave.app') {
  const PAGE = 50;
  let offset = 0;
  const all = [];

  while (true) {
    const res = await fetch(
      `${baseUrl}/api/discovery/skills?limit=${PAGE}&offset=${offset}`
    );
    const { skills, pagination } = await res.json();
    all.push(...skills);
    offset += PAGE;
    if (offset >= pagination.total) break;
  }

  return all;
}
```

#### Workspace-scoped

```javascript
const workspaceId = '10000001-5eed-4000-a000-000000000001';
const res = await fetch(
  `https://api.payweave.app/api/discovery/${workspaceId}/skills`
);
const { skills } = await res.json();
```

### Caching

All Discovery API responses are cached with a **5-minute TTL**. Changes to resource configuration (name, description, discoverable flag, pricing) may take up to 5 minutes to appear.

### Visibility Rules

A resource appears in discovery results only when:
- `discoverable` is set to `true`
- `status` is `active`

Paused or archived resources are excluded even if `discoverable` is enabled.

---

## MCP Server

For agents that support MCP (Model Context Protocol), PayWeave exposes the same discovery capabilities as MCP tools.

### MCP Server URLs

| Scope | URL |
|-------|-----|
| Global (all workspaces) | `https://api.payweave.app/mcp` |
| Workspace-scoped | `https://api.payweave.app/mcp/{workspaceId}` |

### Quick Setup

```bash
# Claude Code
claude mcp add payweave --transport http https://api.payweave.app/mcp

# Workspace-scoped
claude mcp add payweave --transport http https://api.payweave.app/mcp/<workspaceId>
```

### MCP Tools

#### `list_resources`

List all discoverable MPP-enabled resources.

**Use when:** the user wants to browse or enumerate available APIs, functions, or file downloads.

**Parameters:**
- `type` *(optional)* — `"http"` or `"mcp"` to filter by resource type
- `limit` *(optional)* — maximum results (default 100, max 500)

#### `find_resources`

Search resources by natural language query. Results are ranked by relevance using full-text and fuzzy matching.

**Use when:** the user describes what they need (e.g. "weather API", "image classifier", "translate text").

**Parameters:**
- `query` — natural language search term
- `type` *(optional)* — `"http"` or `"mcp"` to filter

#### `get_resource`

Fetch full details for a single resource by its exact URL.

**Use when:** you have a specific resource URL from `list_resources` or `find_resources` and need its full schema, example input, or example output before making a paid call.

**Parameters:**
- `resource_url` — the exact endpoint URL (e.g. `https://api.payweave.app/gw/gw_abc123/v1/analyse`)

### MCP Response Fields

Each resource item includes:

| Field | Description |
|-------|-------------|
| `url` | Full endpoint URL to call |
| `description` | What the endpoint does |
| `type` | `"http"` or `"mcp"` |
| `pricing.amount` | Cost in USD per call |
| `pricing.currency` | Always `"USD"` |
| `discoveryInfo.httpMethod` | HTTP method (`GET`, `POST`, etc.) |
| `discoveryInfo.bodySchema` | JSON input schema (when available) |
| `discoveryInfo.exampleOutput` | Sample response (when available) |

---

## Recommended Workflow

1. **Discover** — call the Discovery API or MCP `find_resources` with the user's intent
2. **Inspect** — fetch full details for the best match to review its schema and example
3. **Inform the user** — show the endpoint URL, HTTP method, price, and input schema
4. **Invoke** — the user calls the endpoint directly with MPP payment credentials

## MPP Payment Notes

- All endpoints require MPP (Machine Payment Protocol) payment credentials
- Pricing is settled on-chain via the Tempo network in pathUSD or USDC.e
- Append `?ref=WALLET_ADDRESS` to any payment URL to earn referral rewards (up to 10%)
- Failed requests (5xx errors) trigger an automatic on-chain refund to the payer
yer
