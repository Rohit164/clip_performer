# ClipForge

Two-sided creator-performance marketplace on Cloudflare (Pages/Workers + D1 + R2 + Queues).

## Creator workflow (link-only, no website video upload)

```text
Creator publishes video on Instagram or YouTube
        ↓
Creator submits public video link
        ↓
Platform validates link and tracks views
```

- Supported platforms: Instagram, YouTube only (`instagram` | `youtube`).
- Creators never upload video files to ClipForge. No creator file input, no creator R2 writes, no `mediaAssetId`.
- Creator `Submission` = `provider` + `postUrl` + `platformPostId` in D1. Views are tracked by platform post ID.
- Brand source assets only (campaign videos, logos, thumbnails) are uploaded to R2/D2 via browser → R2 signed URLs (`packages/storage/`).

## Structure

- `apps/frontend/src/` — `pages/`, `components/`, `hooks/`, `lib/api-client.ts`, `styles/`
- `apps/backend/src/` — `routes/`, `controllers/`, `services/`, `middleware/`, `validators/`
- `packages/` — `auth/`, `contracts/`, `db/`, `storage/` (brand R2 only), `ui/`, `validation/`, `config/`
- `db/` — `schema/`, `migrations/` (D1), `seed/`
- `tests/` — `unit/`, `integration/`, `e2e/`, `mocks/` (Instagram/YouTube + brand R2 + payments), `fixtures/`
- `docs/` — `architecture.md` (processes 1–7, D1/D2/D3 + link-only rule), `overview.md` (link-only spec), `task_per_person.md`, `api.md`, `database.md`, `decisions.md`, `deployment.md`
- `scripts/` — `check-environment.ts`, `seed-database.ts`, `cleanup-orphaned-files.ts`
- `wrangler.toml`, `pnpm-workspace.yaml`, `.env.example`

## Docs

Start with `docs/architecture.md`, then `docs/overview.md`.
