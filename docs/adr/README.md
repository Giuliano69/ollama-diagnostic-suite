# Architecture Decision Records (ADR)

This directory contains the project’s Architecture Decision Records (ADR).

An ADR documents an important technical decision, together with the context, alternatives considered and expected consequences.

The goal is to preserve the reasoning behind architectural choices, making future maintenance and refactoring easier.

## ADR Lifecycle

Each ADR is assigned one of the following states:

- **GREEN** — Accepted and currently valid.

- **YELLOW** — Under review or partially superseded.

- **RED** — Deprecated or replaced by a newer decision.

Older ADRs are never deleted. If a decision changes, a new ADR supersedes the previous one while preserving the project’s historical context.

## Naming Convention

ADR-001-short-title.md  
ADR-002-short-title.md  
ADR-003-short-title.md

## Template

\# ADR-00X: Short Title  
  
\*\*Status:\*\* GREEN  
\*\*Date:\*\* YYYY-MM-DD  
\*\*Author:\*\* Project Maintainer  
  
\#\# 1. Context / Problem  
  
Describe the problem that required a decision.  
  
\#\# 2. Decision  
  
Describe the adopted solution.  
  
\#\# 3. Alternatives Considered  
  
List the main alternatives and explain why they were rejected.  
  
\#\# 4. Consequences  
  
Positive and negative consequences of the decision.  
  
\#\# 5. Review Conditions  
  
When should this ADR be reconsidered?  
  
\#\# 6. Related ADRs  
  
References to related Architecture Decision Records.
