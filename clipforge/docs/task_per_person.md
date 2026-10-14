# ClipForge Equal Four-Person Development Plan

This plan divides code writing across four people as evenly as possible.

The team should divide work by complete feature slices. Each person writes:

- Frontend code.
- API routes.
- Backend services.
- Database migration or repository code.
- Validation.
- Unit/integration tests.
- Documentation for their feature.

Do not assign one person only frontend and another person only backend. That creates bottlenecks and makes integration harder.

---

## 1. Shared project rule

Use one repository and one integration branch.

Branches:

- main: production-ready code
- develop: combined team code for testing
- rohit/feature-name
- nishant/feature-name
- varun/feature-name
- harsh/feature-name

Nobody pushes directly to main or develop.

Every change goes through a pull request.

> Name → architecture mapping: Rohit = Person 1, Nishant = Person 2, Varun = Person 3, Harsh = Person 4. Architecture source of truth is `docs/architecture.md` (processes 1–7, stores D1/D2/D3). Database source of truth is `docs/database.md` plus `db/schema/`, `db/migrations/`, `db/seed/`. Working structure is `apps/frontend/src/`, `apps/backend/src/`, `packages/`, `tests/`, `scripts/`, `wrangler.toml` — all code areas below use those real paths.

---

## 2. Equal work distribution

### Rohit — Foundation, authentication, and user onboarding (Person 1; architecture process 1)

Primary feature ownership:

- User registration and login.
- Role selection.
- Sessions and permissions.
- Brand and creator profile setup.
- Shared application shell.
- Shared UI foundations.

Code areas (real paths only):

- apps/frontend/src/pages/ (auth, onboarding, shell layouts)
- apps/frontend/src/components/ (base UI)
- apps/frontend/src/hooks/, apps/frontend/src/lib/api-client.ts, apps/frontend/src/styles/
- apps/backend/src/routes/ (auth, me), apps/backend/src/controllers/, apps/backend/src/services/ (auth, users), apps/backend/src/middleware/ (role-check), apps/backend/src/validators/
- packages/auth/
- packages/contracts/ (auth, users, organizations)
- packages/validation/, packages/ui/, packages/config/
- db/schema/ + db/migrations/ (users, sessions, organizations, organization_members)
- db/seed/ + scripts/seed-database.ts + scripts/check-environment.ts
- tests/unit/ + tests/integration/ (auth), tests/fixtures/ (users)
- wrangler.toml, pnpm-workspace.yaml, .github/workflows/ci.yml (with reviewers)

Assigned work:

- CF-001: Repository audit and plan.
- CF-002: Application bootstrap and environment setup.
- CF-003: Shared design system and application shell.
- CF-005: Authentication and role-based access control.
- CF-007: Brand and creator onboarding.
- Part of CF-006: Header, footer, signup, login, and public navigation.
- Part of CF-020: Authentication security and rate limits.

Required code:

- Frontend login and onboarding pages.
- Backend auth routes.
- Session service.
- Role-check middleware.
- User and organization migrations.
- Auth validation.
- Auth tests.
- Shared layout components.

Estimated effort: approximately 25 percent of the total project.

---

### Nishant — Campaigns, database, and brand video assets (Person 2; architecture process 2 + D2)

Primary feature ownership:

- Campaign creation.
- Campaign management.
- Brand campaign assets (campaign videos, logos, thumbnails).
- Brand R2 upload flow (D2 Object Storage; D1 holds only brand R2 keys + metadata).
- Brand campaign dashboard.
- Campaign database models.

Code areas (real paths only):

- apps/frontend/src/pages/ (brand, campaigns), apps/frontend/src/components/ (campaign forms, brand upload controls, budget fields), apps/frontend/src/hooks/, apps/frontend/src/lib/api-client.ts
- apps/backend/src/routes/ (campaigns, brands, brand assets/presign), apps/backend/src/controllers/, apps/backend/src/services/ (campaigns), apps/backend/src/validators/
- packages/contracts/ (campaigns, brand assets)
- packages/storage/ (brand R2 interface; browser → R2 signed URLs for brand assets only, never proxy multi-GB uploads; no creator video writes)
- packages/db/ (campaign repositories), packages/validation/, packages/ui/
- db/schema/ + db/migrations/ (campaigns, campaign_assets) + db/seed/
- tests/unit/ + tests/integration/ (campaigns, brand storage), tests/mocks/ (brand R2 mock), tests/fixtures/, tests/e2e/ (campaign slice)

Assigned work:

- CF-004: Database schema and migrations.
- CF-008: Brand R2 asset upload service (campaign videos, logos, thumbnails only).
- CF-009: Campaign creation.
- CF-010: Campaign management and analytics shell.
- Part of CF-006: Brands page and campaign marketing sections.
- Part of CF-020: Brand file validation, signed URLs, and upload security.
- Part of CF-022: Campaign and brand-upload end-to-end tests.

Required code:

- Campaign list and creation screens.
- Campaign API routes.
- Campaign service and repository.
- Campaign and brand asset migrations.
- Brand R2 signed upload and download logic.
- Brand upload progress and retry UI.
- Campaign validation.
- Campaign and brand storage tests.

Estimated effort: approximately 25 percent of the total project.

---

### Varun — Creator, Instagram/YouTube link submission, and moderation (Person 3; architecture processes 3 + 7)

Primary feature ownership:

- Clipper onboarding.
- Campaign discovery.
- Instagram/YouTube link submission (no video file upload, no creator R2 writes).
- Link submission status.
- Moderation queues.
- Reports and policy actions.

Code areas (real paths only):

- apps/frontend/src/pages/ (clipper, admin/moderation), apps/frontend/src/components/ (link submission form/cards, moderation controls, creator cards), apps/frontend/src/hooks/, apps/frontend/src/lib/api-client.ts
- apps/backend/src/routes/ (submissions, moderation, social connect), apps/backend/src/controllers/, apps/backend/src/services/ (link submissions, moderation, Instagram/YouTube link validation), apps/backend/src/validators/ (instagram/youtube URL + post-ID validation), apps/backend/src/middleware/ (ownership checks)
- packages/contracts/ (link submissions, moderation)
- packages/db/ (submission/report repositories; submissions store provider + postUrl + platformPostId only), packages/validation/, packages/ui/
- db/schema/ + db/migrations/ (submissions, reports/flags, audit_log) + db/seed/
- tests/unit/ + tests/integration/ (link submissions, moderation), tests/mocks/ (Instagram/YouTube mock), tests/fixtures/, tests/e2e/ (creator slice)

Assigned work:

- CF-011: Clipper onboarding and creator profile.
- CF-012: Campaign discovery and campaign details.
- CF-013: Instagram/YouTube link submission workflow.
- CF-014: Moderation queues.
- Part of CF-006: Clippers page and creator onboarding marketing sections.
- Part of CF-020: Instagram/YouTube link policy and submission-abuse protection (no creator file-upload attack surface).
- Part of CF-022: Creator link and moderation end-to-end tests.

Required code:

- Creator profile screens.
- Campaign discovery screens.
- Instagram/YouTube link submission form (provider + public link, no file input).
- Link submission API routes.
- Moderation API routes.
- Link submission and moderation migrations (no mediaAssetId, no creator R2 columns).
- Link status-transition validation.
- Instagram/YouTube URL + platformPostId validation and duplicate submission detection.
- Creator link and moderation tests.

Estimated effort: approximately 25 percent of the total project.

---

### Harsh — Metrics, earnings, billing, analytics, and release testing (Person 4; architecture processes 4 + 5 + 6 + D3)

Primary feature ownership:

- Instagram/YouTube metrics synchronization (D3-scheduled).
- View verification.
- Earnings ledger.
- Creator payouts.
- Brand billing.
- Analytics.
- Notifications.
- Full integration testing and release verification.

Code areas (real paths only):

- apps/frontend/src/pages/ (analytics, earnings, billing), apps/frontend/src/components/ (charts, ledger tables, payment status, notifications), apps/frontend/src/hooks/, apps/frontend/src/lib/api-client.ts
- apps/backend/src/routes/ (metrics, earnings, payouts, analytics, webhooks, admin/audit-log), apps/backend/src/controllers/, apps/backend/src/services/ (Instagram/YouTube metrics, ledger, payments, notifications), apps/backend/src/validators/
- packages/contracts/ (metrics, earnings, payments, notifications)
- packages/db/ (ledger/payment repositories), packages/validation/, packages/ui/
- db/schema/ + db/migrations/ (view_snapshots, earning_ledger, payments, notifications) + db/seed/
- tests/unit/ + tests/integration/ (metrics, ledger, payments), tests/e2e/ (full link journey), tests/mocks/ (Instagram/YouTube + payments mocks), tests/fixtures/
- scripts/cleanup-orphaned-files.ts, docs/deployment.md, .github/workflows/deploy.yml

Assigned work:

- CF-015: Mock Instagram/YouTube metrics synchronization.
- CF-016: Earnings ledger and payout workflow.
- CF-017: Analytics and creator performance.
- CF-018: Brand billing with mock provider.
- CF-019: Notifications and activity feed.
- Part of CF-006: Analytics and performance marketing sections.
- Part of CF-020: Payment and earnings abuse protection.
- Part of CF-022: Full end-to-end link-submission test suite.
- CF-023: Deployment and launch checklist.

CF-021, real Instagram/YouTube adapters, should be shared between Person 4 and the person who owns the related mock provider.

Required code:

- Instagram/YouTube metrics provider interface.
- Mock Instagram/YouTube metrics provider.
- View verification service.
- Earnings ledger.
- Payout request flow.
- Brand billing screens.
- Analytics screens.
- Notification service.
- Payment tests.
- Full user journey tests.
- Deployment and rollback documentation.

Estimated effort: approximately 25 percent of the total project.

---

## 3. Shared work that must be divided equally

Some work affects the whole project and should not be left to one person.

### Design system

Person 1 (Rohit) owns the base system in `packages/ui/`, `apps/frontend/src/components/`, `apps/frontend/src/styles/`.

Each person contributes components for their feature in `apps/frontend/src/components/` (no new top-level `src/<feature>/` folders):

- Person 1 (Rohit): auth, profile, navigation, permissions.
- Person 2 (Nishant): campaign forms, brand upload controls, budget fields.
- Person 3 (Varun): link submission form/cards, moderation controls, creator cards.
- Person 4 (Harsh): charts, ledger tables, payment status, notifications.

### API contracts

Each person owns contracts for their feature in `packages/contracts/` (validated in `packages/validation/`, consumed via `apps/frontend/src/lib/api-client.ts`):

- Person 1 (Rohit): auth, users, organizations.
- Person 2 (Nishant): campaigns, brand assets.
- Person 3 (Varun): link submissions, moderation.
- Person 4 (Harsh): metrics, earnings, payments, notifications.

All contract changes require review from at least one other person.

### Tests

Every person writes tests for their own code in `tests/unit/` + `tests/integration/` plus fixtures in `tests/fixtures/` and doubles in `tests/mocks/`.

Person 4 (Harsh) assembles the final end-to-end flow in `tests/e2e/`, but must not be the only person writing tests.

### Security

Each person secures their own feature (backend `middleware/` + `validators/` + D1 ownership checks):

- Person 1 (Rohit): sessions, roles, permissions.
- Person 2 (Nishant): brand R2 signed-URL auth, brand uploads, R2 objects, file validation.
- Person 3 (Varun): Instagram/YouTube link validation, submission abuse, content policy, moderation.
- Person 4 (Harsh): payments, earnings, webhooks, fraud signals.

Person 1 (Rohit) performs the final security review.

---

## 4. Development order

### Stage 1 — Shared foundation

All people agree on (per `docs/architecture.md` + `wrangler.toml` + `.env.example`):

- Repository structure (`apps/frontend/src/pages|components|hooks|lib|styles`, `apps/backend/src/routes|controllers|services|middleware|validators`, `packages/*`, `db/schema|migrations|seed`, `tests/*`, `scripts/*`).
- Branch naming.
- Package manager (pnpm workspace).
- Environment variables (`APP_ENV`, `R2_*`, `DATABASE_ID`, `DEMO_MODE`, `PAYMENTS_PROVIDER`, `SOCIAL_METRICS_PROVIDER`).
- API response format (`{ data, error, meta }`).
- Error format.
- Authentication roles (Visitor, Brand/Advertiser, Clipper/Creator, Admin/Moderator).
- D1 naming conventions (snake_case tables, `*_cents` money, R2 keys only in D1).
- Shared UI tokens (architecture palette + radius in `packages/ui` / `packages/config`).

Person 1 (Rohit) implements the initial foundation. The other three review it before feature work begins.

### Stage 2 — Parallel feature development

Each person develops their own complete feature slice:

- Person 1 (Rohit): auth and onboarding.
- Person 2 (Nishant): campaigns and brand R2 assets.
- Person 3 (Varun): creator link submission and moderation.
- Person 4 (Harsh): Instagram/YouTube metrics, finance, and D3 jobs.

Each feature must work with demo data (`db/seed/`, `scripts/seed-database.ts`, `DEMO_MODE=true`) before connecting to every other feature.

### Stage 3 — Integration

Merge in this order:

1. Shared contracts (`packages/contracts/`).
2. Database migrations (`db/schema/` → `db/migrations/`, reviewed, tested empty + seeded).
3. Backend services and API routes (`apps/backend/src/`).
4. Frontend pages (`apps/frontend/src/pages/` + `components/`).
5. Feature tests (`tests/unit/`, `tests/integration/`).
6. End-to-end tests (`tests/e2e/`).

### Stage 4 — Final verification

All four people test:

- Registration.
- Brand onboarding.
- Campaign creation.
- Brand asset upload (R2).
- Creator discovery.
- Instagram/YouTube link submission.
- Moderation.
- View verification.
- Earnings.
- Payout.
- Analytics.

---

## 5. API connection rules

The frontend (`apps/frontend/src/lib/api-client.ts`) and backend (`apps/backend/src/`) connect only through API contracts (`packages/contracts/`, validated by `packages/validation/`).

Example:

Frontend call:

- POST /api/campaigns

Request fields:

- name
- type
- description
- budgetCents
- ratePerThousandViewsCents

Response fields:

- id
- name
- status
- budgetCents
- spentCents
- remainingCents

The backend must validate all request fields.

The frontend must not assume fields that are not in the shared contract.

If a backend developer changes a response:

1. Update packages/contracts/.
2. Update apps/backend/src/validators/ + services.
3. Update apps/frontend/src/lib/api-client.ts.
4. Update apps/frontend/src/components/ + pages UI.
5. Add or update tests in tests/unit/, tests/integration/, tests/e2e/.
6. Add a note to the pull request.

Frontend must call only contracted fields via `api-client.ts`; backend must validate via `validators/` + `middleware/` ownership checks.

---

## 6. Database connection rules

Only D1 migrations in `db/migrations/` (from `db/schema/`) create or change tables. D1 holds metadata only; R2 (D2) holds brand assets only (campaign videos, logos, thumbnails); D3 Queues hold jobs. Creator videos are never stored — submissions hold only Instagram/YouTube `provider` + `postUrl` + `platformPostId`. Never store videos in D1 — store only brand R2 object keys + metadata. Never proxy multi-GB uploads through Workers — use browser → R2 signed URLs for brand assets from `packages/storage/`. Money uses integer minor units.

The process is:

1. Feature owner writes the proposed schema change in `db/schema/`.
2. Person 2 (Nishant) reviews the migration.
3. New file added in `db/migrations/`; migration is tested on an empty D1 database.
4. Migration is tested with seed data (`db/seed/`, `scripts/seed-database.ts`).
5. Backend repositories in `packages/db/` are updated.
6. API services in `apps/backend/src/services/` are updated.
7. Frontend is connected only through the API (`apps/frontend/src/lib/api-client.ts`).
8. Integration tests in `tests/integration/` are run.

Never create tables manually in the production dashboard.

Never edit an already-merged migration. Create a new migration instead.

---

## 7. Git pull-request process

Each person creates a branch from the latest develop branch.

Example branch names:

- rohit/cf-005-auth
- nishant/cf-009-campaigns
- varun/cf-013-submissions
- harsh/cf-016-earnings

Before starting:

- Pull the latest develop.
- Create the feature branch.
- Confirm the ticket dependencies.

Before opening a pull request:

- Run type checking.
- Run linting.
- Run unit tests.
- Run integration tests.
- Run the application locally.
- Test the ticket manually.
- Update documentation.
- Add screenshots for UI work.

After another person merges code:

- Pull the latest develop.
- Merge or rebase develop into the feature branch.
- Resolve conflicts.
- Run the tests again.
- Continue development only after the branch is green.

---

## 8. What every pull request must contain

Every pull request must include:

- Ticket ID.
- Feature summary.
- Frontend changes.
- API changes.
- Database changes.
- Storage changes.
- Tests added.
- Commands executed.
- Screenshots for UI changes.
- Known limitations.
- Manual verification steps.

A pull request must not include unrelated changes.

---

## 9. Equal workload review

At the end of each development week, compare:

- Completed ticket points.
- Number of frontend files changed.
- Number of backend files changed.
- Number of database changes.
- Number of tests added.
- Number of unresolved bugs.
- Review work completed.

If one person has significantly more work, move the next feature slice to another person.

Do not measure fairness only by the number of tickets. A database migration may take less time than a complete dashboard, while an end-to-end test may expose bugs across the entire system.

---

## 10. Final ownership summary

| Person | Frontend (`apps/frontend/src/`) | Backend/API (`apps/backend/src/`) | Database (D1: `db/`, `packages/db/`) | Storage (D2 brand R2 / D3 Queue) | Tests (`tests/`) |
|---|---|---|---|---|---|
| Rohit (P1) | Auth, profile, shell (`pages/`, `components/`, `hooks/`, `lib/api-client.ts`) | Auth, permissions (`routes/`, `controllers/`, `services/`, `middleware/`, `validators/`) | Users, sessions, organizations, members | Session data (D1 only) | Auth/security (`unit/`, `integration/`) |
| Nishant (P2) | Brand, campaigns, brand uploads (`pages/`, campaign forms, brand upload controls) | Campaign APIs (`routes/`, `controllers/`, `services/`, `validators/`) | Campaigns, brand assets (`schema/`, `migrations/`, `seed/`) | Brand R2 uploads via `packages/storage/` (browser → R2) | Campaign/brand storage (`unit/`, `integration/`, `mocks/`) |
| Varun (P3) | Clipper, moderation (`pages/`, link submission cards, moderation) | Link submission/moderation APIs (`routes/`, `services/`, `middleware/`) | Link submissions, reports, audit_log (no creator R2) | Instagram/YouTube links only (no creator R2) | Creator link/moderation (`unit/`, `integration/`, `mocks/`) |
| Harsh (P4) | Analytics, billing, earnings (`pages/`, charts, ledger tables) | Instagram/YouTube metrics, ledger, payments (`routes/`, `services/`, webhooks) | Snapshots, ledger, payments, notifications | Reports/exports + D3 jobs | E2E/release (`e2e/`, `unit/`, `integration/`, `mocks/`) |

This gives every person frontend, backend, database, and testing work while keeping each feature logically owned. No new top-level folders — all work stays inside the existing `apps/`, `packages/`, `db/`, `tests/`, `scripts/` structure from `wrangler.toml` / `pnpm-workspace.yaml`.

---

## 11. Final definition of done

The team is ready to merge the complete product when (per `docs/architecture.md` processes 1–7 and stores D1/D2/D3):

- [ ] Each person has completed their feature slice.
- [ ] Frontend (`apps/frontend/src/lib/api-client.ts`) uses shared API contracts (`packages/contracts/`).
- [ ] Backend (`apps/backend/src/validators/` + `middleware/`) validates every request and ownership check.
- [ ] Database changes use new files in `db/migrations/` (from `db/schema/`), tested empty + seeded via `scripts/seed-database.ts`.
- [ ] R2 (D2) stores brand videos, logos, and thumbnails via `packages/storage/` browser → R2 signed URLs.
- [ ] Creator videos are links only (Instagram/YouTube `postUrl` + `platformPostId`); no creator R2 objects, no `mediaAssetId`.
- [ ] D1 stores metadata and application records only (brand R2 keys, not binaries).
- [ ] D3 Queues run sync/email/payout jobs; no Redis dependency.
- [ ] No feature contains duplicated auth or database logic.
- [ ] All pull requests pass CI (`.github/workflows/ci.yml`).
- [ ] All four people have reviewed the integration branch.
- [ ] The complete brand and creator journeys work together.
- [ ] End-to-end tests (`tests/e2e/`) pass from a clean D1 database.

