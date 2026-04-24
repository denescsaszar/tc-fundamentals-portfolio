# 📄 cursor-tasks.md

## Coding Tasks — JSON, XML, Git

> **How to use:**
>
> 1. Open Cursor in your repo root
> 2. Navigate to the correct project subfolder before starting
> 3. Read the full task before writing a single line
> 4. Commit when done: `git commit -m "feat: complete [task name]"`
> 5. Each task is completable in under 45 minutes

---

# JSON Tasks

---

## Task J1 — Beginner: Build a Config File

**Folder:** `json/projects/beginner-syntax/`
**Time:** ~20 minutes
**Produces:** `config.json` + `README.md`

### Instructions

Create `config.json` with:

1. An `app` section: `name` (string), `version` (string, e.g. `"1.0.0"`), `debug` (boolean `false`)
2. A `server` section: `host` (`"localhost"`), `port` (`3000`), `https` (`false`)
3. A `database` section: `url` (string), `poolSize` (`5`), `ssl` (boolean)
4. A `features` object with 4 feature flags: `darkMode`, `notifications`, `analytics`, `betaFeatures`
5. An `allowedOrigins` array with at least 2 strings

### Scaffold

```json
{
  "app": {},
  "server": {},
  "database": {},
  "features": {},
  "allowedOrigins": []
}
```

### Validate it

```bash
node -e "
  const fs = require('fs');
  try {
    const cfg = JSON.parse(fs.readFileSync('./config.json', 'utf8'));
    console.log('✓ Valid JSON');
    console.log('✓ App name:', cfg.app.name);
    console.log('✓ Server port:', cfg.server.port);
    console.log('✓ Features:', Object.keys(cfg.features).join(', '));
    console.log('✓ Origins:', cfg.allowedOrigins.length, 'entries');
  } catch(e) {
    console.error('✗ Error:', e.message);
  }
"
```

### Commit when done

```bash
git add json/projects/beginner-syntax/
git commit -m "feat: J1 — JSON config file with all data types"
```

---

## Task J2 — Intermediate: Parse a JSON API Feed

**Folder:** `json/projects/intermediate-api-feed/`
**Time:** ~35 minutes
**Produces:** `feed.json` + `parse.js` + `README.md`

### Step 1 — Create `feed.json`

```json
{
  "status": "ok",
  "timestamp": "2026-04-24T09:00:00Z",
  "data": {
    "books": [
      {
        "id": "B001",
        "title": "Clean Code",
        "author": "Robert C. Martin",
        "year": 2008,
        "available": true,
        "tags": ["programming", "best-practices"]
      },
      {
        "id": "B002",
        "title": "The Pragmatic Programmer",
        "author": "David Thomas",
        "year": 2019,
        "available": true,
        "tags": ["programming", "career"]
      },
      {
        "id": "B003",
        "title": "Design Patterns",
        "author": "Gang of Four",
        "year": 1994,
        "available": false,
        "tags": ["programming", "architecture"]
      },
      {
        "id": "B004",
        "title": "Refactoring",
        "author": "Martin Fowler",
        "year": 2018,
        "available": true,
        "tags": ["programming", "best-practices"]
      }
    ],
    "total": 4,
    "page": 1
  }
}
```

### Step 2 — Create `parse.js`

```javascript
const fs = require("fs");

// 1. Read and parse safely
let feed;
try {
  const raw = fs.readFileSync("./feed.json", "utf8");
  feed = JSON.parse(raw);
} catch (err) {
  console.error("Failed to parse feed.json:", err.message);
  process.exit(1);
}

// 2. Check status
if (feed.status !== "ok") {
  console.error("API returned non-ok status:", feed.status);
  process.exit(1);
}

const books = feed.data.books;

// 3. Filter available books
const available = books.filter((b) => b.available === true);

// 4. Print each available book
console.log("Available Books:");
console.log("─".repeat(40));
available.forEach((b) => {
  console.log(`• ${b.title} — ${b.author} (${b.year})`);
});

// 5. Print count
console.log(`\nTotal available: ${available.length} of ${books.length}`);

// 6. Print unique tags
const allTags = books.flatMap((b) => b.tags);
const uniqueTags = [...new Set(allTags)].sort();
console.log("\nAll unique tags:", uniqueTags.join(", "));
```

### Run it

```bash
node parse.js
```

### Commit when done

```bash
git add json/projects/intermediate-api-feed/
git commit -m "feat: J2 — JSON API feed parser with filtering and tag deduplication"
```

---

## Task J3 — Advanced: JSON-to-XML Converter

**Folder:** `json/projects/advanced-json-to-xml/`
**Time:** ~45 minutes
**Produces:** `input.json` + `output.xml` + `convert.js` + `README.md`

### Step 1 — Create `input.json`

```json
{
  "catalog": {
    "version": "1.0",
    "products": [
      {
        "id": "P001",
        "name": "Wireless Keyboard",
        "price": 49.99,
        "inStock": true,
        "tags": ["electronics", "peripherals"]
      },
      {
        "id": "P002",
        "name": "USB-C Hub",
        "price": 29.99,
        "inStock": false,
        "tags": ["electronics", "accessories"]
      },
      {
        "id": "P003",
        "name": "Mechanical Pencil Set",
        "price": 12.5,
        "inStock": true,
        "tags": ["stationery"]
      }
    ]
  }
}
```

### Step 2 — Create `convert.js`

```javascript
const fs = require("fs");

function escapeXml(str) {
  return String(str)
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;");
}

function productToXml(product) {
  const tags = product.tags
    .map((t) => `      <tag>${escapeXml(t)}</tag>`)
    .join("\n");

  return `  <product id="${escapeXml(product.id)}">
    <name>${escapeXml(product.name)}</name>
    <price>${product.price}</price>
    <inStock>${product.inStock}</inStock>
    <tags>
${tags}
    </tags>
  </product>`;
}

function convertToXml(data) {
  const { catalog } = data;
  const productBlocks = catalog.products.map(productToXml).join("\n");

  return `<?xml version="1.0" encoding="UTF-8"?>
<catalog version="${escapeXml(catalog.version)}">
${productBlocks}
</catalog>`;
}

try {
  const raw = fs.readFileSync("./input.json", "utf8");
  const data = JSON.parse(raw);
  const xml = convertToXml(data);
  fs.writeFileSync("./output.xml", xml, "utf8");
  console.log(
    `✓ Converted ${data.catalog.products.length} products to output.xml`,
  );
} catch (err) {
  console.error("Error:", err.message);
}
```

### Run it

```bash
node convert.js
# Validate the output
xmllint --noout output.xml && echo "✓ XML valid"
```

### Commit when done

```bash
git add json/projects/advanced-json-to-xml/
git commit -m "feat: J3 — JSON to XML converter with escaping"
```

---

# XML Tasks

---

## Task X1 — Beginner: Build a Library Catalog

**Folder:** `xml/projects/beginner-elements/`
**Time:** ~20 minutes
**Produces:** `library.xml` + `README.md`

Create `library.xml` with at least 5 books. Must have:

- XML declaration on line 1
- Single root `<library name="...">` element
- Each `<book>` with `id` and `category` attributes
- Child elements: `<title>`, `<author>`, `<year>`, `<isbn>`, `<available>`
- At least one `<description>` element

### Scaffold

```xml
<?xml version="1.0" encoding="UTF-8"?>
<library name="My Dev Library">

  <book id="B001" category="programming">
    <title></title>
    <author></author>
    <year></year>
    <isbn></isbn>
    <available></available>
  </book>

  <!-- Add 4 more books -->

</library>
```

Validate at [xmlvalidation.com](https://www.xmlvalidation.com) before committing.

### Commit when done

```bash
git add xml/projects/beginner-elements/
git commit -m "feat: X1 — XML library catalog with 5 books"
```

---

## Task X2 — Intermediate: Validate XML Against XSD

**Folder:** `xml/projects/intermediate-validation/`
**Time:** ~35 minutes
**Produces:** `books.xml` + `books.xsd` + `books-invalid.xml` + `README.md`

### Create `books.xsd`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

  <xs:element name="catalog">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="book" minOccurs="1" maxOccurs="unbounded">
          <xs:complexType>
            <xs:sequence>
              <xs:element name="title" type="xs:string"/>
              <xs:element name="author" type="xs:string"/>
              <xs:element name="year">
                <xs:simpleType>
                  <xs:restriction base="xs:integer">
                    <xs:minInclusive value="1800"/>
                    <xs:maxInclusive value="2030"/>
                  </xs:restriction>
                </xs:simpleType>
              </xs:element>
              <xs:element name="price">
                <xs:simpleType>
                  <xs:restriction base="xs:decimal">
                    <xs:minInclusive value="0"/>
                  </xs:restriction>
                </xs:simpleType>
              </xs:element>
            </xs:sequence>
            <xs:attribute name="id" type="xs:ID" use="required"/>
            <xs:attribute name="category" type="xs:string" use="required"/>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>

</xs:schema>
```

### Create `books.xml` (must pass validation)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="books.xsd">
  <book id="B001" category="programming">
    <title>Clean Code</title>
    <author>Robert C. Martin</author>
    <year>2008</year>
    <price>34.99</price>
  </book>
  <book id="B002" category="programming">
    <title>The Pragmatic Programmer</title>
    <author>David Thomas</author>
    <year>2019</year>
    <price>44.99</price>
  </book>
  <book id="B003" category="fiction">
    <title>1984</title>
    <author>George Orwell</author>
    <year>1949</year>
    <price>12.99</price>
  </book>
</catalog>
```

### Create `books-invalid.xml` with 3 intentional errors:

- A year of `1750` (below minimum)
- A missing required `category` attribute
- A price of `-5` (below minimum)

Validate both files at [xmlvalidation.com](https://www.xmlvalidation.com).
Copy the error messages into your `README.md`.

### Commit when done

```bash
git add xml/projects/intermediate-validation/
git commit -m "feat: X2 — XSD schema validation with valid and invalid examples"
```

---

## Task X3 — Advanced: XSLT Transformation to HTML

**Folder:** `xml/projects/advanced-xslt-transform/`
**Time:** ~45 minutes
**Produces:** `catalog.xml` + `transform.xsl` + `output.html` + `README.md`

### Create `catalog.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
  <product id="P001" category="electronics">
    <name>Wireless Keyboard</name>
    <price currency="EUR">49.99</price>
    <inStock>true</inStock>
    <description>Compact wireless keyboard with 2-year battery life.</description>
  </product>
  <product id="P002" category="electronics">
    <name>USB-C Hub</name>
    <price currency="EUR">29.99</price>
    <inStock>false</inStock>
    <description>7-in-1 hub with HDMI, USB-A, SD card reader.</description>
  </product>
  <product id="P003" category="stationery">
    <name>Mechanical Pencil Set</name>
    <price currency="EUR">12.50</price>
    <inStock>true</inStock>
    <description>Set of 3 mechanical pencils with spare lead.</description>
  </product>
</catalog>
```

### Create `transform.xsl`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="html" indent="yes"/>

  <xsl:template match="/">
    <html lang="en">
      <head>
        <meta charset="UTF-8"/>
        <title>Product Catalog</title>
        <style>
          body { font-family: sans-serif; max-width: 800px; margin: 2rem auto; padding: 1rem; }
          table { width: 100%; border-collapse: collapse; }
          th { background: #01696f; color: white; padding: 0.75rem; text-align: left; }
          td { padding: 0.75rem; border-bottom: 1px solid #ddd; }
          .in-stock { color: green; font-weight: bold; }
          .out-of-stock { color: red; }
        </style>
      </head>
      <body>
        <h1>Product Catalog</h1>
        <table>
          <tr>
            <th>ID</th><th>Name</th><th>Category</th>
            <th>Price</th><th>Stock</th><th>Description</th>
          </tr>
          <xsl:for-each select="catalog/product">
            <tr>
              <td><xsl:value-of select="@id"/></td>
              <td><xsl:value-of select="name"/></td>
              <td><xsl:value-of select="@category"/></td>
              <td><xsl:value-of select="price/@currency"/>&#160;<xsl:value-of select="price"/></td>
              <td>
                <xsl:choose>
                  <xsl:when test="inStock = 'true'">
                    <span class="in-stock">In Stock</span>
                  </xsl:when>
                  <xsl:otherwise>
                    <span class="out-of-stock">Out of Stock</span>
                  </xsl:otherwise>
                </xsl:choose>
              </td>
              <td><xsl:value-of select="description"/></td>
            </tr>
          </xsl:for-each>
        </table>
      </body>
    </html>
  </xsl:template>
</xsl:stylesheet>
```

### Apply the transform

```bash
xsltproc transform.xsl catalog.xml > output.html
open output.html
```

### Commit when done

```bash
git add xml/projects/advanced-xslt-transform/
git commit -m "feat: X3 — XSLT transformation from XML catalog to HTML table"
```

---

# Git Tasks

---

## Task G1 — Beginner: Your First Real Commit Workflow

**Folder:** `git/projects/beginner-first-commit/`
**Time:** ~20 minutes

```bash
cd git/projects/beginner-first-commit
echo "# My First Git Task" > README.md
echo "practicing git" > notes.txt

git status
git add README.md
git status
git add notes.txt
git commit -m "feat: add beginner git task files"
git log --oneline

echo "Git tracks changes, not files." >> notes.txt
git add notes.txt
git commit -m "docs: add note about how git works"
git log --oneline
```

In `README.md`, paste your `git log --oneline` output and answer:

- What does the staging area do?
- Why stage files individually instead of `git add .`?

```bash
git add git/projects/beginner-first-commit/
git commit -m "feat: G1 — first commit workflow documented"
```

---

## Task G2 — Intermediate: Feature Branch Workflow

**Folder:** `git/projects/intermediate-branch-workflow/`
**Time:** ~35 minutes

```bash
git checkout main
git checkout -b feature/add-library-catalog

# Create a file on the branch
mkdir -p git/projects/intermediate-branch-workflow
cat > git/projects/intermediate-branch-workflow/library.json << 'EOF'
{
  "library": {
    "name": "Dev Library",
    "books": [
      { "id": "B001", "title": "Clean Code", "available": true },
      { "id": "B002", "title": "The Pragmatic Programmer", "available": true }
    ]
  }
}
EOF

git add git/projects/intermediate-branch-workflow/library.json
git commit -m "feat: add initial library catalog JSON"

# Go back to main — file disappears
git checkout main
ls git/projects/intermediate-branch-workflow/

# Back to branch — file reappears
git checkout feature/add-library-catalog
ls git/projects/intermediate-branch-workflow/

# Merge into main
git checkout main
git merge feature/add-library-catalog
git log --oneline --graph

# Clean up
git branch -d feature/add-library-catalog
```

Add a `README.md` explaining what you observed when switching branches.

```bash
git add git/projects/intermediate-branch-workflow/
git commit -m "feat: G2 — feature branch workflow demonstrated"
```

---

## Task G3 — Advanced: GitHub Actions CI

**Folder:** `git/projects/advanced-github-actions/`
**Time:** ~45 minutes

```bash
mkdir -p .github/workflows
```

Create `.github/workflows/validate.yml`:

```yaml
name: Validate Data Files

on:
  push:
    branches: ["**"]
  pull_request:

jobs:
  validate-json:
    name: Validate JSON files
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
      - name: Validate all JSON files
        run: |
          ERRORS=0
          for file in $(find . -name "*.json" \
              -not -path "*/node_modules/*" \
              -not -path "*/.git/*"); do
            echo -n "Checking $file ... "
            if node -e "JSON.parse(require('fs').readFileSync('$file','utf8'))" 2>/dev/null; then
              echo "✓"
            else
              echo "✗ INVALID"
              ERRORS=$((ERRORS+1))
            fi
          done
          [ $ERRORS -eq 0 ] || exit 1

  validate-xml:
    name: Validate XML files
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: sudo apt-get install -y libxml2-utils
      - name: Validate all XML files
        run: |
          ERRORS=0
          for file in $(find . -name "*.xml" \
              -not -path "*/.git/*" \
              -not -name "*.xsl"); do
            echo -n "Checking $file ... "
            if xmllint --noout "$file" 2>/dev/null; then
              echo "✓"
            else
              echo "✗ INVALID"
              ERRORS=$((ERRORS+1))
            fi
          done
          [ $ERRORS -eq 0 ] || exit 1
```

```bash
git add .github/workflows/validate.yml
git commit -m "ci: add GitHub Actions workflow to validate JSON and XML files"
git push origin main
```

Go to your repo → **Actions tab** → watch it run.

---

## Task Completion Tracker

| Task                    | Topic            | Level        | Status |
| ----------------------- | ---------------- | ------------ | ------ |
| J1 — Config File        | JSON             | Beginner     | ☐      |
| J2 — Parse API Feed     | JSON             | Intermediate | ☐      |
| J3 — JSON→XML Converter | JSON + XML       | Advanced     | ☐      |
| X1 — Library Catalog    | XML              | Beginner     | ☐      |
| X2 — XSD Validation     | XML              | Intermediate | ☐      |
| X3 — XSLT to HTML       | XML + XSLT       | Advanced     | ☐      |
| G1 — First Commit       | Git              | Beginner     | ☐      |
| G2 — Branch Workflow    | Git              | Intermediate | ☐      |
| G3 — GitHub Actions CI  | Git + JSON + XML | Advanced     | ☐      |
| N1 — Webhook Logger     | n8n              | Beginner     | ☐      |
| N2 — API Pipeline       | n8n + JSON + XML | Intermediate | ☐      |
| N3 — AI Agent           | n8n + AI + Git   | Advanced     | ☐      |
