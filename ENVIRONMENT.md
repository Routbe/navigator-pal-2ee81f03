
## PSP_ENCRYPTION_KEY
Server-only. AES-256-GCM key material for creator PSP secrets (`psp_credentials`). At least 32 random characters.

## MIGRATION_URL
Owner connection to Neon, used only by `bun scripts/migrate.ts` (never at runtime). Must differ from DATABASE_URL (rout_app). Not needed on Vercel.

## GITLAB_ISSUER
Optional GitLab server URL (default https://gitlab.com, no trailing slash). Set for self-managed GitLab.

## GITLAB_REDIRECT_URI
Optional. Defaults to <BETTER_AUTH_URL>/api/auth/callback/gitlab (e.g. https://rout.be/api/auth/callback/gitlab). Must match the GitLab application exactly.
