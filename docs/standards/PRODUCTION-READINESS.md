# Production readiness checklist

> Part of **Andreas-standardoppsett** (topic: `production-readiness`). Installed into projects as `docs/standards/PRODUCTION-READINESS.md`.
> Based on the **"production readiness review"** idea from Matt Murphy, as captured in Andreas's course notes; adapted for a small **Next.js + Supabase** web app.
> Use before a launch/release to real users. Mark every item ✅ pass / ❌ fail / ❓ unknown. Unknown is not pass.
> Facts checked 22 Oct 2026. Legal items are **not legal advice**.

## 1. Security

*Why:* one leaked key or one table without row-level security can expose every user's data. Most breaches in small apps are configuration mistakes, not clever hacks.

- [ ] **RLS enabled on every table** in exposed schemas (`public` etc.): `select schemaname, tablename, rowsecurity from pg_tables where schemaname = 'public';` – all `true`.
- [ ] Every table has policies that match intent (read/insert/update/delete), and they're **tested with a second user**: sign in as user B and try to read/update user A's rows (and an org B member trying org A data). Supabase's Security Advisor (Dashboard → Advisors) shows no errors.
- [ ] **`service_role` / secret key never reaches the browser:** only used in server code (Route Handlers, Server Actions, Edge Functions); never in a `NEXT_PUBLIC_*` variable; `grep -r "service_role\|SUPABASE_SERVICE" .next/static` finds nothing after `npm run build`.
- [ ] All secrets in environment variables (hosting provider / `.env.local`), `.env*` in `.gitignore`, no secrets in git history (`git log -p | grep -i "key\|secret"` spot-check, or a scanner like gitleaks). Rotate anything that ever leaked.
- [ ] Server-side authorization on every Server Action / Route Handler (check the session and the user's rights on the server, not just by hiding buttons).
- [ ] Input validated on the server (e.g. zod); no raw SQL string concatenation; no `dangerouslySetInnerHTML` with user content (or sanitized).
- [ ] **Security headers** (in `next.config` `headers()` or middleware): `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `frame-ancestors` (CSP) or `X-Frame-Options`, `Permissions-Policy`. Check with https://securityheaders.com.
- [ ] HTTPS only; cookies `Secure`, `HttpOnly`, `SameSite`.
- [ ] **Rate limiting** on auth and expensive/public API routes (Supabase Auth rate limits reviewed in Dashboard → Authentication; own routes limited via middleware/edge or provider). CAPTCHA on signup if abuse is likely.
- [ ] Auth settings: email confirmation on, sensible password rules, redirect URL allow-list only contains your domains, leaked-password protection if available on your plan.
- [ ] Storage buckets: private unless they must be public; storage policies tested like RLS.
- [ ] **Dependency audit:** `npm audit --omit=dev` has no high/critical; Dependabot (or similar) enabled.
- [ ] **OWASP Top 10 pass** (https://owasp.org/Top10/): access control, crypto, injection, insecure design, misconfiguration, vulnerable components, auth failures, integrity, logging, SSRF – one line each on how the app handles it.
- [ ] Admin features require an admin role checked on the server; at least two people/accounts can access hosting and Supabase (no single point of lock-out), with 2FA.

## 2. Privacy (personvern / GDPR)

*Why:* GDPR applies in Norway through the EEA. If you process personal data (names, emails, member lists), you must say what you collect, keep it minimal, protect it, and let people see and delete it. Datatilsynet supervises.

- [ ] **Data inventory:** list every personal data field, where it's stored (DB tables, storage, logs, analytics, email provider), why, and for how long.
- [ ] **Minimal collection:** remove fields you don't need; no sensitive data (health, religion, etc.) unless necessary and handled with extra care. Note: data revealing religious belief (e.g. membership of a church music team) can be a special category under GDPR Art. 9 – consider whether this applies.
- [ ] **Privacy policy (personvernerklæring) matches reality:** who you are, what data, purpose, legal basis, processors (Supabase, hosting, email, error tracking), storage location, retention, user rights, contact.
- [ ] **Processors and DPAs:** data processing agreement accepted with Supabase (https://supabase.com/legal/dpa) and every other processor (hosting, Sentry, email).
- [ ] **Supabase region** chosen consciously (EU region, e.g. Frankfurt/Stockholm if available, keeps data in the EEA); transfers outside the EEA (e.g. US-based processors) covered by the processor's terms (EU–US Data Privacy Framework or SCCs).
- [ ] **Export path:** a user (or you, on request) can export their data.
- [ ] **Deletion path:** account deletion removes or anonymizes their rows *and* storage files, and is documented (backups expire per retention).
- [ ] **Cookies / local storage:** since 1 Jan 2025 Norway's ekomloven § 3-15 requires GDPR-level consent *before* storing/reading non-essential cookies or similar (analytics, ads, tracking pixels). Strictly necessary ones (login session) are exempt. No non-essential trackers → no banner needed. If you need one: "No" as easy as "Yes".
- [ ] Logs and error tracking don't contain passwords, tokens, or unnecessary personal data (Sentry: scrub PII).
- [ ] A simple plan for a data breach: who decides, notify Datatilsynet within 72 h when required (GDPR Art. 33).

## 3. Backup

*Why:* mistakes (yours, an agent's, or a bad migration) delete data. A backup you have never restored is a hope, not a backup.

- [ ] **Know your plan's backups:** Supabase **Free has no automatic backups** – export regularly yourself (`supabase db dump` / `pg_dump`) and store off-site. **Pro** keeps 7 days of daily backups, **Team** 14, **Enterprise** up to 30. **PITR** (restore to any second, RPO ≈ 2 min) is a paid add-on on Pro+ and needs at least Small compute. Source: https://supabase.com/docs/guides/platform/backups
- [ ] **Storage files are NOT in database backups** (only their metadata). Back up buckets separately (script that downloads/syncs objects, or keep originals elsewhere).
- [ ] **Restore tested** at least once: restore a dump into a new/dev project and check the app works against it. Note the time it took.
- [ ] Deleting a Supabase project deletes its backups too – keep an off-site dump before any project deletion.
- [ ] Migrations are in git (`supabase/migrations/`) so the schema can be rebuilt; destructive migrations have a tested rollback plan.
- [ ] Code, env-var list (names, not values) and config are recoverable (git + password manager).

## 4. Monitoring

*Why:* you should hear about problems from an alert, not from an angry user a week later.

- [ ] **Uptime monitor** on the main URL and a health endpoint (e.g. UptimeRobot, Better Stack, hosting's own), alerting to email/phone.
- [ ] **Error tracking** in browser and server (e.g. Sentry for Next.js), with source maps uploaded and PII scrubbing on.
- [ ] **Logs** available and you know where: hosting logs (Vercel/Coolify), Supabase Logs Explorer (API, Postgres, Auth).
- [ ] **Alerts** for: site down, error spike, Supabase usage near plan limits (database size, egress, MAU), failed backups/dumps.
- [ ] Someone receives the alerts and knows the first steps (a 5-line runbook in the README).

## 5. Error handling

*Why:* things will fail (network, Supabase hiccups, bad input). Users should get a clear message and a way forward, never a white screen or a stack trace.

- [ ] Next.js **`error.tsx`** in key route segments and **`global-error.tsx`**; **`not-found.tsx`** for 404s. Friendly text, a retry button, a link home.
- [ ] **No stack traces, SQL errors or internal IDs shown to users**; detailed errors go to logs/Sentry only.
- [ ] Every Supabase call checks `error` and handles it (no silent failures; no assuming `data` is non-null).
- [ ] **Timeouts and retries** for network calls (retry idempotent reads with backoff; don't blindly retry writes). Loading and empty states exist.
- [ ] Forms keep user input after an error; errors are announced to screen readers (see accessibility topic).
- [ ] Offline / flaky network behaviour is acceptable for the main flows (especially on phones at rehearsals/venues).

## 6. Performance

*Why:* slow apps lose users, especially on phones; free-tier limits can take the app down when it gets popular.

- [ ] **Lighthouse** (mobile) on key pages: Performance ≥ 90 as a goal; **Core Web Vitals** "good": LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1 (https://web.dev/articles/vitals).
- [ ] Images via `next/image` with proper `width`/`height`/`sizes`; large media compressed; fonts via `next/font`.
- [ ] Static assets cached (Next.js hashed assets are immutable by default; check `Cache-Control` on your own public files); use static/ISR rendering where data allows.
- [ ] Bundle size checked (`@next/bundle-analyzer`); heavy libraries (PDF.js, editors, charts) loaded with dynamic `import()` only where needed.
- [ ] **Database indexes** on columns used in `where`, `order by`, joins and RLS policies (e.g. `org_id`, `user_id`); Supabase Performance Advisor has no critical findings; no N+1 query loops; pagination on long lists.
- [ ] **Supabase plan limits** known and monitored (database size, storage, egress, monthly active users, realtime connections; Free projects can also be paused after inactivity). Check current numbers at https://supabase.com/pricing.
- [ ] Realtime subscriptions are cleaned up on unmount.

## 7. Accessibility

*Why:* real users include people using keyboards, screen readers, zoom and phones; Norwegian law requires universal design of ICT aimed at the public.

- [ ] Run the **Definition of done in `docs/standards/ACCESSIBILITY.md`** (topic `accessibility`) on every main page and flow; axe reports 0 violations; keyboard and screen-reader spot-checks done.
- [ ] Optional but good practice: an accessibility statement page (required only for public bodies in Norway).

*(Don't duplicate the accessibility checklist here – keep it in the accessibility topic.)*

## 8. Legal

*Why:* launching to real users creates obligations and risk. Check these with someone qualified – this list is **not legal advice**.

- [ ] **Privacy policy** published and linked from signup and footer (see area 2).
- [ ] **Terms of use** (vilkår): what the service is, acceptable use, user content ownership, liability limits, how you can change/close the service.
- [ ] **Platform terms** you depend on allow your use: Supabase (https://supabase.com/terms), hosting provider, email provider, any APIs. Check plan limits/fair use for commercial use.
- [ ] **Content rights:** song lyrics, chord charts and sheet music are usually copyrighted. Storing/displaying/sharing them may need a licence (e.g. CCLI, or rights holders; in Norway TONO / Kopinor may be relevant depending on use). Users uploading content should confirm they have the rights.
- [ ] **Insurance:** if you charge money or businesses depend on the app, consider professional/cyber liability insurance.
- [ ] Payments (if any): consumer rules (angrerett, price display), payment provider terms, invoicing/MVA – check with an accountant.
- [ ] Accessibility law context: see `docs/standards/ACCESSIBILITY.md` §2 (Norway forskriften, EU EAA).
- [ ] Company/contact info visible (who runs the service and how to reach them).

## Report template

```
Production readiness – <project> – <YYYY-MM-DD>
Summary: X ✅ · Y ❌ · Z ❓   Blockers: …
1 Security        ✅ … / ❌ … / ❓ …
2 Privacy         …
3 Backup          …
4 Monitoring      …
5 Error handling  …
6 Performance     …
7 Accessibility   …
8 Legal           …
Top fixes before launch (by risk): 1. … 2. … 3. …
```
