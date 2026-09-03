# Repo Onboarding Questions — Canonical Checklist

Universal questions any engineer would ask about any repo. Used by the repo-cartographer's newcomer simulation (Phase 3) to verify context file completeness.

Versioned centrally in this skill. Repo-specific questions are appended per-repo during enrichment sessions.

## Verdict rubric

Each question receives one of five verdicts: `ANSWERED-WITH-CITATION`, `ANSWERED-NO-CITATION`, `PARTIAL`, `MISLEADING`, `MISSING`. See `knowledge/newcomer-rubric.md` for definitions, decision rules, citation format, and per-category scoring weights.

When grading, record:

- The verdict
- For `ANSWERED-WITH-CITATION`: a `source: <file>:<heading-or-line>` reference
- For `ANSWERED-NO-CITATION`: a one-line note explaining where the answer is and why no citation was recorded
- For `PARTIAL` or `MISLEADING`: a one-line note explaining what's wrong or missing
- For `MISSING`: a one-line note confirming no draft file addresses the question

## Getting Started

- [ ] How do I clone and set up local dev for this repo?
- [ ] What prerequisites do I need installed (Node version, package manager, Docker, etc.)?
- [ ] How do I run the tests?
- [ ] How do I run the linter and type checker?
- [ ] How do I start the service locally?

## Understanding the System

- [ ] What does this service/library do in one paragraph?
- [ ] What are the main components and how do they relate?
- [ ] What's the request/event lifecycle for the primary operation?
- [ ] What external services does this depend on (databases, queues, APIs)?
- [ ] What are the key domain concepts and their states/lifecycle?

## Working Safely

- [ ] What are the hard constraints I must follow (things that break if violated)?
- [ ] Which files or directories are sensitive and need extra caution?
- [ ] What should I use as a template when adding a new [entity/component/handler]?
- [ ] What should I watch out for if I'm changing a high-traffic area?

## Operating in Production

- [ ] How do I deploy, and how do I roll back?
- [ ] What happens if an external dependency goes down?
- [ ] What are the known failure modes?
- [ ] Where do I look when something is broken in prod (dashboards, logs, alerts)?
- [ ] What does an alert on [primary alarm] mean and what do I do about it?

## Context & History

- [ ] Why were key technical choices made (language, framework, data store)?
- [ ] Are there known tech debt items or things the team would do differently today?
- [ ] What are the cross-repo relationships (upstream producers, downstream consumers)?
