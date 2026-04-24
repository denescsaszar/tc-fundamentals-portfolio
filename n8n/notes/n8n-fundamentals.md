# 📄 n8n-fundamentals.md

## n8n: Visual Automation From First Workflow to AI Agents

> **Goal:** Understand n8n well enough to build real workflows, explain your choices in an interview, and integrate it with JSON and XML projects.

---

## 1. What is n8n?

n8n (pronounced "nodemation") is an **open-source, self-hostable workflow automation platform**. You connect nodes on a visual canvas to automate tasks — without writing a full application. It's the developer-friendly alternative to Zapier or Make, because it lets you drop into real JavaScript/Python when the visual nodes aren't enough.

> **Why this matters:** Automation is now a core dev skill. n8n is used to glue together APIs, process data, trigger AI agents, and replace entire backend scripts — faster than hand-rolling everything.

> ⚠️ **Common mistake:** Thinking n8n is just for "no-code people." The Code node, HTTP Request node, and sub-workflow system make it a serious tool for developers. The visual canvas is a feature, not a limitation.

---

## 2. The Four Core Concepts

### Workflow

A **workflow** is a saved, executable automation — a canvas of connected nodes. It can be triggered manually, on a schedule, or by an external event. Each run is logged as an **execution**.

### Trigger Node

The **trigger** is always the first node. Nothing runs without it.

| Trigger Type         | When to use                                                      |
| -------------------- | ---------------------------------------------------------------- |
| **Webhook**          | An external system sends an HTTP request to n8n                  |
| **Schedule**         | Run at a fixed time/interval (like a cron job)                   |
| **App Event**        | Something happens in a connected app (new Gmail, new Sheets row) |
| **Manual**           | You click "Execute Workflow" — for testing only                  |
| **Workflow Trigger** | Another workflow calls this one (sub-workflow pattern)           |

### Action Node

Action nodes do the actual work. Most important ones:

- **HTTP Request** — call any REST API (the single most important node)
- **Code** — run JavaScript or Python inline
- **Set** — add, rename, or remove fields on your data
- **Read/Write File** — file system access (self-hosted only)
- **Send Email / Slack / Telegram** — notifications
- **Google Sheets / Airtable** — read/write structured data

### Logic Node

| Node                | What it does                                 |
| ------------------- | -------------------------------------------- |
| **IF**              | Split into two paths based on a condition    |
| **Switch**          | Split into multiple paths (like switch/case) |
| **Merge**           | Combine data from multiple branches          |
| **Loop Over Items** | Iterate over an array of items               |
| **Wait**            | Pause execution for a set time               |
| **Stop and Error**  | Halt and throw a custom error                |

> **Why this matters:** Knowing which node to reach for — and why — is what separates someone who's used n8n from someone who understands it.

> ⚠️ **Common mistake:** Using the Code node for everything. Reach for built-in nodes first. Code is the escape hatch, not the default.

---

## 3. The Data Model

This is the concept most people skip — and then wonder why their workflows break.

n8n passes data between nodes as an **array of items**. Each item is a JSON object with this structure:

```json
[
  {
    "json": {
      "name": "Alice",
      "age": 30,
      "city": "Berlin"
    }
  },
  {
    "json": {
      "name": "Bob",
      "age": 25,
      "city": "Munich"
    }
  }
]
```

Every node **receives** this array, **processes** each item, and **outputs** a new array.

### Accessing data in expressions

{{ $json.name }} current item's name field
{{ $json.address.city }} nested field
{{ $node["HTTP Request"].json.id }} data from a specific node
{{ $now.toISO() }} current timestamp
{{ $runIndex }} current loop iteration index

text

### In the Code node (JavaScript)

```javascript
const results = [];

for (const item of $input.all()) {
  const name = item.json.name;
  const age = item.json.age;

  results.push({
    json: {
      greeting: `Hello, ${name}!`,
      isAdult: age >= 18,
      processedAt: new Date().toISOString(),
    },
  });
}

return results;
```

> **Why this matters:** Every n8n bug ultimately comes down to misunderstanding what data is flowing between nodes. Learn this model first and debug 10x faster.

> ⚠️ **Common mistake:** Forgetting to wrap Code node output in `{ json: ... }`. If you return `{ name: "Alice" }` instead of `{ json: { name: "Alice" } }`, the next node sees nothing.

---

## 4. Triggers In Depth

### Webhook Trigger

POST https://your-n8n.com/webhook/your-unique-path
Content-Type: application/json

{ "event": "order.placed", "orderId": "ORD-001", "total": 49.99 }

text

Two modes:

- **Test mode** — activate manually, receives one request, stops
- **Production mode** — runs permanently, processes every request

### Schedule Trigger

Every day at 09:00 → fetch data → process → send report
Every 5 minutes → check API → if new data → notify Slack
First day of month → generate invoice XML → send by email

text

> ⚠️ **Common mistake:** Testing with the Schedule trigger. Always use a Manual trigger during development — waiting for the schedule to fire wastes time.

---

## 5. Error Handling

### Node-level: Continue on Error

In any node's settings → **"On Error"** → set to `Continue`. The workflow keeps running even if that node fails.

### Workflow-level: Error Workflow

In Workflow Settings → set an **Error Workflow**. If the main workflow crashes, n8n automatically triggers a second workflow with the error details.

```javascript
// In your Error Workflow, error data looks like:
// $json.execution.id       which execution failed
// $json.error.message      what went wrong
// $json.workflow.name      which workflow crashed
```

> **Why this matters:** A workflow that silently fails is worse than one that never ran. Error workflows are how you sleep at night.

> ⚠️ **Common mistake:** Building workflows without error handling and only discovering failures when a customer complains. Wire up an Error Workflow before going to production.

---

## 6. The HTTP Request Node

The most important node in n8n — calls any REST API.

### Basic GET

Method: GET
URL: https://api.open-meteo.com/v1/forecast?latitude=52.52&longitude=13.41&current_weather=true

text

Output arrives as `$json.current_weather.temperature`.

### POST with auth header

Method: POST
URL: https://api.example.com/books
Authentication: Header Auth
Name: Authorization
Value: Bearer {{ $env.API_TOKEN }}

Body (JSON):
{
"title": "{{ $json.title }}",
"author": "{{ $json.author }}"
}

text

> **Why this matters:** 90% of real integrations use this node because most APIs don't have a dedicated n8n node.

> ⚠️ **Common mistake:** Hardcoding API keys in the URL or body. Always use **Credentials** stored securely in n8n, or `$env.VARIABLE_NAME`.

---

## 7. Sub-workflows

A **sub-workflow** is a workflow called by another workflow — the n8n equivalent of a function.
Main Workflow
└── Trigger: New order webhook
└── Execute Workflow: "Validate Order Data"
└── Execute Workflow: "Generate Invoice XML"
└── Send confirmation email

text

> **Why this matters:** Without sub-workflows, complex automations become unmaintainable. Sub-workflows let you reuse logic across multiple automations.

---

## 8. AI Agent Node

n8n's AI Agent node connects an LLM to a set of **tools** — other n8n nodes the AI can call.
AI Agent Node
├── Model: GPT-4o / Claude 3.5 / Gemini
├── System prompt: "You are a data analyst..."
└── Tools:
├── HTTP Request (fetch data)
├── Code node (process results)
└── Google Sheets (write output)

text

The LLM decides which tools to call and in what order. The result is an autonomous agent built entirely inside n8n.

> **Why this matters:** AI Agent workflows are the hottest n8n use case right now. Any interview touching n8n in 2026 will ask about this.

> ⚠️ **Common mistake:** Giving the AI Agent too many tools. Start with 2–3 focused tools. More tools = more chance the LLM calls the wrong one.

---

## 9. n8n vs. Competitors

|                      | n8n                      | Zapier              | Make           |
| -------------------- | ------------------------ | ------------------- | -------------- |
| **Self-hostable**    | ✅ Yes                   | ❌ No               | ❌ No          |
| **Open source**      | ✅ Fair-code             | ❌ No               | ❌ No          |
| **Code node**        | ✅ JS + Python           | ❌ No               | Limited        |
| **Pricing**          | Free (self-hosted)       | Per task            | Per operation  |
| **AI Agent support** | ✅ Native                | Limited             | Limited        |
| **Best for**         | Developers, data privacy | Non-technical users | Mid-complexity |

---

## Self-Check

- [ ] Explain the four core building blocks of an n8n workflow
- [ ] Describe the n8n data model (items + json wrapper)
- [ ] Write a Code node that processes input items and returns transformed output
- [ ] Explain when to use Webhook vs. Schedule vs. Manual trigger
- [ ] Describe how error workflows work and why they matter
- [ ] Explain the difference between n8n and Zapier in one minute
- [ ] Describe what the AI Agent node does and what "tools" means
