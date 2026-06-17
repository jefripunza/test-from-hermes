---
name: refactor-pro
description: "Code refactoring expert: SOLID, DRY, KISS, YAGNI. Detect code smells, dead code, tight coupling, duplicate logic. Refactor with measurable improvement."
metadata:
  hermes:
    tags: [refactoring, solid, clean-code, code-quality, technical-debt]
---

# Refactor Pro — Code Refactoring Expert

Apply SOLID, DRY, KISS, YAGNI. Refactor only for **measurable improvement**.

## Principles

- **SOLID** — Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
- **DRY** — Don't Repeat Yourself (every piece of knowledge has one representation)
- **KISS** — Keep It Simple, Stupid (simpler is better)
- **YAGNI** — You Ain't Gonna Need It (don't add speculation)

## Detect

- **Code smells** — long methods, large classes, feature envy, shotgun surgery
- **Dead code** — unused functions, imports, variables, unreachable branches
- **Tight coupling** — hard dependencies between modules, circular deps
- **Large functions** — >20 lines, multiple levels of abstraction, too many params
- **Duplicate logic** — copy-paste code, similar conditional blocks
- **God objects** — classes/modules doing too much
- **Primitive obsession** — using primitives instead of value objects
- **Switch/if-else chains** — polymorphism candidate
- **Temporary fields** — class fields only set in some code paths

## Output Format

```text
Problem: What's wrong
Impact: Why it matters (performance, maintainability, bugs)
Refactor Plan: Step-by-step to fix
Benefits: Measurable improvement after refactor
```

## Rules

- **Never refactor solely for aesthetics** — must have measurable benefit
- **Refactor only when there is measurable improvement** — reduced complexity, better perf, fewer bugs
- **One refactor per commit** — atomic, reversible
- **Tests before refactor** — ensure behavior unchanged
- **Leave code cleaner than you found it** (boy scout rule)

## Refactoring Techniques

| Technique | When | How |
|-----------|------|-----|
| Extract Method | Long function | Break into focused methods |
| Extract Class | God object | Split responsibilities |
| Replace Conditional with Polymorphism | Switch chains | Interface + implementations |
| Introduce Parameter Object | Many params | Wrap in typed struct/class |
| Decompose Conditional | Complex if-else | Extract boolean conditions |
| Pull Up / Push Down | Wrong hierarchy | Move to correct level |
| Replace Magic Number with Constant | Literal values | Named constant |
| Encapsulate Field | Direct field access | Getter/setter |
| Remove Dead Code | Unused paths | Delete |
| Rename | Unclear names | Descriptive naming |
