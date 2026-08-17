# AINews Database Schema v0.1

## Purpose

This schema supports the MVP workflow:

`Story → Sources → Claims/Evidence → Revisions → Review → Publication → Corrections`

It also records AI assistance without allowing AI to become an editorial authority.

## Design principles

1. PostgreSQL is the source of truth.
2. UUIDs are used for externally exposed primary identifiers.
3. Published journalism is immutable at the revision level; corrections create new revisions/records rather than destroying history.
4. Every important editorial action is auditable.
5. Sources and evidence are first-class records, not text blobs hidden inside articles.
6. AI outputs are retained as assistance artifacts with model/provider metadata.
7. Soft deletion is preferred for editorial records; published content is never hard-deleted through the normal UI.
8. Timestamps are stored in UTC.

---

## 1. users

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | User identifier |
| email | citext UNIQUE | Login identity |
| display_name | text | Byline/account name |
| status | enum | active, suspended, invited |
| created_at | timestamptz | |
| updated_at | timestamptz | |

## 2. roles

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | |
| key | text UNIQUE | reader, reporter, editor, admin |
| name | text | Human-readable name |

## 3. user_roles

| Column | Type | Notes |
|---|---|---|
| user_id | uuid FK users | |
| role_id | uuid FK roles | Composite PK |

---

# Editorial domain

## 4. desks

Represents editorial sections.

Fields: `id`, `slug`, `name`, `description`, `active`, `created_at`.

Initial desks:
- ai-technology
- business-economy
- healthcare
- politics-public-policy
- science-research
- football-sport

## 5. stories

The stable identity of a piece of journalism. Content itself is versioned in `story_revisions`.

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | Stable story ID |
| desk_id | uuid FK desks | |
| author_id | uuid FK users | Primary author |
| status | enum | idea, assigned, research, draft, verification, review, approved, published, corrected, archived |
| slug | text UNIQUE | Public URL slug |
| created_at | timestamptz | |
| updated_at | timestamptz | |
| published_at | timestamptz NULL | First publication time |
| current_revision_id | uuid NULL | FK story_revisions; set after revision exists |

## 6. story_revisions

Immutable versions of story content.

Fields:
- `id` uuid PK
- `story_id` uuid FK stories
- `revision_number` integer
- `title` text
- `standfirst` text
- `body` jsonb
- `change_summary` text
- `created_by` uuid FK users
- `created_at` timestamptz

Unique: `(story_id, revision_number)`.

## 7. story_assignments

Fields: `id`, `story_id`, `assigned_to`, `assigned_by`, `due_at`, `created_at`, `completed_at`.

## 8. story_tags

Fields: `story_id`, `tag_id`.

## 9. tags

Fields: `id`, `slug`, `name`, `created_at`.

---

# Sources and evidence

## 10. sources

A reusable source record.

Fields:
- `id` uuid PK
- `source_type` enum: paper, dataset, government, company, interview, court, filing, social_post, media, other
- `publisher` text
- `title` text
- `url` text NULL
- `document_identifier` text NULL
- `author` text NULL
- `published_at` timestamptz NULL
- `retrieved_at` timestamptz
- `reliability_notes` text NULL
- `created_by` uuid FK users
- `created_at` timestamptz

## 11. story_sources

Joins stories to sources.

Fields: `story_id`, `source_id`, `relationship` enum (primary, supporting, background), `notes`, `added_by`, `created_at`.

## 12. claims

A claim is a factual assertion in a story that may require verification.

Fields:
- `id` uuid PK
- `story_id` uuid FK stories
- `revision_id` uuid FK story_revisions
- `claim_text` text
- `risk_level` enum: low, medium, high, critical
- `verification_status` enum: unverified, supported, disputed, rejected, not_applicable
- `verified_by` uuid FK users NULL
- `verified_at` timestamptz NULL
- `notes` text NULL

## 13. claim_evidence

Links a claim to the source/evidence supporting it.

Fields:
- `id` uuid PK
- `claim_id` uuid FK claims
- `source_id` uuid FK sources
- `evidence_quote` text NULL
- `evidence_locator` text NULL
- `relationship` enum: supports, contradicts, context
- `added_by` uuid FK users
- `created_at` timestamptz

---

# Editorial review

## 14. review_requests

Fields: `id`, `story_id`, `revision_id`, `requested_by`, `assigned_editor_id`, `status`, `notes`, `requested_at`, `completed_at`.

Status: pending, changes_requested, approved, rejected.

## 15. editorial_approvals

Immutable approval record.

Fields:
- `id` uuid PK
- `story_id` uuid FK stories
- `revision_id` uuid FK story_revisions
- `editor_id` uuid FK users
- `decision` enum: approved, rejected
- `notes` text NULL
- `created_at` timestamptz

A publication operation must reference an approval for the same revision.

## 16. corrections

Fields:
- `id` uuid PK
- `story_id` uuid FK stories
- `revision_id` uuid FK story_revisions
- `type` enum: correction, clarification, update, retraction
- `description` text
- `created_by` uuid FK users
- `approved_by` uuid FK users
- `published_at` timestamptz

---

# AI assistance

## 17. ai_tasks

Represents a requested AI operation.

Fields:
- `id` uuid PK
- `story_id` uuid FK stories NULL
- `requested_by` uuid FK users
- `task_type` enum: research, summarise, extract_claims, draft, headline, seo, translate, classify
- `provider` text
- `model` text
- `status` enum: queued, running, completed, failed
- `created_at` timestamptz
- `completed_at` timestamptz NULL

## 18. ai_outputs

Fields:
- `id` uuid PK
- `ai_task_id` uuid FK ai_tasks
- `input_reference` jsonb
- `output` jsonb
- `created_at` timestamptz
- `accepted_by` uuid FK users NULL
- `accepted_at` timestamptz NULL

AI output is never considered verified solely because it exists here.

---

# Media

## 19. media_assets

Fields: `id`, `type` enum (image, video, audio, document), `storage_key`, `mime_type`, `alt_text`, `caption`, `credit`, `license`, `created_by`, `created_at`.

## 20. story_media

Fields: `story_id`, `media_id`, `position`, `role`.

---

# Audit

## 21. audit_events

Append-only audit trail.

Fields:
- `id` uuid PK
- `actor_user_id` uuid FK users NULL
- `entity_type` text
- `entity_id` uuid
- `action` text
- `before_state` jsonb NULL
- `after_state` jsonb NULL
- `metadata` jsonb NULL
- `ip_hash` text NULL
- `created_at` timestamptz

Do not store passwords, access tokens or unnecessary personal data in audit metadata.

---

# Public engagement (minimal MVP)

## 22. article_views

Fields: `id`, `story_id`, `session_id`, `occurred_at`.

Use aggregated analytics where possible; avoid storing unnecessary identifiable browsing data.

---

# Sports extension — schema reserved in v0.1

The sports model should be separate from the editorial story model but linkable to it.

Planned tables:
- competitions
- seasons
- teams
- players
- coaches
- matches
- match_events
- standings
- player_team_memberships
- injuries
- suspensions
- transfers

Sports facts should carry source/timestamp metadata and a `status` such as confirmed, reported, disputed or historical where appropriate.

Men's and women's football share the entity model but competitions remain independently modelled.

---

# Key relationships

```text
USER ──< STORY
DESK ──< STORY
STORY ──< STORY_REVISION
STORY ──< STORY_SOURCE >── SOURCE
STORY_REVISION ──< CLAIM ──< CLAIM_EVIDENCE >── SOURCE
STORY ──< REVIEW_REQUEST
STORY ──< EDITORIAL_APPROVAL
STORY ──< CORRECTION
STORY ──< AI_TASK ──< AI_OUTPUT
STORY ──< STORY_MEDIA >── MEDIA_ASSET
ALL IMPORTANT ENTITIES ──< AUDIT_EVENT
```

# Critical constraints

1. Only authorised editors/admins may approve publication.
2. A published revision cannot be silently overwritten.
3. A correction must point to the affected published revision.
4. High-risk claims cannot be marked supported without a human verifier.
5. AI outputs cannot satisfy a verification requirement by themselves.
6. Source records must remain linked to the story revision/claims they support.
7. Audit events are append-only.
8. User permissions are enforced server-side.

# Recommended indexes

- `stories(status, updated_at)`
- `stories(desk_id, published_at)`
- `stories(slug)` unique
- `story_revisions(story_id, revision_number)` unique
- `claims(story_id, verification_status)`
- `claim_evidence(claim_id)`
- `sources(published_at)`
- `review_requests(status, assigned_editor_id)`
- `audit_events(entity_type, entity_id, created_at)`
- full-text/search index on published story title/body

# Migration strategy

Implement schema changes through versioned migrations. Never edit production schema manually. Seed only roles, desks and other non-sensitive reference data.
