# SOFTWARE BUILD FRAMEWORK — VisionQuantech (Production-Grade)
Applies to every tagged app/software. Persona: system architect + senior developer, big-tech bar. No mistakes ship.

## 0. GOLDEN RULES
1. **Search → Think → Understand → Build.** Before writing code: research 3-5 best existing systems in the category, note what makes them win, then design.
2. **Four panels, every platform app:** (1) Admin (platform owner — full control, analytics, user/vendor management), (2) User (end customer), (3) Customer Support (tickets, refunds, user lookup, limited PII), (4) Partner (vendor/restaurant/employer/delivery — whoever supplies the service). Tool-type apps (calculators, converters) are exempt from panels — they stay static; overengineering them is a mistake.
3. **Low cost, huge output.** Every infra choice must have a free tier covering launch scale.
4. **Security is not a phase — it's every phase.** Auth vulns, logic breaks, and error paths are reviewed before merge, not after launch.

## 1. SMART TECH STACK (default; deviate only with written reason)
| Layer | Choice | Why |
|---|---|---|
| Frontend | Static HTML/CSS/JS or lightweight SPA, Cloudflare Pages | Free, global CDN, instant |
| Backend | Cloudflare Workers (100k req/day free) or Supabase Edge Functions | Zero server cost, auto-scale |
| Database | Supabase Postgres (free tier) / Neon | Real SQL, Row-Level Security |
| Cache / Rate-limit / Queues | Upstash Redis (free tier) | Sessions, rate limits, job queues, realtime counters |
| Auth | Supabase Auth + RLS policies | JWT, refresh rotation, OAuth; **RLS on every table, no exceptions** |
| File storage | Cloudflare R2 (10 GB free) / Supabase Storage | No egress fees |
| Realtime | Supabase Realtime | Tracking, chat, live order status |
| Payments | Razorpay (India) — test mode until revenue | Never store card data |
| Monitoring | Uptime checks + error logging from day one | Know before users complain |

## 2. BUILD SEQUENCE (no skipping steps)
1. **PRD** — problem, users (per panel), user stories, success metrics, non-goals, monetization (free/ads/Pro/API).
2. **Competitor research** — 3-5 systems: feature matrix, pricing, what to copy, what to beat.
3. **TRD** — architecture diagram, tech stack, data model (ERD), API contracts (endpoint, auth, rate limit), Redis usage (what's cached/queued), scaling plan (10x/100x), cost estimate at 1k/100k/1M users.
4. **UI flow** — screen map for ALL FOUR panels, click-path per role, empty/error states designed (not discovered).
5. **Build** — backend first (APIs + RLS + tests), then panels.
6. **Security review** — section 4 checklist, signed off per app.
7. **Load & break test** — hammer the logic: double-submit, expired session, negative amounts, huge uploads, concurrent edits.
8. **Launch** — SEO pages, analytics, backup/restore tested, rollback plan.

## 3. FOUR-PANEL RBAC MATRIX (enforce in DB RLS + API middleware, never frontend-only)
- **Admin:** everything except direct PII export without audit log; all actions logged.
- **User:** own data only; payments; support tickets.
- **Support:** read user/order state, issue refunds/credits within limits, no settings/schema access; every action ticket-linked.
- **Partner:** own listings/orders/earnings only; cannot see other partners' data or platform revenue.
- Principle: **deny by default**; each role gets explicit grants. Support/admin sessions expire faster.

## 4. SECURITY CHECKLIST (per app, before launch)
- [ ] Auth: no IDOR (every object fetch checks ownership), JWT expiry + refresh rotation, OAuth state validated, password rules + breach-list check
- [ ] Rate limiting on auth, OTP, and paid endpoints (Redis counters)
- [ ] All input validated server-side; SQL via parameterized queries/RLS; XSS escaped; CSRF tokens on cookie auth
- [ ] File uploads: type + size + malware-scan posture; served from separate domain; no executable
- [ ] Secrets in env only; never in repo, logs, or client bundles
- [ ] Payment webhooks signature-verified; amounts from server, never client
- [ ] Audit log for admin/support actions; PII access logged
- [ ] Error messages leak nothing (no stack traces, no "user exists" on login)
- [ ] Backups tested (restore drill), RPO/RTO written down
- [ ] Dependency scan; no known-critical CVEs at launch

## 5. RELIABILITY & SCALE PATTERNS (use where they pay off)
- Redis: session cache, rate limits, idempotency keys (double-submit protection), queue for emails/PDFs/notifications, leaderboard/counter cache.
- Idempotent writes for payments/orders (retry-safe).
- Pagination on every list; N+1 queries banned; indexes on foreign keys + search fields.
- Graceful degradation: if Redis/secondary is down, core flows still work (slower, not dead).
- Cost guardrails: per-user spend caps, alert at 70% of free-tier quotas.

## 6. DEFINITION OF DONE (production-grade = all true)
- PRD + TRD + UI flow docs exist in the app folder
- All four panels (where applicable) work end-to-end with RBAC tests passing
- Security checklist signed, load test notes filed
- SEO pages + analytics + error tracking live
- README: setup, architecture, cost-at-scale, rollback steps
- Demo video/GIF of the four panels in the app folder
