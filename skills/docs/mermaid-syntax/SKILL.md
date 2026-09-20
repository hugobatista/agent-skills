---
name: mermaid-syntax
author: hugobatista
description: >
  Correct Mermaid diagram syntax reference covering all types (flowchart,
  sequence, class, ERD, state, C4, git, gantt, pie). Emphasizes parser-compatible
  patterns to avoid rendering errors in GitHub, Obsidian, VS Code, and other tools.
  Use when creating, visualizing, or documenting software architecture through
  diagrams.
---

# Mermaid Syntax Reference

## Syntax Compatibility Rules

These 5 rules prevent parser errors across common rendering tools (GitHub, Obsidian, VS Code extensions):

1. **Subgraphs**: use `subgraph "Title"` — NOT `subgraph ID["Title"]` (named subgraphs require mermaid ≥10.3)
2. **Diamond nodes**: always give an explicit ID — `DECISION{"text"}` — NOT anonymous `{"text"}` after `-->`
3. **Line breaks**: `\n` works inside `["text"]`; avoid `\n` inside `{"text"}` — keep diamond labels single-line
4. **Sequence notes**: plain text only — no `<br/>`, `<br>`, or other HTML tags
5. **Special characters**: replace `&` with `and` to avoid HTML entity issues

## Core Syntax

Every diagram starts with a type declaration on the first line:

```mermaid
flowchart LR
sequenceDiagram
classDiagram
erDiagram
stateDiagram-v2
gitGraph
gantt
pie
```

Use `%%` for comments. Indentation improves readability but isn't required.

## Flowchart

### Node shapes

| Syntax | Shape |
|---|---|
| `A[text]` | Rectangle |
| `A(text)` | Rounded rectangle |
| `A([text])` | Stadium / pill |
| `A[[text]]` | Subroutine |
| `A[(text)]` | Database / cylinder |
| `A((text))` | Circle |
| `A{text}` | Rhombus / decision |
| `A{{text}}` | Hexagon |
| `A>text]` | Asymmetric / flag |
| `A[/text/]` | Parallelogram |
| `A[\text\]` | Parallelogram (reverse) |
| `A[/text\]` | Trapezoid |

### Edges

| Syntax | Type |
|---|---|
| `A --> B` | Arrow |
| `A --- B` | Link (no arrow) |
| `A -.-> B` | Dotted arrow |
| `A ==> B` | Thick arrow |
| `A -->\|label\| B` | Labeled arrow (preferred) |
| `A -- label --> B` | Labeled arrow (alt) |

### Subgraphs

Always use quoted titles:

```mermaid
flowchart LR
    subgraph "Group Name"
        A --> B
    end
```

### Example

```mermaid
flowchart TD
    Start([Start]) --> Auth{Authenticated?}
    Auth -->|Yes| Dashboard[Show dashboard]
    Auth -->|No| Login[Show login form]
    Login --> Auth
```

## Sequence Diagram

### Participants

```mermaid
sequenceDiagram
    participant A
    participant B as "Display Name"
    actor User
```

### Messages

| Syntax | Type |
|---|---|
| `->>` | Synchronous call |
| `-->>` | Synchronous response |
| `-)` | Asynchronous call |
| `--)` | Asynchronous response |
| `-x` | Delete / loss |

### Structure blocks

```mermaid
sequenceDiagram
    participant A
    participant B

    alt Condition
        A->>B: branch
    else Other
        A->>B: other branch
    end

    opt Optional
        A->>B: optional
    end

    loop Every N seconds
        A->>B: repeat
    end

    par Parallel
        A->>B: parallel 1
    and
        A->>B: parallel 2
    end
```

### Notes

Plain text only — no HTML tags:

```mermaid
sequenceDiagram
    participant App
    Note over App: Plain text description
    Note right of App: Plain text
    Note left of App: Plain text
```

### Example

```mermaid
sequenceDiagram
    participant App
    participant DB

    App->>DB: Query
    DB-->>App: Result

    alt Found
        App->>App: Process
    else Not found
        App->>App: Handle error
    end

    Note over App: Plain text only
```

## Class Diagram

### Class declaration

```mermaid
classDiagram
    class ClassName {
        +publicField
        -privateField
        #protectedField
        +method() returnType
        -method() void
    }
```

### Relationships

| Syntax | Relationship |
|---|---|
| `<\|--` | Inheritance |
| `*--` | Composition |
| `o--` | Aggregation |
| `-->` | Association |
| `..\|>` | Realization |
| `..>` | Dependency |
| `--` | Link |

Multiplicity: `"1" --> "*"`

### Example

```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    Animal <|-- Zebra
    Animal : +int age
    Animal : +isMammal() bool
    Duck : +beakColor string
    Duck : +swim()
```

## Entity-Relationship Diagram

### Cardinality

| Symbol | Meaning |
|---|---|
| `\|o` | Zero or one |
| `\|\|` | Exactly one |
| `}o` | Zero or more |
| `}\|` | One or more |

### Example

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ LINE_ITEM : contains
    PRODUCT ||--o{ LINE_ITEM : includes
    CUSTOMER {
        int id PK
        string name
    }
    ORDER {
        int id PK
        decimal total
    }
```

## State Diagram

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Processing : start
    Processing --> Completed : finish
    Processing --> Failed : error
    Failed --> Idle : retry
    Completed --> [*]
```

Composite states:

```mermaid
stateDiagram-v2
    state Active {
        [*] --> Working
        Working --> Waiting
        Waiting --> Working
    }
```

## C4 Diagram

```mermaid
C4Context
    Person(user, "Customer", "Bank customer")
    System(bankingSystem, "Banking System", "Handles accounts")
    System_Ext(mail, "Email System")
    Rel(user, bankingSystem, "Uses")
    Rel(bankingSystem, mail, "Sends emails")
```

Other levels: `C4Container`, `C4Component`, `C4Deployment`.

Boundaries: `System_Boundary(name, "Label") { ... }`, `Container_Boundary`, etc.

## Git Graph

```mermaid
gitGraph
    commit
    branch feature
    checkout feature
    commit
    checkout main
    merge feature
```

## Gantt Chart

```mermaid
gantt
    title Project Timeline
    dateFormat YYYY-MM-DD
    section Design
    Research    :2024-01-01, 30d
    Prototype   :2024-01-15, 20d
    section Dev
    Backend     :2024-02-01, 45d
    Frontend    :2024-02-15, 30d
```

## Pie Chart

```mermaid
pie title Distribution
    "Category A" : 45
    "Category B" : 30
    "Category C" : 25
```

## Common Pitfalls

- **Subgraph syntax**: `subgraph Name["Label"]` fails in many renderers — use `subgraph "Label"` instead
- **Anonymous diamond nodes**: `-->{"text"}` causes parse errors — use `--> DECISION{"text"}` with an explicit ID
- **HTML in sequence notes**: `<br/>` renders literally — use plain text
- **`\n` in diamonds**: `{"line1\nline2"}` is not supported — keep text single-line
- **`&` character**: triggers HTML entity issues — write `and` instead
- **Unclosed brackets**: every `[`, `(`, `{` needs a matching closing bracket
- **Unknown words**: misspellings break diagrams; validate at https://mermaid.live

## Best Practices

1. **Start simple** — build up complexity incrementally
2. **Use action-oriented labels** — "Process payment" not "Payment"
3. **One concept per diagram** — split complex systems into focused views
4. **Validate syntax** — test in mermaid.live before committing
