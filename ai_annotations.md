# AI Annotations Specification

## 1. Introduction

The Agent Annotation Standard provides a unified syntax for embedding semantic metadata in code and documentation. Using the `@!` prefix, it enables developers, reviewers, and AI agents to attach machine-readable directives to any textual artifact, bridging the gap between human intent and automated tooling.

---

## 2. Annotation Syntax

AI annotations support two syntax styles that can be used interchangeably depending on your needs.

### 2.1 Inline Syntax

The inline syntax is a single-line annotation that can be placed before any code element—functions, methods, classes, variables, or even at the top of files. It's the most concise form and is ideal for attaching metadata to specific declarations.

**Format:**
```text
@!<annotation-key> [<annotation-value>] { <properties> }
```

**Examples:**
```python
# At the file level
# @!readonly true { "author": "alice" }
# @!link "https://example.com/api" { "tags": ["backend"] }

# Before a function
# @!deprecated "Use process_data_v2 instead"
def process_data(data):
    return data

# Before a class
# @!experimental { "author": "bob", "since": "2024-01-15" }
class DataProcessor:
    pass

# Before a method
# @!method my_method { "author": "alice", "since": "2023-04-01" }
def my_method():
    pass
```

**Scope:** The annotation applies to the declaration that immediately follows it. Multiple annotations can be stacked above a single declaration. No explicit end marker is needed since the annotation is tied to a single element.

**Use cases:**
- Marking individual functions, methods, or classes
- Adding metadata at the file level (at the top of the file)
- Quick annotations that don't need to span multiple lines
- Tagging specific declarations with status, ownership, or links

### 2.2 Block-Level Syntax

The block-level syntax is more verbose but provides tighter control for defining annotated regions. Use this when you need to mark a specific section of code or text that spans multiple lines or contains multiple declarations.

**Format:**
```text
@!begin <key> [<value>] { <props> }
… code or text …
@!end <key>
```

**Examples:**
```python
# @!begin experimental { "author": "bob" }
# This entire section is experimental and may change.
def new_feature():
    pass

def another_experimental_function():
    pass
# @!end experimental

# @!begin refactor { "priority": "high", "assigned": "team-alpha" }
# Legacy code that needs refactoring
legacy_function_1()
legacy_function_2()
# @!end refactor

# @!begin readonly { "author": "bob" }
def internal_function():
    pass
# @!end readonly
```

**Scope:** The annotation applies to the entire region between the `@!begin` and `@!end` markers. The key in the `@!end` marker must match the key in the `@!begin` marker.

**Use cases:**
- Marking regions that span multiple functions or statements
- Annotating code sections that need refactoring or review
- Defining experimental or deprecated blocks
- Creating boundaries for tool processing or AI context
- Precisely controlling the boundaries of the annotated region

### 2.3 Choosing Between Syntax Styles

- **Inline syntax**: Best for annotating specific declarations (functions, classes, variables) or adding file-level metadata. Quick and concise.
- **Block-level syntax**: Best when you need precise control over the boundaries of the annotated region, especially for multi-line sections containing multiple declarations.

Both styles support the same metadata structure (key-value pairs and JSON objects) and can be processed by the same tooling.

---

## 3. Common Annotation Keys

| Key | Value | Description | Example |
|-----|-------|-------------|---------|
| `readonly` | `true/false` | Marks a block or file as immutable | `// @!readonly true` |
| `link` | URL | Points to external resources (docs, issue, code review) | `# @!link "https://github.com/...#L42"` |
| `todo` | String | Human‑readable task description | `// @!todo "Add unit tests"` |
| `license` | SPDX identifier | Declares licensing for the block | `/* @!license MIT */` |
| `agent` | Name | Indicates which automated agent should process the block | `# @!agent "linter"` |
| `metadata` | JSON | Arbitrary key/value pairs for custom tooling | `// @!metadata { "priority": "high" }` |

---

## 4. Tags – Fine‑Grained Context for LLMs

### 4.1 Purpose

Tags are lightweight, reusable descriptors that can be attached to any annotation. They enable **contextual filtering** and **dynamic routing** for language‑model agents, allowing a single source of truth to drive behavior across multiple tools.

### 4.2 Syntax

Tags are expressed as a comma‑separated list inside the annotation value or within the JSON properties block:

```text
@!link "https://example.com" { "tags": ["docs", "frontend"] }
```

or

```text
# @!todo "Fix edge case" tags=["bug", "critical"]
```

When using the comma‑separated form, the parser should interpret it as an array of strings.

### 4.3 Common Tag Names

| Tag | Typical Use | Example |
|-----|-------------|---------|
| `bug` | Indicates a defect | `# @!todo "Resolve crash" tags=["bug"]` |
| `feature` | Marks a new capability | `# @!todo "Add export" tags=["feature"]` |
| `frontend` / `backend` | Scope of the change | `# @!link "..." { "tags": ["frontend"] }` |
| `critical` | High priority | `# @!todo "Security fix" tags=["critical"]` |
| `deprecated` | Obsolete code | `# @!deprecated "Use new API" tags=["deprecated"]` |

### 4.4 Tooling Integration

* **LLM Context Manager** – A lightweight service can query annotations by tags to decide which snippets to feed into a prompt.
* **CI Gatekeeper** – Enforce that critical tags are addressed before merging.
* **Documentation Generator** – Exclude sections tagged `draft` or `internal` from public docs.

### 4.5 Example Workflow

1. **Annotate** a new function with a tag:
   ```python
   # @!method process_data tags=["backend", "critical"]
   def process_data():
       …
   ```
2. **Run** a tag‑aware linter that skips `critical` sections in the staging branch.
3. **Feed** the tagged code to an LLM that only consumes `frontend` snippets for UI suggestions.

---

## 5. Real‑World Use Cases

### 5.1 Source Code

| Scenario | Annotation | Benefit |
|----------|------------|---------|
| **Read‑only sections** | `@!readonly true` | Prevent accidental edits; CI can flag modifications. |
| **Editing instructions** | `@!link "https://docs.example.com/editing#section-42"` | Direct developers to the right guidelines. |
| **Automated linting** | `@!agent "linter"` | Linter can skip or apply special rules. |
| **License attribution** | `@!license GPL-3.0` | Makes licensing explicit for each file/section. |

### 5.2 Documentation

* Deprecated sections: `<!-- @!deprecated "Use new API" -->`
* Support tickets: `<!-- @!link "https://jira.company.com/browse/PROJ-123" -->`
* Review status: `<!-- @!review "approved" -->`

### 5.3 Data Files (JSON/YAML)

* Schema version: `# @!schema "v2.1"`
* Sensitive fields: `# @!secret true`

### 5.4 CI/CD Pipelines

* Skip tests for a branch: `# @!agent "ci" { "skip_tests": true }`

---

## 6. Comment Integration by Language

Annotations work within the comment syntax of any programming language:

| Language | Comment Syntax | Example |
|----------|----------------|---------|
| C/C++/Java | `//` or `/* … */` | `// @!readonly { "author": "alice" }` |
| Python | `#` | `# @!link "https://docs.example.com/editing#section-42"` |
| Markdown | `<!-- … -->` | `<!-- @!todo "Refactor this block" -->` |

---

## 7. Tooling Ecosystem

| Tool | Purpose | `@!` Support |
|------|---------|--------------|
| **Linters** (ESLint, Pylint, etc.) | Skip or enforce rules based on annotations | `@!agent "linter"` |
| **Code Review Bots** | Auto‑comment on TODOs, link to docs | `@!todo`, `@!link` |
| **Documentation Generators** | Include or exclude annotated sections | `@!hidden` |
| **AI Assistants** | Generate explanations, code completions | `@!agent "ai"` |
| **Version‑Control Hooks** | Enforce read‑only blocks | `@!readonly` |
| **Custom Parsers** | Domain‑specific processing | `@!metadata { … }` |

### 7.1 Sample Parser (Python)

```python
import re
from pathlib import Path

ANNOTATION_RE = re.compile(r'''
    ^\s*#\s*@!                    # start marker (example for Python)
    (?P<key>\w+)\s*                # annotation key
    (?P<value>[^{}\s]+)?\s*        # optional value
    (\{(?P<props>[^}]+)\})?        # optional JSON props
''', re.VERBOSE)

def parse_annotations(file_path: Path):
    annotations = []
    for line in file_path.read_text().splitlines():
        m = ANNOTATION_RE.match(line)
        if m:
            annotations.append(m.groupdict())
    return annotations
```

---

## 8. Migration Strategy

1. **Audit** – Use `searchInFiles` to locate all comment blocks that could be annotated.
2. **Define a policy** – Decide which keys are mandatory (e.g., `@!link` for all public APIs).
3. **Create a linter rule** – Flag missing or malformed annotations.
4. **Iterate** – Add annotations gradually; run CI to ensure no regressions.
5. **Document** – Publish a style guide; train developers.

---

## 9. Best Practices

* Keep annotations short – use concise keys; elaborate in the linked docs.
* Avoid over‑annotation – only annotate where automation or clarity benefits.
* Version your annotations – include a `version` property if the semantics evolve.
* Test your parsers – write unit tests for common patterns.
* Encourage community feedback – open the spec for extensions.

---

## 10. Future Directions

* **Standardized schema registry** – A central JSON‑Schema that all tools can reference.
* **IDE integration** – Inline rendering of annotation metadata (e.g., hover tooltips).
* **AI‑driven annotation generation** – Auto‑suggest tags based on code context.
* **Cross‑language adapters** – Plug‑in libraries for Rust, Go, etc.

---

## 11. Conclusion

The Agent Annotation Standard, using the concise `@!` prefix, offers a **single, coherent language** for embedding semantic metadata in any textual artefact. By bridging the gap between human intent and machine‑readable directives, AAS empowers developers, reviewers, and AI agents to work more efficiently, reduces friction in tooling pipelines, and promotes clearer communication across teams.

---

## 12. Sample Project

Below is a minimal Python project demonstrating the use of annotations.

```
# project/README.md
# @!link "https://github.com/example/project" { "tags": ["docs", "repo"] }

# project/main.py
# @!readonly true { "author": "alice" }
# @!link "https://example.com/api" { "tags": ["backend"] }

def process_data(data):
    # @!todo "Handle edge cases" tags=["bug", "critical"]
    return data

# @!begin experimental { "author": "bob" }
# This block is experimental and may change.
# @!end experimental
```

The accompanying parser can extract all annotations and feed them to a CI job or an LLM prompt.

---

## Appendix A: Quick Syntax Reference

For detailed explanation of annotation syntax, see **Section 2**.

**Inline Syntax:**
```text
@!<annotation-key> [<annotation-value>] { <properties> }
```

**Block-Level Syntax:**
```text
@!begin <key> [<value>] { <props> }
… code or text …
@!end <key>
```

| Element | Meaning | Example |
|---------|---------|---------|
| `@!` | Marker that introduces an annotation | `@!readonly true` |
| `<annotation-key>` | Identifier (snake_case, kebab-case, camelCase) | `readonly`, `link`, `todo` |
| `<annotation-value>` | Optional scalar or structured value (string, number, array) | `true`, `"https://example.com"` |
| `{ <properties> }` | Optional JSON‑like block with additional metadata (e.g., `author`, `date`) | `{ "author": "alice", "since": "2023-05-01" }` |

### JSON‑Object Rules

* Keys must be double‑quoted strings.
* Values can be strings, numbers, booleans, null, arrays, or nested objects.
* No comments are allowed inside the JSON block.

---

## Appendix B: Glossary

| Term | Definition |
|------|------------|
| **Annotation** | A metadata marker prefixed with `@!` that attaches information to code or text. |
| **Tag** | A lightweight label used to group or filter annotations, typically for LLMs or tooling. |
| **Agent** | An automated process (e.g., linter, CI, AI assistant) that consumes annotations to modify behavior. |
| **Inline Syntax** | A single-line annotation that can be placed before any code element (functions, methods, classes, variables, or at the top of files). |
| **Block‑Level Syntax** | An annotation that applies to a contiguous region of text between `@!begin` and `@!end`, providing precise control over annotated boundaries. |

---

## Appendix C: FAQ

| Question | Answer |
|----------|--------|
| **Can I use `@!` inside string literals?** | Yes, but the parser should ignore annotations that are not at the start of a line or are inside quotes. Use a language‑specific tokenizer if you need stricter rules. |
| **What if my language uses `@` for decorators?** | The `@!` prefix is distinct from a single `@`. If your language already has `@` for decorators, you can still use `@!` as it will be parsed as a comment or string literal. |
| **Do I need to commit the annotations to source control?** | Absolutely. Annotations are part of the code/documentation and should be versioned. |
| **Can I nest annotations?** | Nesting is not formally supported. If you need hierarchical metadata, use the `metadata` key to embed a JSON object. |

---

## Appendix D: References

* [JSON Schema](https://json-schema.org/)
* [PEP 484 – Type Hints](https://www.python.org/dev/peps/pep-0484/)
* [GitHub Actions](https://docs.github.com/en/actions)
* [OpenAI API](https://platform.openai.com/docs)

---

## Appendix E: Contribution Guidelines

1. Fork the repository and create a feature branch.
2. Add or update the specification in `ai_annotations.md` following the existing style.
3. Update the parser examples if you add new syntax.
4. Add unit tests for the parser in `tests/`.
5. Submit a pull request with a clear description of the changes.

---
