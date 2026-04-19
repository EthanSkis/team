# This site is migrating to the unified ClearBot app

The `team.clearbot.io` dashboard is moving into the single Next.js
project at [`EthanSkis/clearbot.io`](https://github.com/EthanSkis/clearbot.io),
which now serves every `*.clearbot.io` host from one Vercel project.

The overview + bookings admin pages were ported into that repo's
`app/(team)` route group. The hardcoded `TEAM_EMAILS` allowlist from
the old `index.html` is gone: team access now gates on
`profiles.role IN ('admin','team')`, enforced server-side in
`lib/auth/require.ts#requireTeam`, with a `TEAM_EMAIL_FALLBACK` env
for bootstrapping before the first admin row exists.

Booking mutations (status updates + delete) go through
`PATCH/DELETE /api/bookings/[id]` with the service-role client, so
the admin allowlist can't be bypassed from the browser.

## Cutover

Don't delete this repo's `CNAME` until `team.clearbot.io` is pointed
at `cname.vercel-dns.com`. Staged order: `signup` → `login` →
`client` → `team` → apex. Keep this repo live as a rollback target
during the cutover window.

## After cutover

Once DNS is flipped, this repo can be archived.
