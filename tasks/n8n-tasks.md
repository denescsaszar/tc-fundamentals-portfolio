# 📄 n8n-tasks.md
## Coding Tasks — n8n Workflow Automation

> **How to use:**
> 1. Start n8n locally: `npx n8n` → open http://localhost:5678
> 2. Create a new workflow for each task
> 3. Export finished workflow: Menu → Download → save as `workflow.json` in the task folder
> 4. Commit the JSON file as your portfolio artifact
> 5. Each task is completable in under 45 minutes

---

## Task N1 — Beginner: Webhook Logger

**Folder:** `n8n/projects/beginner-webhook/`
**Time:** ~20 minutes
**Produces:** `workflow.json` + `README.md` + `test-request.sh`

### What you'll build
A webhook endpoint that receives a POST request, transforms the data, and returns a structured JSON response.

### Step 1 — Build the workflow

Add these nodes in order:

**Node 1: Webhook**
Type: Webhook
HTTP Method: POST
Path: logger
Response: Using Respond to Webhook node

text

**Node 2: Set**
Name: Transform Data
Add fields:
- received_at → {{ $now.toISO() }}
- event_type → {{ $json.event }}
- payload_keys → {{ Object.keys($json).join(", ") }}
- status → "processed"

text

**Node 3: Respond to Webhook**
Respond With: JSON
Response Body:
{
"success": true,
"message": "Event logged",
"data": {
"event": "{{ $json.event_type }}",
"receivedAt": "{{ $json.received_at }}",
"keys": "{{ $json.payload_keys }}"
} }

text

### Step 2 — Test it

Create `test-request.sh`:

```bash
#!/bin/bash
# Run this while n8n is running in test mode
curl -X POST http://localhost:5678/webhook-test/logger \
  -H "Content-Type: application/json" \
  -d '{
    "event": "user.signup",
    "userId": "U123",
    "email": "alice@example.com",
    "plan": "pro"
  }'
```

Run it:
```bash
chmod +x test-request.sh
./test-request.sh
```

### Step 3 — Save and commit
- Activate the workflow in n8n
- Export: ⋮ Menu → Download
- Save as `n8n/projects/beginner-webhook/workflow.json`

```bash
git add n8n/projects/beginner-webhook/
git commit -m "feat: N1 — n8n webhook logger with Set and Respond nodes"
git push origin main
```

---

## Task N2 — Intermediate: API Pipeline with Dual Output

**Folder:** `n8n/projects/intermediate-api-pipeline/`
**Time:** ~40 minutes
**Produces:** `workflow.json` + `README.md`

### What you'll build
A workflow that fetches books from the Open Library API, filters by availability, then outputs results as both JSON and XML using a Code node.

### Step 1 — Build the workflow

**Node 1: Manual Trigger**
(for testing — swap for Schedule in production)

**Node 2: HTTP Request**
Method: GET
URL: https://openlibrary.org/search.json?q=clean+code&limit=5

text

**Node 3: Code — Extract Books**
```javascript
const docs = $input.first().json.docs || [];

const books = docs.map(doc => ({
  json: {
    title: doc.title || "Unknown",
    author: (doc.author_name || ["Unknown"]),
    year: doc.first_publish_year || null,
    key: doc.key
  }
}));

return books;
```

**Node 4: IF — Filter Recent Books**
Condition: Number
Value 1: {{ $json.year }}
Operation: Larger Than
Value 2: 2000

text

**Node 5: Code — Generate XML (True branch)**
```javascript
const items = $input.all();

const xmlItems = items.map(item => {
  const b = item.json;
  const escape = s => String(s)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;");

  return `  <book>
    <title>${escape(b.title)}</title>
    <author>${escape(b.author)}</author>
    <year>${b.year}</year>
  </book>`;
}).join("\n");

const xml = `<?xml version="1.0" encoding="UTF-8"?>\n<books>\n${xmlItems}\n</books>`;

return [{
  json: {
    format: "xml",
    count: items.length,
    output: xml
  }
}];
```

**Node 6: Code — Format JSON (False branch)**
```javascript
const items = $input.all();

return [{
  json: {
    format: "json",
    count: items.length,
    books: items.map(i => i.json)
  }
}];
```

### Step 2 — Test and observe
- Run the workflow manually
- Check the IF node's true/false outputs
- Inspect the XML string in Node 5's output panel
- Copy the XML from the output and validate at xmlvalidation.com

### Step 3 — Save and commit
Export as `n8n/projects/intermediate-api-pipeline/workflow.json`

```bash
git add n8n/projects/intermediate-api-pipeline/
git commit -m "feat: N2 — n8n API pipeline with IF filter and dual JSON/XML output"
git push origin main
```

---

## Task N3 — Advanced: AI Agent with Structured Output

**Folder:** `n8n/projects/advanced-ai-agent/`
**Time:** ~45 minutes
**Produces:** `workflow.json` + `system-prompt-v1.md` + `README.md`

### What you'll build
An AI Agent workflow that takes a book title as input, researches it, and returns **enforced structured JSON output** — validated by a Code node. The system prompt is version-controlled in Git.

### Step 1 — Create `system-prompt-v1.md`

```markdown
# System Prompt — Book Analyst Agent
Version: 1.0.0
Updated: 2026-04-24

## Role
You are a book analysis assistant. Given a book title, return a structured JSON
analysis with no extra text, no markdown formatting, and no explanation.

## Output format (strict JSON only)
{
  "title": "exact book title",
  "author": "author full name",
  "year": 1900,
  "genre": "one word",
  "summary": "max 2 sentences",
  "difficulty": "beginner|intermediate|advanced",
  "recommendedFor": ["role1", "role2"]
}

## Rules
- Never add text before or after the JSON
- Never wrap the JSON in markdown code blocks
- If you don't know a field, use null
- difficulty must be exactly one of: beginner, intermediate, advanced
```

Commit the prompt first:
```bash
git add n8n/projects/advanced-ai-agent/system-prompt-v1.md
git commit -m "docs: add AI agent system prompt v1.0.0"
```

### Step 2 — Build the workflow

**Node 1: Manual Trigger**

**Node 2: Set — Input**
book_title → "Clean Code"

text

**Node 3: AI Agent**
Model: GPT-4o (or Claude 3.5 Sonnet)
System prompt: [paste full contents of system-prompt-v1.md]
User message: Analyze this book: {{ $json.book_title }}

text

**Node 4: Code — Validate Output**
```javascript
const raw = $input.first().json.output || $input.first().json.text || "";

let parsed;
try {
  // Strip accidental markdown code fences if present
  const cleaned = raw.replace(/```json\n?/g, "").replace(/```\n?/g, "").trim();
  parsed = JSON.parse(cleaned);
} catch (err) {
  throw new Error(`AI returned invalid JSON: ${raw.substring(0, 200)}`);
}

// Validate required fields
const required = ["title", "author", "year", "genre", "summary", "difficulty", "recommendedFor"];
const missing = required.filter(f => !(f in parsed));
if (missing.length > 0) {
  throw new Error(`Missing required fields: ${missing.join(", ")}`);
}

// Validate difficulty enum
const validDifficulty = ["beginner", "intermediate", "advanced"];
if (!validDifficulty.includes(parsed.difficulty)) {
  throw new Error(`Invalid difficulty: ${parsed.difficulty}`);
}

return [{ json: { valid: true, data: parsed } }];
```

**Node 5: Set — Final Output**
title → {{ $json.data.title }}
author → {{ $json.data.author }}
difficulty → {{ $json.data.difficulty }}
summary → {{ $json.data.summary }}
validated_at → {{ $now.toISO() }}

text

### Step 3 — Test it
- Run manually with "Clean Code"
- Try "The Lord of the Rings" — watch the difficulty field
- Break it intentionally: change the system prompt to return random text → see the Code node error

### Step 4 — Save and commit
Export as `n8n/projects/advanced-ai-agent/workflow.json`

```bash
git add n8n/projects/advanced-ai-agent/
git commit -m "feat: N3 — n8n AI agent with structured JSON output validation"
git push origin main
```

---

## n8n Task Completion Tracker

| Task | What you build | Skills | Status |
|------|---------------|--------|--------|
| N1 — Webhook Logger | POST → transform → response | Webhook, Set, Respond | ☐ |
| N2 — API Pipeline | API → filter → JSON + XML | HTTP Request, IF, Code | ☐ |
| N3 — AI Agent | LLM → validated JSON output | AI Agent, Code, prompt engineering | ☐ |

---

## Useful n8n Tips

```bash
# Start n8n locally (no install needed)
npx n8n

# Start with a specific port
N8N_PORT=5679 npx n8n

# Access the UI
open http://localhost:5678

# Export a workflow via CLI (if n8n is installed globally)
n8n export:workflow --id=1 --output=workflow.json
```
