# AINews MVP v0.1

## 1. Product goal

Build the smallest trustworthy South Africa-first AI newsroom that can move a story from intake to publication with evidence, human editorial approval, provenance, revisions and corrections.

The MVP is a newsroom product first and an AI automation product second.

## 2. Primary users

### Reader
- Browse published stories.
- Search and filter stories.
- View article source/provenance information where published.

### Reporter
- Create story pitches and drafts.
- Add sources and evidence.
- Submit stories for review.
- Respond to editor requests.

### Editor
- Assign stories.
- Review drafts and evidence.
- Request changes.
- Approve or reject publication.
- Publish and correct stories.

### Administrator
- Manage users and roles.
- Review audit logs.
- Configure system settings.

## 3. MVP editorial workflow

`Idea → Assigned → Research → Draft → Verification → Editor Review → Approved → Published → Updated/Corrected`

A story cannot enter `Published` unless an authorised editor approves it.

AI output is always an artifact attached to a story or research task. It is never an approval authority.

## 4. Core screens

### Public
1. Homepage
2. Section page
3. Article page
4. Search results
5. Author page

### Newsroom
1. Login
2. Newsroom dashboard
3. Story queue
4. Story editor
5. Sources & evidence panel
6. Verification panel
7. Review/approval panel
8. Publication scheduler
9. Corrections/revision history

### Admin
1. Users
2. Roles/permissions
3. Audit log
4. System settings

## 5. MVP story model

Each story must support:
- title
- standfirst
- body
- author
- desk/category
- status
- created/updated/published timestamps
- sources
- claims/evidence
- revisions
- editor approvals
- correction records
- AI assistance records, if used
- media attachments, if used

## 6. Source and provenance model

A source record should contain:
- source type
- publisher/organisation
- title
- URL or document identifier
- publication date
- retrieval date
- author, where known
- source reliability notes
- relevant excerpt or evidence pointer

High-risk stories should support claim-level evidence rather than only an article-level source list.

## 7. Roles and permissions

| Capability | Reader | Reporter | Editor | Admin |
|---|---:|---:|---:|---:|
| Read published stories | Yes | Yes | Yes | Yes |
| Create story | No | Yes | Yes | Yes |
| Edit own draft | No | Yes | Yes | Yes |
| Review other stories | No | No | Yes | Yes |
| Approve publication | No | No | Yes | Yes |
| Publish | No | No | Yes | Yes |
| Correct published story | No | No | Yes | Yes |
| Manage users | No | No | No | Yes |
| View audit log | No | No | Limited | Yes |

## 8. AI capabilities in v0.1

Only assistive features are included:
- research summarisation
- source extraction
- claim extraction
- draft suggestions
- headline suggestions
- SEO metadata suggestions
- translation draft suggestions

Every AI action must record:
- model/provider
- prompt/task type
- timestamp
- input reference
- output
- user who requested it

No autonomous publishing.

## 9. South Africa-first launch desks

Initial editorial desks:
- AI & Technology
- Business & Economy
- Healthcare
- Politics & Public Policy
- Science & Research
- Football/Sport

The sports desk starts with South African football and supports men's and women's competitions.

## 10. Explicit non-goals

Not part of v0.1:
- mobile apps
- subscriptions/paywalls
- advertising platform
- personalised recommendation engine
- automated video production
- podcast automation
- international editions
- broad global sports data
- autonomous AI publishing

## 11. MVP success criteria

A complete production test must demonstrate:

1. Reporter creates a story.
2. Reporter attaches at least two sources.
3. Reporter records evidence for important claims.
4. AI may assist research/drafting.
5. Editor sees the evidence and AI assistance history.
6. Editor requests changes or approves.
7. Approved article is published publicly.
8. Published article retains author/editor/source/revision metadata.
9. Editor can issue a correction without destroying the previous revision.
10. Audit log records the important workflow actions.

## 12. Quality gates

Before MVP release:
- Authentication and authorization tests pass.
- Publication permission is tested server-side.
- Correction workflow is tested.
- Audit logging is tested.
- Source/provenance records survive article revisions.
- AI outputs are never silently inserted as verified facts.
- Critical editorial workflows have automated tests.
- Security review is completed.

## 13. Recommended implementation order

1. Database schema and migrations.
2. Authentication and role-based access.
3. Story, source, claim and revision models.
4. Newsroom dashboard and editor.
5. Verification workflow.
6. Approval/publication workflow.
7. Public article pages.
8. Audit/correction system.
9. AI assistance layer.
10. Search and analytics basics.

## 14. Definition of Done

AINews MVP v0.1 is complete when an editor can publish a verified South Africa-first story from the newsroom without direct database intervention, and a reader can see the resulting article while the organisation retains a traceable record of sources, approvals, revisions and corrections.
