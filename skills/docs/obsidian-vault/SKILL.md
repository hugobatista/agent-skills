---
name: obsidian-vault
description: Read, search, and save structured findings to any Obsidian vault. Works across projects and vaults.
author: hugobatista
---

Read, search, and save structured findings to any Obsidian vault. Works across projects and vaults — configured by `vaults.json` in this directory.

## Trigger phrases

**Writing:** "document this in {vault} vault", "document in {vault} obsidian",
"save to vault", "write a finding", "save finding",
"record in {vault}", "cross-reference with {doc}"

**Reading:** "read from {vault}", "read {doc} from {vault}", "find in {vault}",
"search vault for", "show me {doc}", "what's in {path} in {vault}",
"list documents in {vault}"

## Setup

Check `vaults.json` in this directory (see `assets/vaults.json.example`). Add vaults as needed:

```json
{
  "vaults": {
    "work": "/home/user/obsidian/work",
    "personal": "/home/user/obsidian/personal"
  }
}
```

## Common: Resolve vault and index

### Resolve vault

From user phrase, match `{vault}` against keys in `vaults.json`. If not found or ambiguous, list available vaults and ask.

### Read vault index

Read `{vault_root}/_vault_index.md` if it exists. Use it to resolve document titles to file paths without asking the user.

---

## Writing: Save a document

### 1. Resolve path

User may supply a relative path inside the vault (e.g., `1-projects/`). If not, ask. Suggest recently used paths per vault if available.

### 2. Resolve filename

User may supply a filename. If not, ask. Strip spaces, use kebab-case with `.md` extension.

### 3. Resolve content

If the user provided content in the conversation, use it. If not, ask. If `--structured` is indicated, use the template below. Otherwise write raw content as-is.

### 4. Check for cross-references

If the user mentions a known document (e.g., "cross-reference with the backend readiness doc"), look it up in `_vault_index.md` inside the vault. If found, append a reference line in the new document. If not found, ask for the relative path.

### 5. Write the file

Create the file at `{vault_root}/{path}/{filename}`. Create intermediate directories if needed.

### 6. Update `_vault_index.md`

Append a row to the index table. If `_vault_index.md` doesn't exist at the vault root, create it with headers and this entry.

**Index format:**

```markdown
# Vault Index

Auto-managed by obsidian-vault skill.

| Title | File | Created | Tags |
|---|---|---|---|
| {title} | {relative path from vault root} | {date} | {tags} |
```

### Template (optional, for structured findings)

```
# {title}

**Source:** {source}
**Date:** {date}
**Scope:** {scope}

---

## {first finding heading}

{content}

## Summary

| Finding | Severity | Impact |
|---|---|---|
```

---

## Reading: Retrieve documents

### Read a specific document

1. User says: "read {doc} from {vault}" or "show me {doc}"
2. Try to resolve `{doc}` via:
   - Title match in `_vault_index.md` (fuzzy, case-insensitive)
   - Direct relative path from user (e.g., `1-projects/monetization/notes.md`)
3. Read the file and return the full content inline.
4. If the user asks about a specific section (e.g., "show me the cross-references section"), read the relevant portion by searching for heading anchors.

### Search vault for content

1. User says: "find {query} in {vault}" or "search vault for {query}"
2. Run content search across `{vault_root}` (using grep or the search tool).
3. Return matching file paths and the relevant snippet around each match.
4. Scope search to markdown files by default. Respect `_vault_index.md` boundaries when the user asks about specific known documents.

### List documents in a path

1. User says: "what's in {path} in {vault}" or "list documents in {vault}"
2. List `.md` files under `{vault_root}/{path}`.
3. If no path given and `_vault_index.md` exists, return the index table.
4. If no path and no index, list top-level `.md` files in the vault root.

### Scope output to the question

- If the question is specific ("what does the backend readiness doc say about WebSockets?"), read the document and return the relevant section — not the full document.
- If the question is broad ("show me the garmin mfa doc"), return the full content.
- If the question is comparative ("what do both readiness docs say about state locality?"), read both, extract the relevant parts, and present them together.

Always prioritize conciseness: answer the question directly rather than dumping content.
