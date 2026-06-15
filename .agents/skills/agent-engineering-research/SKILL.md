---
name: agent-engineering-research
description: Use when absorbing new agent-engineering sources, discussing agent-building practice, turning failures into methodology, or updating this lab with structured notes, templates, eval ideas, or framework backlog items.
---

# Agent Engineering Research

Use this skill inside `agent-engineering-lab` when the user brings a new article, official doc, project issue, implementation experience, or hypothesis about building agents.

## Goal

Turn raw input into durable project knowledge without overgeneralizing.

## Required Context

Read these files first:

1. `README.md`
2. `AGENTS.md`
3. `01_sources/sources.md`
4. `02_agent_engineering_methodology/agent-system-model.md`
5. `03_openai_agent_practice/README.md`

## Source Handling

Classify each input:

- `official`: OpenAI official docs, cookbook, or blog.
- `external`: third-party article, paper, repo, video, or talk.
- `practice`: user's own implementation note, failure, debugging experience, or project decision.
- `hypothesis`: an idea not yet validated.

For OpenAI product/API claims, verify with current official OpenAI docs when the claim could have changed.

## Extraction Flow

1. Register the source in `01_sources/sources.md`.
2. Split the content into:
   - facts from the source,
   - author's opinion,
   - user's or assistant's interpretation,
   - hypotheses to validate,
   - engineering implications.
3. Map each implication to one or more surfaces:
   - agent definition,
   - tool contract,
   - context and state,
   - orchestration and handoffs,
   - guardrails and human review,
   - structured output,
   - tracing and observability,
   - evals,
   - production readiness,
   - product UX.
4. Update the relevant file in:
   - `02_agent_engineering_methodology/`,
   - `03_openai_agent_practice/`,
   - `04_templates/`,
   - `05_cases/`,
   - `06_experiments/`,
   - `07_framework_roadmap/`.

## Output Requirements

When done, summarize:

- what source or practice was added,
- what methodology changed,
- what remains a hypothesis,
- what should be validated in a real agent project.

Do not present external-source claims as universal rules unless they have official support or practice validation.
