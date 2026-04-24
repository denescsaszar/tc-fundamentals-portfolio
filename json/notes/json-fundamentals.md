# 📄 json-fundamentals.md

## JSON: From Syntax to Real-World Use

> **Goal:** Understand JSON well enough to read, write, and validate it confidently — and know exactly where it lives in the real world.

---

## 1. What is JSON?

JSON (JavaScript Object Notation) is a **lightweight text format for storing and exchanging data**. It's language-agnostic: Python, Java, Rust, and Go all read it just fine. It won the format wars because it's human-readable, writable by hand, and maps directly to data structures that every language understands.

> **Why this matters:** JSON is _the_ lingua franca of APIs. Every time your app fetches data from a server — weather, stock prices, user profiles — that data almost certainly arrives as JSON.

> ⚠️ **Common mistake:** Confusing JSON with a JavaScript object. They look similar, but JSON requires double-quoted keys. `{name: "Alice"}` is a JS object. `{"name": "Alice"}` is valid JSON.

---

## 2. Data Types

JSON has exactly **six value types** — no more, no less.

| Type    | Example            | Notes                                |
| ------- | ------------------ | ------------------------------------ |
| String  | `"hello"`          | Always double quotes — never single  |
| Number  | `42`, `3.14`, `-7` | No distinction between int and float |
| Boolean | `true`, `false`    | Lowercase only                       |
| Null    | `null`             | Represents "no value"                |
| Object  | `{"key": "value"}` | Unordered key-value pairs            |
| Array   | `[1, 2, 3]`        | Ordered list of any types            |

```json
{
  "name": "Alice",
  "age": 30,
  "score": 9.8,
  "active": true,
  "nickname": null,
  "tags": ["developer", "learner"],
  "address": {
    "city": "Berlin",
    "zip": "10115"
  }
}
```

> **Why this matters:** Knowing the exact 6 types prevents bugs. When an API sends `"42"` instead of `42`, your code breaks — those are different types.

> ⚠️ **Common mistake:** Using `undefined`, comments (`// like this`), or trailing commas (`[1, 2, 3,]`). All three are **illegal JSON** and will cause parse errors.

---

## 3. Syntax Rules

1. **Keys must be strings** wrapped in double quotes
2. **String values** must use double quotes
3. **No trailing commas** — the last item has no comma after it
4. **No comments** — JSON has no comment syntax
5. **Root element** can be any valid JSON value

```json
{
  "valid": "yes",
  "number": 100,
  "list":,[1][2][3]
  "nested": {
    "works": true
  }
}
```

> ⚠️ **Common mistake:** Adding a trailing comma after the last key-value pair. This is the #1 JSON syntax error. Use [jsonlint.com](https://jsonlint.com) to validate instantly.

---

## 4. Nesting

Objects and arrays can nest inside each other to any depth.

```json
{
  "company": "Acme Corp",
  "employees": [
    {
      "id": 1,
      "name": "Alice",
      "skills": ["JSON", "Python"],
      "address": {
        "city": "Berlin",
        "country": "DE"
      }
    },
    {
      "id": 2,
      "name": "Bob",
      "skills": ["XML", "Java"],
      "address": {
        "city": "Munich",
        "country": "DE"
      }
    }
  ]
}
```

> **Why this matters:** Real API responses are deeply nested. You need to navigate several levels to extract the data you actually want.

> ⚠️ **Common mistake:** Going 6–7 levels deep in your own designs. Deep nesting is a code smell — it means your data structure needs rethinking.

---

## 5. Parsing JSON

**Parsing** means converting a JSON string into a data structure your code can work with.

### In JavaScript (Node.js)

```javascript
// String → Object
const jsonString = '{"name": "Alice", "age": 30}';
const obj = JSON.parse(jsonString);
console.log(obj.name); // "Alice"

// Object → String
const user = { name: "Bob", city: "Berlin" };
const output = JSON.stringify(user, null, 2);
console.log(output);
```

### Always parse safely

```javascript
try {
  const data = JSON.parse(maybeInvalidString);
} catch (err) {
  console.error("Invalid JSON:", err.message);
}
```

### Reading a JSON file in Node.js

```javascript
const fs = require("fs");
const raw = fs.readFileSync("./config.json", "utf8");
const config = JSON.parse(raw);
console.log(config.server.port);
```

> **Why this matters:** Parsing is the entry point to every API integration. If you can't parse JSON, you can't work with web data.

> ⚠️ **Common mistake:** Forgetting `try/catch` around `JSON.parse()`. Malformed JSON from an API will crash your program without it.

---

## 6. JSON Schema

**JSON Schema** is a vocabulary for describing and validating the shape of JSON data.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "User",
  "type": "object",
  "required": ["name", "age"],
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1
    },
    "age": {
      "type": "integer",
      "minimum": 0,
      "maximum": 150
    },
    "email": {
      "type": "string",
      "format": "email"
    }
  }
}
```

> **Why this matters:** Schema validation catches bad data before it enters your system. APIs without validation are ticking time bombs.

> ⚠️ **Common mistake:** Writing schemas after the fact. Define the schema first — it forces you to think clearly about your data structure before writing any code.

---

## 7. Real-World Use Cases

### Config file (`config.json`)

```json
{
  "server": {
    "host": "localhost",
    "port": 3000
  },
  "database": {
    "url": "postgres://localhost/mydb",
    "poolSize": 10
  },
  "features": {
    "darkMode": true,
    "betaFeatures": false
  }
}
```

### REST API response

```json
{
  "status": "ok",
  "data": {
    "users": [
      { "id": 1, "name": "Alice", "role": "admin" },
      { "id": 2, "name": "Bob", "role": "viewer" }
    ],
    "total": 2,
    "page": 1
  }
}
```

### `package.json` (Node.js project manifest)

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

---

## Self-Check

Before moving on, you should be able to:

- [ ] Write valid JSON from memory without looking it up
- [ ] Explain the difference between JSON and a JavaScript object literal
- [ ] Parse a JSON string in at least one language
- [ ] Describe what JSON Schema is and why you'd use it
- [ ] Name 3 real places where JSON is used in software
