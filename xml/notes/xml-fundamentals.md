# 📄 xml-fundamentals.md

## XML: Structure, Validation, and Transformation

> **Goal:** Read, write, and validate XML — and understand why it still powers critical systems in 2026.

---

## 1. What is XML?

XML (eXtensible Markup Language) is a **text-based format for structured data**. Unlike HTML, which has fixed tags (`<div>`, `<p>`), XML lets you define your own tags. This flexibility made it the standard for data interchange in enterprise systems, configuration, and document formats.

> **Why this matters:** XML powers RSS feeds, SVG graphics, Microsoft Office files (`.docx` is a zip of XML), Android layouts, Maven/Gradle build configs, and countless enterprise APIs. You will encounter it.

> ⚠️ **Common mistake:** Thinking XML is obsolete. It's been partially replaced by JSON in web APIs — but it dominates in document formats, financial systems (XBRL, FpML), publishing (EPUB, DocBook), and anywhere strong schema validation is needed.

---

## 2. Elements and Attributes

### Elements

```xml
<?xml version="1.0" encoding="UTF-8"?>
<library>
  <book>
    <title>Clean Code</title>
    <author>Robert C. Martin</author>
    <year>2008</year>
    <available>true</available>
  </book>
  <book>
    <title>The Pragmatic Programmer</title>
    <author>David Thomas</author>
    <year>2019</year>
    <available>false</available>
  </book>
</library>
```

Rules:

- One **root element** per document — everything lives inside it
- Tags are **case-sensitive**: `<Title>` ≠ `<title>`
- All tags must be **properly closed**: `<tag>content</tag>` or self-closing `<tag/>`
- Elements must be **properly nested**: `<a><b></b></a>` ✓ — `<a><b></a></b>` ✗

### Attributes

```xml
<book id="B001" category="programming" lang="en">
  <title>Clean Code</title>
  <author>Robert C. Martin</author>
</book>
```

> **Why this matters:** The choice between element and attribute is a design decision. Rule of thumb: use attributes for metadata about the element (IDs, types, flags) and child elements for actual content or data.

> ⚠️ **Common mistake:** Overloading attributes with complex data. If a value is itself structured or could have multiple instances, it belongs as a child element, not an attribute.

---

## 3. Namespaces

Namespaces prevent **name collisions** when combining XML from different sources.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<root
  xmlns:book="http://example.com/book"
  xmlns:author="http://example.com/author">

  <book:title>Clean Code</book:title>
  <author:name>Robert C. Martin</author:name>
  <author:country>USA</author:country>

</root>
```

SVG uses namespaces too:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100">
  <circle cx="50" cy="50" r="40" fill="blue"/>
</svg>
```

> **Why this matters:** You need to handle namespaces when parsing XML from enterprise APIs or processing SVGs programmatically. Ignoring them will cause your XPath queries to fail silently.

> ⚠️ **Common mistake:** Forgetting that `xmlns="..."` (default namespace) affects all unprefixed elements. When you query for `<title>` in a namespaced document, you must include the namespace in your XPath or the query returns nothing.

---

## 4. DTD vs XSD Validation

### DTD — Simple but Limited

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE library [
  <!ELEMENT library (book+)>
  <!ELEMENT book (title, author, year)>
  <!ELEMENT title (#PCDATA)>
  <!ELEMENT author (#PCDATA)>
  <!ELEMENT year (#PCDATA)>
  <!ATTLIST book id ID #REQUIRED>
]>
<library>
  <book id="B001">
    <title>Clean Code</title>
    <author>Robert C. Martin</author>
    <year>2008</year>
  </book>
</library>
```

DTD can say "a book has a title, author, and year." It **cannot** say "year must be a number between 1900 and 2099."

### XSD — Precise and Typed

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

  <xs:element name="library">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="book" maxOccurs="unbounded">
          <xs:complexType>
            <xs:sequence>
              <xs:element name="title" type="xs:string"/>
              <xs:element name="author" type="xs:string"/>
              <xs:element name="year">
                <xs:simpleType>
                  <xs:restriction base="xs:integer">
                    <xs:minInclusive value="1900"/>
                    <xs:maxInclusive value="2099"/>
                  </xs:restriction>
                </xs:simpleType>
              </xs:element>
            </xs:sequence>
            <xs:attribute name="id" type="xs:ID" use="required"/>
          </xs:complexType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>

</xs:schema>
```

| Feature    | DTD              | XSD              |
| ---------- | ---------------- | ---------------- |
| Syntax     | Compact, non-XML | XML itself       |
| Data types | None (only text) | Full type system |
| Namespaces | Limited          | Full support     |
| Used in    | Legacy systems   | Modern systems   |

> **Why this matters:** XSD is how enterprise systems enforce data contracts. A bank won't accept your XML without it passing XSD validation.

> ⚠️ **Common mistake:** Starting with XSD for small personal projects. Use DTD for quick validation during learning; move to XSD when you need type constraints.

---

## 5. XPath Basics

Given this document:

```xml
<library>
  <book id="B001" category="programming">
    <title>Clean Code</title>
    <author>Robert C. Martin</author>
    <year>2008</year>
  </book>
  <book id="B002" category="fiction">
    <title>1984</title>
    <author>George Orwell</author>
    <year>1949</year>
  </book>
</library>
```

| XPath Expression                         | Selects                                  |
| ---------------------------------------- | ---------------------------------------- |
| `/library`                               | The root `library` element               |
| `/library/book`                          | All `book` elements                      |
| `/library/book[1]`                       | First `book` (XPath is 1-indexed)        |
| `/library/book/@id`                      | The `id` attribute of all books          |
| `/library/book[year>2000]/title`         | Titles of books after 2000               |
| `//title`                                | All `title` elements anywhere in the doc |
| `/library/book[@category='programming']` | Books with category="programming"        |

> **Why this matters:** XPath is used in XSLT transformations, XML testing frameworks, and document processing pipelines.

> ⚠️ **Common mistake:** Using `//` everywhere. `//title` searches the entire document tree — it's slow on large documents. Prefer specific paths like `/library/book/title`.

---

## 6. XSLT Introduction

**XSLT** transforms XML into other formats — HTML, another XML structure, plain text, CSV.

**Input (`catalog.xml`):**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<library>
  <book id="B001">
    <title>Clean Code</title>
    <author>Robert C. Martin</author>
    <year>2008</year>
  </book>
  <book id="B002">
    <title>1984</title>
    <author>George Orwell</author>
    <year>1949</year>
  </book>
</library>
```

**Stylesheet (`transform.xsl`):**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
  xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

  <xsl:output method="html" indent="yes"/>

  <xsl:template match="/">
    <html>
      <body>
        <h1>Book Catalog</h1>
        <table border="1">
          <tr>
            <th>Title</th>
            <th>Author</th>
            <th>Year</th>
          </tr>
          <xsl:for-each select="library/book">
            <tr>
              <td><xsl:value-of select="title"/></td>
              <td><xsl:value-of select="author"/></td>
              <td><xsl:value-of select="year"/></td>
            </tr>
          </xsl:for-each>
        </table>
      </body>
    </html>
  </xsl:template>

</xsl:stylesheet>
```

Apply it:

```bash
# macOS / Linux (pre-installed)
xsltproc transform.xsl catalog.xml > output.html
```

> **Why this matters:** XSLT is how publishing systems, invoice generators, and data pipelines transform structured data into presentable output.

> ⚠️ **Common mistake:** Writing XSLT without an XML-aware editor. Use VS Code with the XML extension — it catches namespace and syntax errors before you run the transform.

---

## 7. Real-World Use Cases

### RSS Feed

```xml
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
  <channel>
    <title>Dev Blog</title>
    <link>https://devblog.example.com</link>
    <item>
      <title>Understanding JSON Schema</title>
      <link>https://devblog.example.com/json-schema</link>
      <pubDate>Fri, 24 Apr 2026 09:00:00 +0000</pubDate>
    </item>
  </channel>
</rss>
```

### SVG

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 200 100">
  <rect x="10" y="10" width="80" height="80" fill="#4f98a3" rx="8"/>
  <text x="110" y="60" font-size="16" fill="#28251d">Hello SVG</text>
</svg>
```

### Maven `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <groupId>com.example</groupId>
  <artifactId>my-app</artifactId>
```
