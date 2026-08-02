---
name: cosmic-python
description: Evaluates a codebase or specific files/modules against the architectural patterns from "Architecture Patterns with Python" (Cosmic Python) — Domain Model, Repository, Service Layer, Unit of Work, and Aggregates. Use whenever the user says "cosmic python", "review architecture", "DDD review", "domain driven design", "check my architecture", "evaluate this layer", "is this good architecture", "does this follow clean architecture", "review my service layer", "check my repository", "review my domain model", "is this anemic", "where is my business logic", "am I violating DIP", "dependency inversion", "ports and adapters", "onion architecture", "hexagonal architecture", or points you at a codebase and asks if it's well-structured. Also trigger when reviewing any Python backend service that has models, repositories, services, or ORM code and the user wants architectural feedback.
argument-hint: "[path to file, module, or whole codebase] [--pattern domain|repository|service|uow|aggregate|all] [--save]"
model: opus
effort: high
---

# /cosmic-python

Architectural review through the lens of *Architecture Patterns with Python* by Harry Percival and Bob Gregory.

The question is never "does it work." The question is: **does this code keep business logic sovereign, dependencies pointing inward, and infrastructure at arm's length?** The patterns in this book exist to solve one problem — the Big Ball of Mud — where domain knowledge scatters across API handlers, business logic entangles with I/O, and the system becomes increasingly fragile. When those patterns are missing or violated, name exactly what's wrong and why it matters.

## Input

`$ARGUMENTS` — what to evaluate.

**Source options:**
- A file path: `src/services/allocation.py`
- A directory: `src/`
- A module name + paste in conversation
- Nothing specified: ask the user what to look at

**Flags:**
- `--pattern <name>` — focus on one pattern: `domain`, `repository`, `service`, `uow`, `aggregate`, or `all` (default: `all`)
- `--save` — write the review to a file
- `--save path/to/file.md` — write to a specific path

## How to think

You are reviewing code against a coherent theory of architecture, not a checklist. Before jumping to findings, form a mental model of how the code is actually structured:

1. **What is the domain?** What real-world problem is this code solving? What are the core business objects?
2. **Where does the business logic live?** Is it in domain objects, or scattered across services, endpoints, and ORM models?
3. **What does the dependency graph look like?** Do infrastructure concerns (database, HTTP, frameworks) leak inward toward domain logic, or are they kept peripheral?
4. **What would it take to test the core logic?** If you'd need a database, HTTP client, or framework to run any test of business logic, something is wrong.

Review the full reference at `references/patterns.md` before writing findings. It contains the complete theory for each pattern.

## Review structure

Always produce output in this format:

---

# Cosmic Python Architecture Review

## What I read
> Brief description of what was examined (files, layers, key classes)

## Overall verdict
> One paragraph: is this architecture sound, partially sound, or problematic — and why. Be direct.

## Findings

For each finding:

### [Pattern name] — [Finding title]
**Severity:** Critical / Warning / Suggestion
**Location:** `path/to/file.py:line` (or class/function name)

What the code is doing. What the pattern requires. Why the gap matters — specifically what breaks, becomes harder, or becomes riskier because of it.

**What to do:** A concrete, actionable fix. Show code if helpful.

---

## What's working well
> Name the things the code gets right. Don't skip this — knowing what to keep is as important as knowing what to change.

## Priority order
> If there are multiple findings, give a ranked list of what to fix first and why.

---

## Calibration rules

- **Do not fabricate violations.** If the code follows a pattern correctly, say so. False positives erode trust.
- **Severity is about blast radius.** Critical = the whole architectural purpose is defeated. Warning = the pattern is partially implemented but has a real gap. Suggestion = stylistic or minor.
- **Be concrete.** "Business logic in the API layer" means nothing without pointing to the exact function and explaining what rule it encodes.
- **Connect to consequences.** Every finding should explain what actually breaks — not just "this violates the pattern" but "this means you can't test the allocation logic without spinning up a database" or "this means a change to SQLAlchemy's session API will ripple into your domain model."
- **Context matters.** A simple CRUD endpoint may not need a full domain model. Don't over-engineer small things. The book itself says: "A big ball of mud is fine for small apps." Flag over-engineering too.

## Pattern quick reference

For the full theory, see `references/patterns.md`. Quick reminders:

| Pattern | Core question to ask |
|---------|---------------------|
| Domain Model | Do entities/value objects encode business rules, or are they just data bags? |
| Repository | Does persistence hide behind a narrow interface? Can you swap storage without touching domain code? |
| Service Layer | Is orchestration separated from business rules and from HTTP/framework concerns? |
| Unit of Work | Are atomic operations explicit? Does the service layer control transaction boundaries? |
| Aggregates | Are consistency boundaries explicit? Is there one repository per aggregate? |
| Abstractions/DIP | Do high-level modules depend on abstractions, not implementations? Do dependencies point inward? |
| Testing | Can business logic be tested without infrastructure? Is the test pyramid healthy? |
