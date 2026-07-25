# Cloud Agent brief — full Hostinger migration

**Mission:** Handle the **entire** WordPress/WooCommerce migration of https://alldrivesauto.com from Namecheap shared hosting to the **InterServer** VPS, end-to-end, with zero surprise DNS cutover.

Cloud Agents do **not** see local Cursor chats. This file is the source of truth.

---

## Site / target

| Item | Value |
|------|--------|
| Live site | https://alldrivesauto.com |
| Current host | Namecheap shared / cPanel |
| New host | **InterServer Premium VPS** (target **8 slices** = ~16 GB RAM / ~320 GB disk), **New Jersey East Coast · USA** |
| OS | **Ubuntu** plain (no cPanel unless later requested) |
| Approx data | ~50 GB files + database |
| Stack | WordPress + WooCommerce (Mobex theme) + existing plugins |

---

## Human vs Cloud Agent

### Human must do (cannot be automated)

1. Finish InterServer checkout / pay for the VPS  
2. Put these into **Cloud Agent Secrets** (or paste once in the kickoff prompt) — never commit to git:
   - VPS SSH: host/IP, user (`root`), password  
   - Namecheap cPanel **or** SFTP + DB export access  
   - Domain DNS login (if not Namecheap)  
   - WP admin URL/user if needed for smoke checks  
3. Explicitly approve **DNS cutover** when the agent says staging looks good  

### Cloud Agent must do (entire technical migration)

Follow phases below in order. Prefer idempotent steps; log progress in `docs/migration-log.md` in this repo. **Do not change public DNS until Phase 6 is approved.**

---

## Phase 0 — Kickoff checks

1. Confirm SSH to VPS works  
2. Confirm Namecheap/cPanel or SFTP + MySQL dump access works  
3. Record VPS public IP, disk free space, Ubuntu version  
4. Create `docs/migration-log.md` with timestamps  

Stop and ask the human if any access is missing.

---

## Phase 1 — Harden & base stack on VPS

Install and configure (Ubuntu):

- Firewall (UFW): SSH, HTTP, HTTPS only (plus anything required for Hostinger panel if needed)  
- Fail2ban (basic SSH protection)  
- Nginx (or Apache if existing stack strongly prefers it — default **Nginx**)  
- PHP 8.x (match site needs; prefer 8.1+), PHP-FPM  
- MySQL or MariaDB  
- Composer / WP-CLI  
- Certbot (Let’s Encrypt) — certs after domain points, or use temporary hostname/IP testing first  
- Swap if RAM pressure during import (16 GB should be fine; still add modest swap)  

Create system user / web root, e.g. `/var/www/alldrivesauto` (adjust as needed).

---

## Phase 2 — Export from Namecheap (while live site stays up)

1. Full files backup: `public_html` (or WP root), including `wp-content/uploads`  
2. Full MySQL dump of the WordPress database  
3. Note PHP version, cron jobs, email setup, SSL, redirects  
4. Transfer archives to VPS (rsync/SCP) — expect large uploads (~50 GB); resume if interrupted  

Keep Namecheap serving production the whole time.

---

## Phase 3 — Restore on VPS (staging on IP)

1. Unpack files into web root  
2. Import database  
3. Create DB user/password; update `wp-config.php` (DB_*, salts if needed, FS method)  
4. Set correct file ownership/permissions  
5. Configure Nginx vhost for:
   - temporary: `http://VPS_IP` and/or hosts-file name, **or**
   - a staging subdomain if human provides one  
6. Search-replace URLs only for **staging** if using a temp domain (use WP-CLI `search-replace` carefully; keep serialized data safe)  
7. Disable aggressive cache plugins until stable  
8. Smoke-test: home, shop, product, cart, checkout page load, admin login, sample search  

---

## Phase 4 — Fix migration issues

Resolve until green:

- Missing PHP extensions  
- Permalink / rewrite rules  
- HTTPS mixed content (after SSL)  
- Cron / WP-Cron  
- Object cache / Redis optional later — not required for cutover  
- Large uploads / max execution / memory limits appropriate for WooCommerce  
- Mobex / plugin compatibility  

Document each fix in `docs/migration-log.md`.

---

## Phase 5 — Pre-cutover checklist (agent prepares; human approves)

Agent produces a short checklist in the log:

- [ ] Site works on VPS IP / staging URL  
- [ ] Admin works  
- [ ] Products/images load  
- [ ] Checkout page loads (full payment test optional with human)  
- [ ] Backup of Namecheap taken again (fresh dump)  
- [ ] Rollback plan written (DNS back to Namecheap; keep Namecheap online 48h+)  
- [ ] TTL lowered on DNS if possible (e.g. 300s) before switch  

**STOP.** Ask human: “Approve DNS cutover to VPS IP?”

---

## Phase 6 — DNS cutover (only after approval)

1. Point A/AAAA (and www CNAME/A as needed) to VPS  
2. Issue Let’s Encrypt certs for `alldrivesauto.com` + `www`  
3. Force HTTPS  
4. Final WP URL check (`siteurl` / `home`)  
5. Monitor 30–60 minutes: errors, SSL, forms, cart  

If broken: roll DNS back to Namecheap immediately and report.

---

## Phase 7 — Post-cutover (same agent run or follow-up)

1. Keep Namecheap as fallback ~48–72 hours  
2. Confirm Hostinger/InterServer backup options; enable if available  
3. Optional next (separate prompt unless human says include now): install **Meilisearch** + Woo search integration  

---

## Kickoff prompt (paste when VPS + secrets are ready)

In this Cursor chat, say:

```text
Let the Cloud Agent handle the entire migration.
Secrets are in Cloud Agent Secrets (or pasted below).
Read CLOUD_AGENT_BRIEF.md and execute Phases 0–7 for InterServer.
Do not change public DNS until I approve Phase 6.
```

Or on https://cursor.com/agents with repo `Trudyranks/ada-product-search`:

```text
Read CLOUD_AGENT_BRIEF.md. Execute the full InterServer migration (Phases 0–7).
Never change public DNS until I explicitly approve. Log progress in docs/migration-log.md.
```

---

## Out of scope unless asked

- Paying InterServer / Hostinger / Contabo  
- Mass product scraping/import during migration  
- Meilisearch (default: after cutover)  
- Phone/voice AI  
- cPanel (not required; use SFTP/SSH)

---

## Success definition

Public https://alldrivesauto.com serves from InterServer VPS with working shop/admin, SSL valid, Namecheap still available for rollback, and migration log committed to this repo.
