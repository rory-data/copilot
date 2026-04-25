---
description: "Performance guidance covering model selection tiers and context window management"
applyTo: "**"
---

# Performance Guidance

## Model Selection Strategy

Choose the model tier appropriate to the task's complexity:

| Tier | Use Case |
|------|----------|
| **Fast / lightweight** | Frequent invocations, worker agents, simple code generation, pair programming |
| **Standard** | Main development work, orchestrating multi-step workflows, complex coding tasks |
| **Deep reasoning** | Complex architectural decisions, research, analysis tasks requiring maximum reasoning |

Default to the standard tier. Step up to deep reasoning only when the problem genuinely requires it.
Step down to fast/lightweight for high-frequency, low-stakes operations to reduce latency and cost.

## Context Window Management

Avoid the last 20% of the context window for:

- Large-scale refactoring spanning multiple files
- Feature implementation touching many modules
- Debugging complex multi-system interactions

These tasks are **lower risk** at higher context usage:

- Single-file edits
- Independent utility creation
- Documentation updates
- Simple, isolated bug fixes

## Complex Task Strategy

For tasks requiring deep reasoning:

1. Use plan mode before implementing
2. Break the problem into phases, each completable within 50% of the context window
3. Use multiple critique rounds to stress-test the plan before writing code
4. Run sub-agents for independent analysis streams (security, performance, correctness) in parallel

## Build Troubleshooting

If the build fails:

1. Read the error message carefully before acting
2. Fix incrementally — address one error at a time
3. Verify after each fix before moving to the next
4. If the same fix has been attempted 3 times without success, stop and reassess the approach
