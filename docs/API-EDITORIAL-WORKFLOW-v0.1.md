# AINews API & Editorial Workflow v0.1

## 1. Purpose

Define the API contract and state machine for the AINews MVP newsroom.

Core rule: **the server enforces editorial authority. The client UI is never trusted to enforce publication permissions.**

## 2. API conventions

- Base path: `/api/v1`
- JSON request/response bodies.
- UUIDs for resource identifiers.
- UTC timestamps in ISO 8601 format.
- Cursor pagination for collections where practical.
- Standard error envelope:

```json
{
  "error": {
    "code": "EDITOR_APPROVAL_REQUIRED",
    "message": "An approved revision is required before publication.",
    "request_id": "..."
  }
}
```

- Authentication: secure session or short-lived access token; refresh mechanism must be protected.
- Authorization: role and resource checks performed server-side on every mutation.

## 3. Resource groups

### Auth
- `POST /auth/login`
- `POST /auth/logout`
- `POST /auth/refresh`
- `GET /auth/me`

### Stories
- `POST /stories`
- `GET /stories`
- `GET /stories/{storyId}`
- `PATCH /stories/{storyId}`
- `POST /stories/{storyId}/assign`
- `POST /stories/{storyId}/submit-review`
- `POST /stories/{storyId}/request-changes`
- `POST /stories/{storyId}/approve`
- `POST /stories/{storyId}/publish`
- `POST /stories/{storyId}/corrections`
- `GET /stories/{storyId}/revisions`

### Sources
- `POST /stories/{storyId}/sources`
- `GET /stories/{storyId}/sources`
- `DELETE /stories/{storyId}/sources/{sourceId}`

### Claims / evidence
- `POST /stories/{storyId}/claims`
- `PATCH /claims/{claimId}`
- `POST /claims/{claimId}/evidence`
- `POST /claims/{claimId}/verify`

### Reviews
- `GET /reviews`
- `GET /reviews/{reviewId}`
- `POST /reviews/{reviewId}/approve`
- `POST /reviews/{reviewId}/request-changes`
- `POST /reviews/{reviewId}/reject`

### AI assistance
- `POST /ai/tasks`
- `GET /ai/tasks/{taskId}`
- `GET /stories/{storyId}/ai-tasks`
- `POST /ai/outputs/{outputId}/accept`

### Media
- `POST /media`
- `GET /media/{mediaId}`
- `POST /stories/{storyId}/media`
- `DELETE /stories/{storyId}/media/{mediaId}`

### Audit
- `GET /audit/events`
- `GET /stories/{storyId}/audit`

### Public
- `GET /public/stories/{slug}`
- `GET /public/sections/{slug}`
- `GET /public/search`

## 4. Story state machine

```text
IDEA
  ↓ assign
ASSIGNED
  ↓ begin research
RESEARCH
  ↓ save draft
DRAFT
  ↓ submit
VERIFICATION
  ↓ submit for editor review
REVIEW
  ├── request changes → DRAFT
  ├── reject → DRAFT / ARCHIVED
  └── approve → APPROVED
                     ↓ publish
                  PUBLISHED
                     ↓
              UPDATED / CORRECTED
```

### Transition rules

| From | Action | To | Authority |
|---|---|---|---|
| IDEA | assign | ASSIGNED | Editor/Admin |
| ASSIGNED | begin | RESEARCH | Reporter/Editor |
| RESEARCH | draft | DRAFT | Reporter/Editor |
| DRAFT | submit | VERIFICATION | Reporter/Editor |
| VERIFICATION | submit review | REVIEW | Reporter/Editor |
| REVIEW | request changes | DRAFT | Editor |
| REVIEW | reject | DRAFT/ARCHIVED | Editor |
| REVIEW | approve | APPROVED | Editor |
| APPROVED | publish | PUBLISHED | Editor/Admin |
| PUBLISHED | update | UPDATED | Editor |
| PUBLISHED | correct | CORRECTED | Editor/Admin |

The API must reject invalid transitions even if a caller attempts to construct the request manually.

## 5. Publication gate

`POST /stories/{storyId}/publish` must verify all of the following server-side:

1. Caller has Editor or Admin authority.
2. Story status is `APPROVED`.
3. An approval exists for the exact revision being published.
4. Required claims have acceptable verification status.
5. High/critical-risk claims have human verification.
6. Required sources are attached.
7. No unresolved blocking editorial review exists.
8. Publication metadata is valid.

If any check fails, publication is rejected.

## 6. Verification workflow

A reporter can attach sources and create claims. A verifier/editor can mark claims:

- `supported`
- `disputed`
- `rejected`
- `not_applicable`

A claim cannot become `supported` merely because an AI output supports it.

For high-risk and critical claims, `verified_by` and `verified_at` are mandatory.

Examples of high-risk areas:
- healthcare/medical claims
- allegations about identifiable people or organisations
- legal/criminal claims
- financial claims with material consequences
- breaking claims where misinformation risk is high

## 7. AI workflow

```text
User requests AI task
        ↓
POST /ai/tasks
        ↓
AI worker executes
        ↓
ai_task + ai_output recorded
        ↓
Human reviews output
        ↓
Optional accept into draft/research workspace
        ↓
Normal verification + editor approval
```

AI workers do not receive publication authority.

## 8. Idempotency and concurrency

Mutation endpoints that may be retried should accept an idempotency key.

Publication must use a transaction/lock strategy so two editors cannot publish competing revisions simultaneously.

Revision numbers must be unique per story.

## 9. Audit requirements

Record audit events for:
- login/security-sensitive actions where appropriate
- story creation
- assignment
- status transitions
- source changes
- claim verification
- AI task creation/completion/acceptance
- editor approval/rejection
- publication
- corrections
- role/permission changes

Audit events are append-only and not editable through the application UI.

## 10. Security requirements

- Validate and authorize every mutation server-side.
- Rate-limit authentication and expensive AI endpoints.
- Validate uploaded media types and sizes.
- Sanitize rendered rich text to prevent stored XSS.
- Protect secrets in server-side environment configuration only.
- Never expose provider API keys to browsers.
- Restrict audit log access.
- Minimize personal data collection.
- Log request IDs for incident investigation without logging sensitive payloads.

## 11. Public API rules

Public article responses should expose only publication-safe data.

Do not expose:
- internal audit events
- private source notes
- unpublished revisions
- internal prompts
- raw AI task metadata that could expose secrets or private data
- reporter/editor private information

Where editorially appropriate, expose a concise source/provenance section rather than internal evidence records.

## 12. Sports API extension

Sports endpoints should be introduced after the core newsroom API is stable:

- `/sports/competitions`
- `/sports/seasons`
- `/sports/teams`
- `/sports/players`
- `/sports/matches`
- `/sports/transfers`
- `/sports/injuries`
- `/sports/suspensions`
- `/sports/standings`

Sports facts should include source, retrieved timestamp and confidence/status metadata. Transfer rumours must never be represented as confirmed transfers.

## 13. API testing priorities

Automated tests must cover:

1. Reporter cannot publish.
2. Editor cannot publish an unapproved revision.
3. High-risk claims cannot bypass human verification.
4. AI output cannot satisfy an approval requirement.
5. Published revisions cannot be silently overwritten.
6. Corrections retain historical revision data.
7. Unauthorized users cannot access private newsroom resources.
8. Invalid state transitions return errors.
9. Duplicate publication requests are safely handled.
10. Audit events are created for critical actions.

## 14. Implementation sequence

1. Auth/session middleware.
2. RBAC authorization middleware.
3. Story/revision endpoints.
4. Source and claim/evidence endpoints.
5. Review and approval endpoints.
6. Transactional publication endpoint.
7. Corrections/audit endpoints.
8. AI task queue and worker interface.
9. Public read API.
10. Sports extension.

## 15. Definition of Done

A test reporter can create and research a story, attach evidence, submit it for review, have an editor approve the exact revision, publish it through the API, and later issue a correction—while unauthorized operations fail and the complete critical workflow remains auditable.
