# Creator Performance Marketplace — Full Website Build Specification

## 1. Objective

Build a production-ready two-sided marketplace where:

- Brands create campaigns and upload approved assets (campaign videos, logos, thumbnails) to R2.
- Independent creators (“clippers”) turn those assets into short-form videos and publish them on Instagram or YouTube. Creators never upload video files to ClipForge.
- Clippers submit the public Instagram/YouTube video link to the platform.
- The platform validates the Instagram/YouTube link, tracks views by platform post ID, verifies eligible views and pays clippers according to campaign rates.
- Brands monitor reach, submissions, spend, and performance from a dashboard.

The product should be functionally similar to a creator-powered clipping marketplace, but must use original branding, copy, colors, illustrations, and UI. Do not copy any third-party logo, text, screenshot, or proprietary visual asset.

Suggested product name: `ClipForge`. Make the name configurable through environment variables and a central brand configuration file.

## 2. Product principles

1. Performance-based: brands pay for verified results, not promised reach.
2. Simple onboarding: a brand or clipper should understand the value within 30 seconds.
3. Trust and transparency: show how views, approvals, earnings, fees, and refunds are calculated.
4. Creator-friendly: campaign requirements and payout rates must be clear before a creator submits work.
5. Safe by default: copyright, fraud, prohibited-content, and privacy protections are built into every workflow.
6. Responsive: the core marketplace must work well on mobile, tablet, and desktop.

## 3. Recommended implementation

Use the Cloudflare stack defined in `docs/architecture.md`. It is already reflected in the repository working structure — do not introduce a different stack (no Next.js App Router, no Prisma/PostgreSQL, no Auth.js, no Redis, no standalone S3):

- Frontend + application logic: Cloudflare Pages/Workers. Code lives in `apps/frontend/src/` (`pages/`, `components/`, `hooks/`, `lib/api-client.ts`, `styles/`).
- Backend API: Cloudflare Workers. Code lives in `apps/backend/src/` (`routes/`, `controllers/`, `services/`, `middleware/`, `validators/`, entry `apps/backend/src/index.ts`).
- Database and metadata: Cloudflare D1 (architecture `D1`). Schema in `db/schema/`, migrations in `db/migrations/`, seed in `db/seed/` (run via `scripts/seed-database.ts`), repositories/helpers in `packages/db/`. Full entities are documented in `docs/database.md`. Store only metadata and R2 object keys in D1 — never store large videos in D1.
- File storage: Cloudflare R2 (architecture `D2 Object Storage — Brand Videos · Logos · Assets`). Storage interface in `packages/storage/`. Brand assets only are uploaded directly from the browser to R2 using signed URLs — never proxy multi-GB uploads through the Workers server. Creator videos are never uploaded; creators submit Instagram/YouTube links only.
- Authentication: secure cookie/session system in `packages/auth/` (email/password with verification, optional magic link, optional Google login, password reset, session revocation, MFA-ready for admins). No Auth.js dependency.
- Payments: mock provider first, Stripe adapter later behind the same interface (`tests/mocks/` for local adapters).
- Social metrics: mock provider first, platform adapters later behind `SocialMetricsProvider` (`tests/mocks/` for local adapters).
- Background jobs: Cloudflare Queues or equivalent job interface (architecture `D3 Background Jobs — Sync · Emails · Payouts`). No Redis dependency.
- Email/notifications: transactional email provider plus in-app notifications, both enqueued via D3. Reusable templates only.
- Analytics: first-party D1 event/snapshot tables plus optional product analytics provider.
- Shared code: `packages/contracts/` (typed API contracts), `packages/validation/` (typed schemas), `packages/ui/` (reusable component system), `packages/config/` (brand config, tokens).
- Tests: `tests/unit/` (Vitest/Jest), `tests/integration/`, `tests/e2e/` (Playwright), `tests/mocks/`, `tests/fixtures/`.
- Scripts and ops: `scripts/check-environment.ts`, `scripts/seed-database.ts`, `scripts/cleanup-orphaned-files.ts`; config in `wrangler.toml`, `pnpm-workspace.yaml`, `.env.example`; CI in `.github/workflows/ci.yml`, deploy in `.github/workflows/deploy.yml`; static assets in `public/`.
- Mapping to `docs/architecture.md` processes: 1. Authentication & User Management → `packages/auth` + backend `auth` routes/services; 2. Campaign Management → campaign routes/services + brand R2 assets; 3. Content & Clip Submission → Instagram/YouTube link submission routes/services (no creator video upload, no creator R2 writes); 4. View Verification & Fraud Detection → metrics services + D3 queue; 5. Analytics & Reporting → analytics routes/services; 6. Billing & Payouts → payments/ledger services; 7. Moderation & Policy → moderation routes/services + `AuditLog`.

If external integrations are unavailable during development, implement provider interfaces and realistic local adapters in `tests/mocks/`. The app must run fully in demo mode (`DEMO_MODE=true`) with seeded data and no external credentials.

## 4. User roles

### Visitor

Can view marketing pages, public campaign examples, FAQs, pricing, and sign-up CTAs.

### Clipper / creator

Can create a creator profile, connect Instagram/YouTube accounts, browse campaigns, submit Instagram/YouTube video links, view approval status, track verified views, and request payouts. No video file upload — link submission only.

### Brand / advertiser

Can create a company and brand profile, create campaigns, upload brand assets (campaign videos, logos, thumbnails) to R2, define payout rules, review link submissions, monitor results, manage billing, and export reports.

### Moderator

Can review flagged campaigns, clips, accounts, and fraud signals. Can approve, reject, suspend, request changes, and record an audit reason. In `docs/architecture.md` this is part of the combined `Admin / Moderator (A)` actor and process `7. Moderation & Policy`.

### Admin

Can manage users, roles, campaigns, platform settings, fees, payout rules, integrations, fraud rules, feature flags, and audit logs. In `docs/architecture.md` this is part of the combined `Admin / Moderator (A)` actor; moderation queues and admin settings share the same `/admin` console with server-side role checks.

## 5. Core terminology

- Campaign: a brand-created opportunity with a brief, assets, target platforms (Instagram, YouTube), payout rules, and budget.
- Clip: a short-form video published by a creator on Instagram or YouTube and submitted to ClipForge as a public video link (no file upload).
- Supported platform: Instagram or YouTube only. `provider` is `instagram` | `youtube`.
- Source asset: video, stream recording, logo, audio, image, or other approved content supplied by a brand and stored in R2. Creator videos are never source assets and never stored in R2.
- Verified view: a view accepted by the platform after Instagram/YouTube API checks and fraud filtering.
- Eligible view: a verified view that satisfies campaign rules and has not exceeded a campaign or clip cap.
- Payout: money owed to a creator based on eligible views.
- Platform fee: the service fee charged to the brand or deducted according to configuration.
- Hold period: a configurable period during which views and payouts may be reviewed for fraud or reversals.

## 6. Brand and visual direction

Create an original visual identity:

- Feel: energetic, modern, creator-first, credible.
- Default palette: near-black ink, warm white, electric violet, cyan accent, and green success state.
- Use high-contrast typography and generous spacing.
- Use rounded cards, subtle borders, strong metric numbers, and compact dashboard charts.
- Avoid copying any existing website’s exact layout, wording, colors, logo, or illustrations.
- All interactive controls must have visible focus states and accessible labels.
- Support light and dark themes if practical; dark theme is preferred for dashboards.

Suggested tokens:

```ts
colors: {
  ink: '#111318',
  surface: '#181B22',
  surfaceMuted: '#232733',
  paper: '#F7F7F4',
  violet: '#7657FF',
  cyan: '#4ED9E8',
  green: '#39C88A',
  amber: '#F2B84B',
  red: '#EE6B78',
}
radius: { sm: '8px', md: '14px', lg: '22px', pill: '999px' }
```

## 7. Public website

### 7.1 Global navigation

Desktop navigation:

- Logo and product name.
- “For Brands”.
- “For Clippers”.
- “How It Works”.
- “Resources”.
- “Log in”.
- Primary CTA: “Get started”.

Mobile navigation:

- Logo.
- Menu button opening a full-width drawer.
- Keep “Get started” visible in the drawer.

### 7.2 Homepage `/`

Build these sections in order:

1. Hero section:
   - Headline explaining the two-sided value proposition.
   - Supporting copy: brands launch campaigns; creators make clips and earn from performance.
   - CTAs: “I’m a brand” and “I’m a clipper”.
   - A dashboard-style visual showing views, clips, and earnings.
2. Trust bar:
   - Use seeded/demo metrics only until real metrics exist.
   - Clearly label claims as platform metrics.
3. Split explanation:
   - Brand path: upload brand assets to R2, set brief, launch, track verified results.
   - Creator path: choose campaign, publish on Instagram/YouTube, submit link, earn. No website video upload.
4. How it works timeline:
   - Create campaign → creators publish on Instagram/YouTube and submit links → views verified → payouts processed.
5. Campaign types:
   - Logo/watermark campaigns.
   - Source-content clipping campaigns.
   - Livestream/highlight campaigns.
   - Music promotion campaigns.
   - UGC campaigns marked “coming soon” if not implemented.
6. Performance section:
   - Explain cost-per-view and verified-view billing.
   - Never promise guaranteed virality.
7. Creator earnings section:
   - Explain that rates vary by campaign and region.
   - Show a simple earnings calculator using clearly marked estimates.
8. Testimonials/case studies:
   - Seeded content must be fictional or explicitly labeled as example content.
9. FAQ accordion.
10. Final CTA and footer.

### 7.3 Brand landing page `/brands`

Include:

- Hero: performance-based short-form distribution.
- Benefits: scalable creator distribution, transparent tracking, flexible budgets, fraud filtering.
- Four-step campaign workflow.
- Campaign type cards.
- Example dashboard screenshots created in the app, not copied from another site.
- Pricing explanation.
- CTA to create a brand account or book a demo.

### 7.4 Clipper landing page `/clippers`

Include:

- Hero: create clips for Instagram/YouTube and earn from verified views.
- Explain that follower count is not the only factor; campaign rules and eligible views matter.
- Steps: connect Instagram/YouTube account, browse, publish on Instagram/YouTube, submit link, earn.
- Creator earnings calculator.
- Supported platform badges: Instagram and YouTube only.
- Content and conduct requirements.
- FAQ and sign-up CTA.

### 7.5 How it works `/how-it-works`

Explain the full lifecycle with an interactive visual:

```text
Brand creates campaign
        ↓
Creator selects campaign
        ↓
Creator publishes video on Instagram or YouTube
        ↓
Creator submits public Instagram/YouTube link (no file upload)
        ↓
Platform validates link and syncs social metrics
        ↓
Fraud and eligibility checks
        ↓
Eligible views become earnings
        ↓
Creator receives payout
```

Include a clear note that Instagram/YouTube API access, view verification, hold periods, and payout timing may vary by platform and region. Creators never upload video files to ClipForge.

### 7.6 Resources

Create:

- `/resources` index.
- `/resources/[slug]` article pages.
- `/case-studies` index.
- `/calculator` public earnings/estimated-reach calculator.

Add a basic CMS-ready content model, even if initial articles are stored as MDX or seed records.

### 7.7 Legal and support pages

Create:

- `/terms`.
- `/privacy`.
- `/creator-terms`.
- `/brand-terms`.
- `/acceptable-use`.
- `/copyright-policy`.
- `/refund-policy`.
- `/contact`.
- `/status`.

Use placeholder legal copy only with an explicit “replace with reviewed legal text” marker in the source. Do not present placeholder text as final legal advice.

## 8. Authentication and onboarding

### 8.1 Authentication

Implements architecture process `1. Authentication & User Management`. Backend lives in `apps/backend/src/routes/`, `controllers/`, `services/`, `middleware/`, `validators/`; shared logic in `packages/auth/`; contracts in `packages/contracts/`; sessions and users in D1 (`db/schema/`, `db/migrations/`, `packages/db/`).

Support:

- Email/password with verification.
- Magic link as optional local adapter (no external dependency required in demo mode).
- Google login as optional adapter behind the same session interface.
- Password reset.
- Session revocation.
- MFA-ready architecture for admins.

Security requirements:

- Hash passwords using a vetted library.
- Secure, HTTP-only cookies.
- CSRF protection where applicable.
- Rate limit login, password reset, and verification endpoints.
- Never expose secrets in client code or logs.

### 8.2 Role selection

After sign-up, ask: “What are you here to do?”

- Grow a brand.
- Create clips and earn.

Users may request an additional role later, but role-specific dashboards should remain clear.

### 8.3 Brand onboarding

Collect:

- Legal/business name.
- Public brand name.
- Website.
- Industry/category.
- Country and billing currency.
- Logo.
- Billing contact.
- Terms acceptance.

Show a checklist:

1. Complete company profile.
2. Add billing method.
3. Create first campaign.
4. Upload brand campaign assets to R2.

### 8.4 Clipper onboarding

Collect:

- Display name.
- Country and payout currency.
- Content categories.
- Instagram/YouTube profiles (public link + optional OAuth connection).
- Payout account.
- Creator agreement acceptance.
- Content policy acknowledgement.

Do not require a follower-count minimum unless a specific campaign does.

## 9. Brand dashboard

Base route: `/app/brand`

### 9.1 Layout

Desktop:

- Left sidebar.
- Top bar with workspace switcher, notifications, help, and profile menu.
- Main content area.

Mobile:

- Bottom navigation or collapsible drawer.
- Cards stack vertically.
- Tables become cards with filters.

Sidebar items:

- Overview.
- Campaigns.
- Clips.
- Analytics.
- Billing.
- Brand assets.
- Team.
- Settings.

### 9.2 Brand overview `/app/brand`

Show:

- Active campaigns.
- Total verified views.
- Eligible views.
- Total spend.
- Average effective CPM.
- Clips submitted.
- Pending reviews.
- Recent performance chart.
- Campaign status table.
- Quick action: “Create campaign”.

Every metric needs a tooltip explaining its definition and date range.

### 9.3 Campaign list `/app/brand/campaigns`

Features:

- Search.
- Status filter: draft, pending review, active, paused, completed, rejected.
- Date filter.
- Platform filter.
- Sort by spend, views, submissions, updated time.
- Table columns: name, status, budget, views, spend, submissions, created date.
- Empty state with CTA.

### 9.4 Campaign creation `/app/brand/campaigns/new`

Use a multi-step form with autosave:

#### Step 1: Campaign type

- Logo/watermark.
- Source-content clipping.
- Livestream highlights.
- Music promotion.
- UGC, disabled or coming soon.

#### Step 2: Campaign basics

- Campaign name.
- Brand.
- Description.
- Objective.
- Target audience.
- Target countries/languages.
- Target platforms: Instagram, YouTube (only supported options).

#### Step 3: Assets (brand assets only; creators never upload video files)

- Upload brand source videos, logos, thumbnails, and optional reference examples directly to R2 (D2).
- Drag-and-drop brand asset uploader.
- Upload progress.
- File validation.
- Virus/malware scan status.
- R2 signed upload/download URLs for brand assets (browser → R2; only R2 keys + metadata in D1).
- Asset preview.

#### Step 4: Creator brief

- What creators should do.
- Required hook or talking points.
- Do’s and don’ts.
- Required watermark placement.
- Required hashtags, mentions, or music.
- Minimum/maximum clip duration.
- Content restrictions.

#### Step 5: Budget and payout

- Total campaign budget.
- Rate per 1,000 or 1,000,000 eligible views.
- Per-clip eligible-view cap.
- Campaign-wide cap.
- Hold period.
- Platform fee preview.
- Estimated range, clearly labeled as an estimate.

#### Step 6: Review and publish

- Show full summary.
- Validate required fields.
- Require confirmation of rights to uploaded brand assets.
- Save draft.
- Submit for moderation.

Campaigns should not go live until payment method, assets, brief, and moderation checks pass.

### 9.5 Campaign details `/app/brand/campaigns/[id]`

Tabs:

- Summary.
- Brief and assets.
- Submitted clips.
- Performance.
- Activity log.

Actions:

- Pause/resume campaign.
- Edit allowed fields.
- Duplicate campaign.
- Export results.
- End campaign.
- Contact support.

Show a budget ledger:

- Starting budget.
- Reserved budget.
- Earned by creators.
- Platform fees.
- Refunds/adjustments.
- Remaining budget.

### 9.6 Clip review `/app/brand/campaigns/[id]/clips`

Provide a review queue with:

- Instagram/YouTube embed preview via submitted link (no uploaded-file player).
- Creator profile.
- Instagram/YouTube URL + platform post ID.
- Submission date.
- Current views.
- Eligibility status.
- Caption/hashtags.
- Approve, reject, request changes, flag buttons.
- Rejection reason selection and optional note.

Support bulk actions only after explicit confirmation.

### 9.7 Brand analytics `/app/brand/analytics`

Charts:

- Verified views over time.
- Eligible views over time.
- Spend over time.
- Effective CPM.
- Views by platform.
- Views by country.
- Top clips.
- Creator contribution.

Filters:

- Date range.
- Campaign.
- Platform.
- Country.
- Clip status.

Export CSV and JSON. Make exports asynchronous for large datasets.

### 9.8 Billing `/app/brand/billing`

Include:

- Current balance.
- Add funds/top up.
- Payment methods.
- Invoices.
- Transactions.
- Campaign budget reservations.
- Refunds and adjustments.
- Billing contact.

Use payment-provider webhooks as the source of truth for successful payment events. Never mark a payment successful only from a client-side response.

## 10. Clipper dashboard

Base route: `/app/clipper`

Sidebar items:

- Discover campaigns.
- My submissions.
- Performance.
- Earnings.
- Connected accounts.
- Profile and settings.

### 10.1 Clipper overview

Show:

- Available earnings.
- Pending earnings.
- Total verified views.
- Active submissions.
- Top-performing clip.
- Recent activity.
- Recommended campaigns.

### 10.2 Campaign discovery `/app/clipper/campaigns`

Campaign cards must show:

- Campaign name and brand.
- Campaign type.
- Payout rate.
- Eligible-view cap.
- Target platforms (Instagram, YouTube).
- Target languages/countries.
- Brief summary.
- Brand asset availability (R2).
- Estimated remaining budget.
- “View campaign” CTA.

Filters:

- Category.
- Platform (Instagram, YouTube).
- Language.
- Payout rate.
- Campaign type.
- Newest/trending.

### 10.3 Campaign details `/app/clipper/campaigns/[id]`

Show:

- Full brief.
- Download/access brand R2 assets through signed URLs (D2; only metadata and R2 keys come from D1).
- Required and prohibited content.
- Example Instagram/YouTube links if provided.
- Payout rules.
- Submission requirements.
- Remaining campaign budget.
- FAQ/contact support.

The creator must explicitly accept campaign rules before submitting.

### 10.4 Clip submission (Instagram/YouTube link only — no video file upload)

Creator workflow:

```text
Creator publishes video on Instagram or YouTube
        ↓
Creator submits public video link
        ↓
Platform validates link and tracks views
```

Form fields:

- Social platform: `instagram` | `youtube` only.
- Public Instagram/YouTube video link (required).
- Caption (optional, copied from Instagram/YouTube where available).
- Optional notes.
- Confirmation that the creator owns or is authorized to use the linked content and that the post is public.

Validation:

- Provider is `instagram` or `youtube`; URL matches the Instagram/YouTube video link format and the selected provider.
- The post is public and accessible where required; private/deleted/unresolvable links are rejected.
- The platform post ID is extracted and stored; duplicate `platformPostId` submissions are rejected.
- Required watermark, hashtags, mentions, and duration rules are met when automatically checkable via the Instagram/YouTube post metadata.
- No video file upload, no `multipart` upload, no R2 write, and no format/size/duration/malware file checks on the creator side. Malware/file checks apply only to brand asset uploads in §9.4 Step 3.

Submission states:

`draft → submitted → processing → approved → tracking → completed`

Alternative states:

`rejected`, `changes_requested`, `flagged`, `disqualified`, `expired`.

### 10.5 My submissions `/app/clipper/submissions`

Display a filterable table/card view with:

- Campaign.
- Platform (Instagram, YouTube).
- Submission date.
- Status.
- Current verified views.
- Eligible views.
- Estimated earnings.
- Last synced.

### 10.6 Earnings `/app/clipper/earnings`

Show:

- Available balance.
- Pending balance.
- Lifetime earnings.
- Earnings by campaign.
- Earnings by month.
- View ledger.
- Payout history.
- Payout account status.

Support a minimum payout threshold and configurable payout schedule. Clearly distinguish estimated, pending, available, paid, reversed, and failed amounts.

### 10.7 Connected social accounts

Allow a creator to:

- Connect Instagram/YouTube accounts through OAuth where available.
- Add a verified public Instagram/YouTube profile URL as a fallback.
- See account connection status.
- Reconnect expired credentials.
- Remove a connection.

Store provider tokens encrypted. Request the minimum scopes required.

## 11. Admin and moderation console

Base route: `/admin`

Protect every admin route with server-side role checks.

Pages:

- Overview.
- Users.
- Brands.
- Clippers.
- Campaigns.
- Clips.
- Reports and flags.
- Payments and payouts.
- Fraud signals.
- Content policy.
- Audit log.
- Settings.

Admin actions require confirmation for suspension, payout reversal, campaign deletion, and balance adjustments.

Every sensitive action creates an immutable audit record with actor, timestamp, target, action, reason, and before/after values.

## 12. Data model

Implements architecture stores `D1` (metadata only), `D2 Object Storage` (brand assets only), `D3 Background Jobs`. D1 schema lives in `db/schema/`, changes ship as new files in `db/migrations/`, demo rows in `db/seed/` plus `scripts/seed-database.ts`, typed access via `packages/db/`. Contracts live in `packages/contracts/`, validation in `packages/validation/`. Never store videos in D1 — store only brand R2 object keys plus file metadata. Creator videos are never stored: they are Instagram/YouTube links + platform post IDs only. Money uses integer minor units (`amountCents`, `budgetCents`), never floating point.

Implement at minimum these entities (full DDL belongs in `docs/database.md`):

### User

- id
- email
- passwordHash or authProviderId
- displayName
- role
- status
- country
- timezone
- createdAt
- updatedAt

### Organization

- id
- name
- type: brand or agency
- ownerUserId
- billingEmail
- country
- status

### BrandProfile

- id
- organizationId
- name
- slug
- website
- description
- logoAssetId
- industry
- socialLinks

### ClipperProfile

- id
- userId
- username
- bio
- avatarAssetId
- categories
- languages
- country
- payoutStatus
- policyStatus

### SocialAccount

- id
- userId
- provider: `instagram` | `youtube`
- providerAccountId
- username
- profileUrl (public Instagram/YouTube profile link)
- tokenEncrypted
- tokenExpiresAt
- scopes
- status

### Campaign

- id
- organizationId
- brandProfileId
- name
- slug
- type
- description
- objective
- status
- moderationStatus
- budgetCents
- reservedCents
- spentCents
- ratePerThousandViewsCents
- perClipViewCap
- campaignViewCap
- holdPeriodDays
- targetPlatforms: subset of [`instagram`, `youtube`]
- targetCountries
- targetLanguages
- requirementsJson
- startsAt
- endsAt
- createdAt
- updatedAt

### CampaignAsset (brand assets only; stored in R2)

- id
- campaignId
- storageKey (brand R2 object key)
- fileName
- mimeType
- fileSize
- durationSeconds
- checksum
- assetType: brand video | logo | thumbnail | reference
- scanStatus

### Submission (Instagram/YouTube link only; no uploaded file, no R2 object)

- id
- campaignId
- clipperProfileId
- socialAccountId
- provider: `instagram` | `youtube`
- postUrl (public Instagram/YouTube video link, required)
- platformPostId (extracted Instagram/YouTube ID, unique)
- caption
- status
- rejectionReason
- moderationNote
- submittedAt
- approvedAt
- lastSyncedAt

### ViewSnapshot

- id
- submissionId
- capturedAt
- rawViews
- verifiedViews
- eligibleViews
- engagementJson
- source

### EarningLedgerEntry

- id
- clipperProfileId
- campaignId
- submissionId
- type: accrual, hold, release, reversal, adjustment, fee
- amountCents
- currency
- eligibleViews
- description
- availableAt
- createdAt

### Payment

- id
- organizationId or clipperProfileId
- provider
- providerPaymentId
- type: topup, payout, refund
- amountCents
- currency
- status
- failureReason
- createdAt

### Report / Flag

- id
- reporterUserId
- targetType
- targetId
- reason
- details
- status
- resolvedBy
- resolvedAt

### AuditLog

- id
- actorUserId
- action
- targetType
- targetId
- reason
- beforeJson
- afterJson
- createdAt

### Session

- id
- userId
- tokenHash
- expiresAt
- revokedAt
- ipHash
- userAgent
- createdAt

### OrganizationMember

- id
- organizationId
- userId
- role
- status
- createdAt

### Notification

- id
- userId
- type
- title
- body
- targetType
- targetId
- readAt
- createdAt

`CampaignAsset.storageKey` is a brand R2 object key (D2); the binary stays in R2. Creator `Submission` rows hold only `provider` + `postUrl` + `platformPostId` — no `mediaAssetId`, no R2 object. `SocialAccount.tokenEncrypted` is encrypted at rest with minimum scopes. `SocialAccount.provider` and `Submission.provider` are `instagram` | `youtube` only.

Use indexes on campaign status, submission status, social post IDs (`platformPostId` unique), created timestamps, and ledger availability dates. Add foreign keys from `BrandProfile.organizationId`, `Campaign.organizationId`, `Submission.campaignId`, `ViewSnapshot.submissionId`, `EarningLedgerEntry.submissionId`, and `Payment` owner references.

## 13. API requirements

Use consistent JSON responses:

```json
{
  "data": {},
  "error": null,
  "meta": { "page": 1, "pageSize": 25, "total": 0 }
}
```

Error shape:

```json
{
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please correct the highlighted fields.",
    "fields": { "name": "Required" }
  }
}
```

Required endpoint groups (implemented in `apps/backend/src/routes/` + `controllers/` + `services/` + `validators/`, typed in `packages/contracts/`, validated in `packages/validation/`, called from `apps/frontend/src/lib/api-client.ts`):

- `GET /api/health`
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/logout`
- `GET /api/me`
- `POST /api/brands`
- `GET /api/brands`
- `POST /api/campaigns`
- `GET /api/campaigns`
- `GET /api/campaigns/:id`
- `PATCH /api/campaigns/:id`
- `POST /api/campaigns/:id/submit-for-review`
- `POST /api/campaigns/:id/pause`
- `POST /api/campaigns/:id/end`
- `POST /api/campaigns/:id/assets/presign`
- `POST /api/submissions`
- `GET /api/submissions`
- `PATCH /api/submissions/:id/review`
- `GET /api/analytics/brand`
- `GET /api/analytics/clipper`
- `GET /api/earnings`
- `POST /api/payouts/request`
- `POST /api/social/instagram/connect`
- `POST /api/social/youtube/connect`
- `POST /api/webhooks/payments/:provider`
- `POST /api/webhooks/social/instagram`
- `POST /api/webhooks/social/youtube`
- `GET /api/admin/audit-log`

All list endpoints require pagination, stable sorting, and authorization checks. Do not trust organization IDs or user IDs supplied by the client.

## 14. Verification and payout engine

Implement the verification engine behind an interface:

```ts
interface SocialMetricsProvider {
  getPostMetrics(input: {
    provider: 'instagram' | 'youtube';
    postId: string;
    accessToken?: string;
  }): Promise<{
    views: number;
    likes?: number;
    comments?: number;
    shares?: number;
    capturedAt: Date;
  }>;
}
```

Rules (implements architecture process `4. View Verification & Fraud Detection`; scheduling via D3/Cloudflare Queues; snapshots in D1 `ViewSnapshot`):

1. Pull metrics on a D3 schedule and on-demand with rate limits.
2. Store raw and normalized snapshots.
3. Detect decreases, deleted posts, duplicate post IDs, suspicious spikes, and disconnected accounts.
4. Apply campaign caps and per-clip caps.
5. Apply hold periods before funds become available.
6. Create immutable ledger entries rather than mutating historical earnings.
7. Reversals must create a new negative ledger entry and an audit record.
8. Use idempotency keys for every sync run and payout event.

Default formula:

```text
eligibleViews = min(
  verifiedViews,
  perClipViewCap,
  remainingCampaignEligibleViews
)

grossEarning = eligibleViews / 1000 * campaignRatePerThousand
creatorEarning = round(grossEarning)
```

Make the formula configurable per campaign and currency. Never use floating point for money; store integer minor units.

## 15. Payments and payouts

Implements architecture process `6. Billing & Payouts`. Ledgers live in D1 (`EarningLedgerEntry`, `Payment`); events are queued via D3; provider webhooks are the source of truth. Build the mock provider first in `tests/mocks/`; add the Stripe adapter later behind the same interface.

### Brand payments

- Allow a brand to add funds using a payment provider.
- Reserve budget when a campaign is activated.
- Decrease available budget as approved eligible views accrue.
- Release unused reserved budget when a campaign ends.
- Maintain a complete transaction ledger.

### Creator payouts

- Require a connected payout account.
- Enforce minimum payout threshold.
- Support manual and scheduled payouts.
- Show estimated arrival date.
- Handle failed, reversed, and restricted payouts.
- Do not expose full bank or tax details in the UI.

### Fees

Store platform fees as configurable settings. Show brands an explicit breakdown before payment and in invoices.

## 16. Moderation, safety, and fraud

Implement:

- Campaign moderation queue.
- Clip moderation queue.
- User reports.
- Copyright complaint workflow.
- Prohibited category settings.
- Age-restricted content handling.
- Malware scanning for brand asset uploads (R2).
- Duplicate-content detection hook (duplicate Instagram/YouTube `platformPostId`).
- Suspicious view-spike detection.
- Account and IP rate limits.
- Manual account suspension.

Example fraud signals:

- Repeated identical view increments across many links.
- Sudden impossible growth.
- Same Instagram/YouTube post submitted to multiple campaigns.
- Private/deleted/unresolvable Instagram/YouTube post.
- Instagram/YouTube account mismatch.
- Excessive submissions rejected for policy violations.

Fraud signals should lower trust scores and route items to review; they should not automatically permanently ban a user without a policy-supported review path.

## 17. Notifications

Implements the `Email / Notification Service` plus D3 queue fan-out from `AUTH`, `MODERATION`, and `BILLING` in `docs/architecture.md`. Store preferences and feed rows in D1 `Notification`; send email via queued jobs.

Support in-app notifications and email notifications for:

- Email verification.
- Campaign approved/rejected.
- Submission approved/rejected/changes requested.
- Social account disconnected.
- Views synchronized.
- Earnings released.
- Payout sent/failed.
- Payment succeeded/failed.
- Campaign budget low.
- Campaign ending soon.

Provide notification preferences per user and per event category.

## 18. Search, filtering, and pagination

- Use server-side pagination for all tables.
- Preserve filters in the URL.
- Provide loading, empty, error, and retry states.
- Debounce search inputs.
- Use cursor pagination for high-volume analytics or activity feeds.
- Do not load all campaigns, clips, or users into the browser.

## 19. Accessibility and UX requirements

- Meet WCAG 2.2 AA where practical.
- Keyboard navigation for every workflow.
- Visible focus indicators.
- Semantic headings and landmarks.
- Accessible dialog, dropdown, tab, and accordion behavior.
- Captions or transcript field for video previews where appropriate.
- Never rely on color alone for status.
- Use clear error messages next to fields.
- Preserve form data after recoverable errors.
- Confirm destructive actions.

## 20. SEO and performance

- Server-render public marketing pages.
- Add page titles and meta descriptions.
- Generate sitemap and robots rules.
- Add Open Graph and Twitter card metadata.
- Use structured data for articles and FAQs.
- Optimize images and video thumbnails.
- Lazy-load below-the-fold media.
- Avoid autoplaying video with sound.
- Target strong Core Web Vitals for public pages.

## 21. Observability

Track:

- Authentication failures.
- Brand asset upload failures (R2).
- Payment webhook failures.
- Instagram/YouTube sync failures.
- Payout failures.
- Queue retries.
- API latency and error rates.
- Fraud signal counts.
- Campaign activation failures.

Use structured logs with request IDs. Redact tokens, passwords, payment secrets, and personal financial information.

## 22. Seed/demo mode

Provide a `demo` environment that contains:

- 3 fictional brands.
- 8 fictional creators.
- 6 campaigns in different statuses.
- 20 sample submissions.
- View snapshots and charts.
- Sample invoices and payout records.
- Admin account clearly marked for local development.

Demo mode must not contact real payment or social APIs. Add a visible “Demo data” indicator when enabled.

## 23. Environment variables

Provide `.env.example` (validated at startup by `scripts/check-environment.ts`, deployed via `wrangler.toml`) with:

```env
APP_ENV=development
APP_URL=http://localhost:8788
PUBLIC_APP_NAME=ClipForge
SESSION_SECRET=
R2_BUCKET_NAME=
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
DATABASE_ID=
DEMO_MODE=true
PAYMENTS_PROVIDER=mock
SOCIAL_METRICS_PROVIDER=mock
```

Never commit real values. Fail gracefully with a clear setup message when an optional provider is not configured. Demo mode must run without real Cloudflare, payment, or social credentials.

## 24. Testing requirements

Locations: unit → `tests/unit/`, integration → `tests/integration/`, e2e (Playwright) → `tests/e2e/`, provider doubles → `tests/mocks/`, deterministic datasets → `tests/fixtures/`. Run via CI in `.github/workflows/ci.yml`.

### Unit tests

Cover:

- Payout formula.
- View cap calculation.
- Budget reservation.
- Fee calculation.
- Status transitions.
- Permission checks.
- Input validation.
- Idempotency behavior.

### Integration tests

Cover:

- Register and sign in.
- Create and save a campaign draft.
- Upload brand asset using an R2 signed-URL mock (`tests/mocks/`).
- Submit campaign for moderation.
- Creator accepts a campaign and submits an Instagram/YouTube link (duplicate and invalid-link cases covered).
- Metrics sync creates view snapshots.
- Eligible views create ledger entries.
- Brand payment webhook updates balance.
- Creator payout request validates threshold and payout account.

### End-to-end tests

Cover:

1. Visitor lands on homepage and reaches sign-up.
2. Brand creates campaign, uploads brand assets, and sees it in dashboard.
3. Clipper discovers campaign and submits an Instagram/YouTube link.
4. Moderator approves submission.
5. Demo metrics sync updates analytics and earnings.
6. Brand pauses campaign and sees budget state update.

## 25. Required deliverables from the coding agent

The coding agent must produce:

- Complete source code in `apps/frontend/src/`, `apps/backend/src/`, `packages/`.
- Database schema (`db/schema/`) and migrations (`db/migrations/`).
- Seed/demo data (`db/seed/`, `scripts/seed-database.ts`, `tests/fixtures/`).
- `.env.example` plus `scripts/check-environment.ts` and `wrangler.toml`.
- Setup instructions in `README.md` plus deploy notes in `docs/deployment.md`.
- API documentation (`docs/api.md`) plus typed contracts in `packages/contracts/`.
- Test suite and test commands.
- Responsive public pages.
- Authenticated brand, clipper, and admin dashboards.
- Mock payment and social providers for local development.
- Error, loading, empty, and permission states.
- A short list of production integrations still requiring credentials or legal review.

## 26. Definition of done

The build is complete when:

- A visitor can understand the product and start either onboarding flow.
- A brand can create a campaign, upload brand assets to R2, configure payout rules, and submit it for review.
- A clipper can browse campaigns, inspect requirements, submit an Instagram/YouTube link, and see status. No clipper video upload exists.
- A moderator can approve/reject campaigns and clips with reasons.
- Demo metric synchronization updates views and earnings deterministically.
- Brand analytics and clipper earnings are backed by database records.
- Payment and payout flows are represented by provider interfaces and safe demo adapters.
- All protected routes enforce role and organization ownership.
- Responsive layouts work on mobile and desktop.
- Core workflows have automated tests.
- No copied third-party branding, screenshots, assets, or text remain.
- The project runs from a clean checkout using the documented setup steps.

## 27. Recommended implementation order

### Phase 1 — Foundation

- Project setup, design tokens, layout, database, authentication, seed data, public homepage.

### Phase 2 — Brand workflow

- Brand onboarding, campaign creation, brand asset upload (R2), campaign list/detail, billing mock.

### Phase 3 — Clipper workflow

- Clipper onboarding, campaign discovery, campaign details, submission flow, earnings ledger.

### Phase 4 — Moderation and analytics

- Admin console, moderation queues, view snapshots, analytics charts, exports.

### Phase 5 — Production hardening

- Real payment integration, social providers, background jobs, fraud signals, notifications, observability, security review, accessibility audit.

At the end of each phase, keep the app runnable and seedable. Do not build static screens that are disconnected from the data model.

## 28. Coding-agent instructions

Before coding:

1. Inspect the existing repository and preserve its working conventions.
2. Identify the current framework, package manager, database, and test setup.
3. If the repository is empty, initialize the recommended stack.
4. Create a concise implementation plan in the project notes.

While coding:

1. Build reusable components instead of duplicating page markup.
2. Keep business rules in server-side services, not only in client components.
3. Use typed schemas for forms and APIs.
4. Add tests for every money, permission, and state-transition rule.
5. Use realistic demo data rather than hardcoded numbers inside UI components.
6. Keep integration providers behind interfaces.
7. Add audit logging for sensitive state changes.
8. Run linting, type checking, migrations, seed, and tests after each major phase.

Before completion:

1. Run the app from a clean environment.
2. Verify the main flows with demo mode enabled.
3. Check mobile layouts and keyboard navigation.
4. Verify that no secrets or personal data are committed.
5. Document any external credential, policy, or legal dependency.

