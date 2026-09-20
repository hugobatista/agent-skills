---
name: ste100
description: 'Use when writing or editing technical documentation (manuals, procedures, work instructions, warnings, cautions, notes, maintenance guides, user guides). Applies ASD-STE100 Simplified Technical English Issue 9 rules.'
author: hugobatista
---

Apply ASD-STE100 Simplified Technical English Issue 9 rules to technical documentation.

Scope: manuals, procedures, work instructions, warnings, cautions, notes, maintenance guides, user guides. For Markdown formatting, load `markdown-style` as well.

## Rules

- **Classify text type**: Procedural = step-by-step commands in imperative mood.
  Descriptive = explanations in present tense, active voice.
- **Sentences**: Max 20 words (procedural) / 25 words (descriptive).
  One topic per sentence.
- **Voice**: Active only. Exception: passive allowed in descriptive writing
  when the receiver of the action is the topic.
- **Verb forms**: Imperative for procedures. Present tense for descriptions.
  No "-ing" forms unless part of an approved noun.
- **Vocabulary**: Each word has one meaning, one part of speech. No synonyms.
  Use approved words only (start ✓, begin/commence ✗). Define technical jargon
  on first use.
- **Noun clusters**: Max 3 nouns in sequence. "API rate limit" ✓,
  "User authentication token validator" ✗ → "Token validator for user authentication"
- **Articles**: Use a/an/the consistently. Do not omit.
- **Lists**: Each procedural step is one action in imperative mood.
