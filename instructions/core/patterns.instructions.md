---
description: "Common utility patterns for API responses, repository abstraction, and utilities. For Gang of Four design patterns (Factory, Observer, Strategy, Decorator, etc.), invoke the gang-of-four skill."
applyTo: "**"
---

# Common Patterns

> For GoF design patterns (creational, structural, behavioural), invoke the **gang-of-four** skill.
> Code examples use Python for illustration. For Python-specific conventions, invoke the python-conventions skill.

## API Response Format

Use a consistent response envelope for all API responses with these fields: `success` (bool),
`data` (nullable payload), `error` (nullable message), and optional `meta` (pagination: `total`,
`page`, `limit`). For Python implementations see the `python-conventions` skill.

## Repository Pattern

Abstract data access behind a typed interface with these operations: `find_all(filters)`,
`find_by_id(id)`, `create(data)`, `update(id, data)`, `delete(id)`. Inject the repository as a
dependency — never instantiate it directly. For Python implementations see the `python-conventions`
skill.

## Debounce / Throttle Utility

A debounce utility cancels any pending invocation when called again within the delay window,
then executes after the delay elapses. Use for search inputs, resize handlers, and any event that
fires faster than you need to react. For Python async implementations see the `python-conventions`
skill.

## Skeleton Projects

When implementing new functionality:

1. Search for battle-tested starter projects or libraries
2. Evaluate options across multiple dimensions:
   - Security posture
   - Extensibility
   - Relevance to the problem
   - Maintenance activity
3. Clone or install the best match as a foundation
4. Iterate within the proven structure
