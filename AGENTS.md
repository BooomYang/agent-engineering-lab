# AGENTS.md

This repository is a knowledge lab for agent engineering. Treat it as a long-running research and practice workspace, not as an application codebase.

## Project Intent

The goal is to build a durable methodology for designing, building, evaluating, and operating AI agents. Keep the distinction clear:

- `agent-engineering-lab`: full agent system design and operation.
- `loop-engineering-lab`: feedback-loop engineering, especially around AI coding agents.

Cross-link when useful, but do not collapse the two projects into one.

## Source Policy

- OpenAI product or API claims must be checked against current official OpenAI docs when they could have changed.
- First-party sources include `developers.openai.com`, `platform.openai.com`, official OpenAI cookbook pages, and official OpenAI GitHub links referenced from those pages.
- Third-party articles, papers, videos, and repos must be marked as external sources.
- Personal experience must be marked as practice evidence.
- Do not convert one article or one debugging story into a rule without labeling it as a hypothesis or validating it through practice.

## Writing Style

- Write in Chinese by default.
- Prefer compact, operational notes over abstract essays.
- Separate facts, interpretation, implications, and open questions.
- Keep links close to the claims they support.
- Use checklists and templates when they make future reuse easier.

## Update Flow

When processing a new source or practice note:

1. Register it in `01_sources/sources.md`.
2. Summarize the source using `01_sources/article-note-template.md` when it is article-like.
3. Map the insight to one or more engineering surfaces:
   - agent definition
   - tool contract
   - context and state
   - orchestration and handoffs
   - guardrails and human review
   - structured output
   - tracing and observability
   - evals
   - production readiness
   - product UX
4. Update the relevant methodology or practice file.
5. If the insight is reusable, add or update a template in `04_templates/`.
6. If it implies a future system capability, add it to `07_framework_roadmap/backlog.md`.

## Editing Guardrails

- Keep edits scoped to this project unless the user explicitly asks to update another project.
- Do not rewrite existing notes wholesale when an incremental update is enough.
- Preserve uncertainty markers such as `待验证`, `假设`, and `实践观察`.
- When official docs conflict with an external article, keep the official docs as the baseline and record the conflict.

## Verification

For documentation-only changes, verify by checking:

- directory links resolve locally,
- source URLs are present where major claims depend on them,
- new templates have clear fields for future use,
- no OpenAI-specific claim is presented as timeless if it may change.
