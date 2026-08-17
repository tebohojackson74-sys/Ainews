# AINews Project Review & Improvement Plan

**Review date:** 17 August 2026  
**Scope:** repository, editorial foundation, governance and launch readiness

## Current state

AINews has a strong editorial foundation: a South Africa-first mission, newsroom roles, article templates, a 30-day content calendar, a sample morning brief, a healthcare case study, and a reproducibility starter structure. The repository is currently an editorial/prototyping repository rather than a production news application.

## Strengths

- Clear South Africa-first positioning.
- Human editor sign-off is already a core principle.
- Primary-source and reproducibility expectations are documented.
- Editorial roles and article templates give contributors a common operating model.
- The repository already contains reproducibility scaffolding and a content calendar.
- The roadmap is staged rather than attempting regional expansion immediately.

## Weaknesses identified

### 1. Product scope is broader than the current repository
The roadmap describes CMS, AI orchestration, authentication, sports data, multimedia, multilingual publishing and subscriptions, but the repository contains mostly documentation and content assets. The next engineering phase needs an explicit MVP boundary.

### 2. Editorial governance is not yet operationalised
Human sign-off is required, but CODEOWNERS currently contains placeholder teams and GitHub branch protection rules have not been documented as launch prerequisites. A policy must define who can approve breaking, health, legal, investigative and AI-generated stories.

### 3. AI governance needs stronger controls
The project needs explicit rules for source attribution, hallucination handling, model/version logging, AI disclosure, prompt/data privacy, and prohibition of autonomous publication.

### 4. Corrections and provenance need structured records
The current policy mentions corrections, but the future CMS should model corrections, source records, claim verification and article revisions as first-class entities rather than relying only on prose or links.

### 5. Sports is under-specified
Sports was identified as a launch focus, but the current repository does not yet define authoritative data providers, competition taxonomy, women's-football parity, match-data freshness, transfer verification or injury/suspension sourcing.

### 6. Security and privacy controls are incomplete
There is no visible security reporting policy, secret-management guidance, data classification standard, dependency/update policy or documented incident response procedure.

### 7. No automated quality gate is defined
There is no CI workflow enforcing Markdown checks, notebook integrity, secret scanning, link validation or required editorial metadata.

## Improvements in this hardening pass

- Added this review as a living project-risk register.
- Added an editorial and AI governance policy.
- Added security reporting guidance.
- Added GitHub issue templates for story proposals and engineering work.
- Added explicit MVP boundaries and launch gates to the governance documentation.
- Updated CODEOWNERS to use the current repository owner as an interim required owner instead of placeholders.

## Next priority

**Do not build every planned feature at once.** The next implementation milestone should be a newsroom MVP that can reliably move one story from intake → research → verification → editor approval → publication → correction/audit trail.

Sports data, subscriptions, mobile apps, automated multimedia and international expansion should remain behind explicit feature gates until that core workflow is proven.

## Launch gates

Before public launch, AINews should have:

- [ ] Protected default branch with required review.
- [ ] Working CI checks.
- [ ] Security reporting process.
- [ ] Editorial/AI governance policy approved.
- [ ] Source and claim provenance model.
- [ ] Correction and revision model.
- [ ] CMS editorial workflow implemented.
- [ ] No autonomous AI publication path.
- [ ] Primary-source verification for high-risk stories.
- [ ] Sports data-source policy before live sports claims are published.
- [ ] Backup and incident-response procedure.
