# TC Fundamentals Portfolio

### JSON · XML · Git · n8n — by Dénes

> A structured learning portfolio demonstrating JSON, XML, Git, and n8n automation skills
> built through 12 hands-on projects. Every folder contains working code, not just notes.

---

## 🗂 What's in this repo

| Folder                       | Topic                          | Projects                                                |
| ---------------------------- | ------------------------------ | ------------------------------------------------------- |
| [`json/`](./json/)           | JSON syntax, parsing, schema   | Config files, API feed parser, JSON→XML converter       |
| [`xml/`](./xml/)             | XML, XSD validation, XSLT      | Library catalog, schema validation, HTML transformation |
| [`git/`](./git/)             | Git workflow, branching, CI    | Commit history, branch workflow, GitHub Actions         |
| [`n8n/`](./n8n/)             | Workflow automation, AI agents | Webhook logger, API pipeline, AI agent                  |
| [`notes/`](./notes/)         | Cheatsheets                    | Quick-reference guides for each topic                   |
| [`resources/`](./resources/) | Curated links                  | Best free learning resources per topic                  |

---

## 🔨 Featured Projects

### JSON → XML Converter · [`json/projects/advanced-json-to-xml/`](./json/projects/advanced-json-to-xml/)

Node.js script that reads a JSON product catalog and outputs well-formed XML with proper escaping.  
**Skills:** JSON parsing, XML generation, Node.js I/O, error handling

### XSD Schema Validation · [`xml/projects/intermediate-validation/`](./xml/projects/intermediate-validation/)

Book catalog validated against a typed XSD schema with integer constraints and required attributes.  
**Skills:** XML authoring, XSD design, type constraints, online validation

### XSLT → HTML · [`xml/projects/advanced-xslt-transform/`](./xml/projects/advanced-xslt-transform/)

Product catalog transformed into a styled HTML table with conditional rendering and sorting.  
**Skills:** XSLT templates, xsl:for-each, xsl:choose, xsl:sort, xsltproc

### GitHub Actions CI · [`git/projects/advanced-github-actions/`](./git/projects/advanced-github-actions/)

CI workflow validating all `.json` and `.xml` files on every push using Node and xmllint.  
**Skills:** GitHub Actions YAML, bash scripting, CI/CD, PR-based workflow

### n8n API Pipeline · [`n8n/projects/intermediate-api-pipeline/`](./n8n/projects/intermediate-api-pipeline/)

n8n workflow fetching from a public API, filtering with an IF node, outputting JSON + XML.  
**Skills:** HTTP Request node, Code node, IF branching, XML generation

### n8n AI Agent · [`n8n/projects/advanced-ai-agent/`](./n8n/projects/advanced-ai-agent/)

AI Agent workflow with enforced structured JSON output, Code node validation, and Git-versioned prompt.  
**Skills:** AI Agent node, structured output, prompt engineering, prompt versioning

---

## 📋 Progress

| Task                     | Level        | Status |
| ------------------------ | ------------ | ------ |
| J1 — JSON Config File    | Beginner     | ☐      |
| J2 — Parse API Feed      | Intermediate | ☐      |
| J3 — JSON→XML Converter  | Advanced     | ☐      |
| X1 — XML Library Catalog | Beginner     | ☐      |
| X2 — XSD Validation      | Intermediate | ☐      |
| X3 — XSLT to HTML        | Advanced     | ☐      |
| G1 — First Commit        | Beginner     | ☐      |
| G2 — Branch Workflow     | Intermediate | ☐      |
| G3 — GitHub Actions      | Advanced     | ☐      |
| N1 — Webhook Logger      | Beginner     | ☐      |
| N2 — API Pipeline        | Intermediate | ☐      |
| N3 — AI Agent            | Advanced     | ☐      |

---

## 🛠 Tech Stack

**Formats:** JSON · XML · XSLT · YAML  
**Languages:** JavaScript (Node.js) · Bash  
**Tools:** Git · GitHub Actions · n8n · xmllint · xsltproc · Cursor

---

## 🚀 How to run

```bash
git clone https://github.com/denescsaszar/tc-fundamentals-portfolio.git
cd tc-fundamentals-portfolio

# Node.js projects (requires Node 18+)
cd json/projects/intermediate-api-feed && node parse.js

# XSLT transformation (requires xsltproc)
cd xml/projects/advanced-xslt-transform
xsltproc transform.xsl catalog.xml > output.html && open output.html

# n8n locally
npx n8n
```
