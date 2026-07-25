# Cloud Agent brief — InterServer migration

**Mission:** Handle the **entire** WordPress/WooCommerce migration of https://alldrivesauto.com from Namecheap shared hosting to the **InterServer** VPS, end-to-end, with zero surprise DNS cutover.

Cloud Agents do **not** see local Cursor chats. This file is the source of truth in the repo.

**Repo:** `Trudyranks/ada-product-search` · branch `main`  
**Kickoff phrase:** *Let the Cloud Agent handle the entire migration.*

---

## Current status (2026-07-24) — VPS ordered & registered

| Item | Value |
|------|--------|
| Live site | https://alldrivesauto.com |
| Current host | Namecheap shared / cPanel (keep live until Phase 6 approved) |
| New host | **InterServer Premium VPS** — paid & Active |
| Service ID | **VPS#3516118** |
| Hostname | `vps3516118` / PTR `vps3516118.trouble-free.net` |
| Public IP | **162.35.185.121** (confirmed on invoice #47647919 + reverse DNS) |
| Host node | KVM546 (panel also shows this IP under “HOST SERVER” — guest IP is the same) |
| Package | KVM Linux VPS Slice (**Ubuntu 8 Slices**) · **$24/mo** · New Jersey East Coast · USA |
| Panel | https://my.interserver.net/view_vps?id=3516118 |
| Account email | bearasspain@gmail.com |
| Call-in PIN | 623313 (for InterServer phone support) |
| Console OS | Ubuntu **24.04 LTS** at tty1 login (noVNC available via panel “View Desktop”) |
| Panel OS label | Was shown as `ubuntu26` / template options include `ubuntu24`, `ubuntu26`, desktop |
| cPanel | **Not** installed (use SSH/SFTP). Optional later (~$23+/mo; marked Not Supported on some SKUs) |
| Approx data | ~50 GB files + database |
| Stack | WordPress + WooCommerce (Mobex theme) + existing plugins |

### SSH / credentials status (important)

- SSH **port 22 is open** on `162.35.185.121`.
- Banner: `OpenSSH_10.2p1 Ubuntu-2ubuntu3.5`.
- Auth methods: `publickey,password`.
- A panel **Change Root Password** was queued successfully (“allow up to 2 minutes”), but SSH still returned **Permission denied (password)** afterward.
- **Do not assume** the local `.secrets` password works until Phase 0 verifies SSH.
- Recovery if SSH fails at kickoff:
  1. Use InterServer panel → **Change Root Password** again, wait 2+ minutes, retry SSH as `root`.
  2. Or **Reinstall OS** → choose **Ubuntu 24.04** (`ubuntu24`, **not** Desktop) → set a known root password → confirm with InterServer **account** password → wait for reinstall → SSH.
  3. Or use **View Desktop** (noVNC) if credentials from welcome email work on console.
- **Never commit passwords** to git. Put them in **Cloud Agent Secrets** (see below).

---

## Secrets checklist (Cloud Agent Secrets / kickoff prompt)

Put these in Cursor **Cloud Agent Secrets** before kickoff (or paste once in the prompt). Do **not** commit to the repo.

| Secret key (suggested) | What it is | Status |
|------------------------|------------|--------|
| `VPS_IP` | `162.35.185.121` | Known — can hardcode or secret |
| `VPS_USER` | `root` | Known |
| `VPS_ROOT_PASSWORD` | Working root password | **Must verify / fix at Phase 0** |
| `INTERSERVER_PANEL_URL` | https://my.interserver.net/view_vps?id=3516118 | Known |
| `INTERSERVER_ACCOUNT_PASSWORD` | my.interserver.net login (for reinstall confirm if needed) | Optional backup |
| `NAMECHEAP_CPANEL_URL` | cPanel URL | **Human must provide** |
| `NAMECHEAP_CPANEL_USER` | cPanel user | **Human must provide** |
| `NAMECHEAP_CPANEL_PASSWORD` | cPanel password | **Human must provide** |
| `WP_ADMIN_URL` / user | Optional smoke-test admin | Optional |

Local draft (gitignored, not for Cloud Agent): `.secrets/interserver-vps.env` on the operator machine.

---

## Human vs Cloud Agent

### Human must do

1. ~~Finish InterServer checkout / pay~~ **DONE**
2. Ensure **Cloud Agent Secrets** include working SSH + Namecheap access  
3. Explicitly approve **DNS cutover** when the agent says staging looks good  

### Cloud Agent must do (entire technical migration)

Follow phases below in order. Prefer idempotent steps; log progress in `docs/migration-log.md` in this repo. **Do not change public DNS until Phase 6 is approved.**

---

## Phase 0 — Kickoff checks

1. Confirm SSH to `root@162.35.185.121` works (fix password via panel/reinstall if needed)  
2. Confirm Namecheap/cPanel or SFTP + MySQL dump access works  
3. Record VPS public IP, disk free space, Ubuntu version (`uname -a`, `df -h`, `free -h`)  
4. Create `docs/migration-log.md` with timestamps  

Stop and ask the human if any access is missing.

---

## Phase 1 — Harden & base stack on VPS

Install and configure (Ubuntu Server preferred):

- Firewall (UFW): SSH, HTTP, HTTPS only  
- Fail2ban (basic SSH protection)  
- Prefer **Nginx** + PHP-FPM  
- PHP 8.x (match Namecheap site; prefer 8.1+)  
- MySQL or MariaDB  
- Composer / WP-CLI  
- Certbot (Let’s Encrypt) — after domain points, or temporary IP/staging first  
- Modest swap if needed during large import  

Create web root, e.g. `/var/www/alldrivesauto`.

---

## Phase 2 — Export from Namecheap (while live site stays up)

1. Full files backup: `public_html` (or WP root), including `wp-content/uploads`  
2. Full MySQL dump of the WordPress database  
3. Note PHP version, cron jobs, email setup, SSL, redirects  
4. Transfer archives to VPS (rsync/SCP) — ~50 GB; resume if interrupted  

Keep Namecheap serving production the whole time.

---

## Phase 3 — Restore on VPS (staging on IP)

1. Unpack files into web root  
2. Import database  
3. Create DB user/password; update `wp-config.php`  
4. Set ownership/permissions  
5. Nginx vhost for `http://162.35.185.121` and/or staging hostname  
6. WP-CLI `search-replace` only for staging URLs if needed (serialized-safe)  
7. Disable aggressive cache plugins until stable  
8. Smoke-test: home, shop, product, cart, checkout, admin, search  

---

## Phase 4 — Fix migration issues

Resolve until green: PHP extensions, permalinks, HTTPS mixed content, cron, memory/upload limits for WooCommerce, Mobex/plugin compatibility. Document each fix in `docs/migration-log.md`.

---

## Phase 5 — Pre-cutover checklist (agent prepares; human approves)

- [ ] Site works on VPS IP / staging URL  
- [ ] Admin works  
- [ ] Products/images load  
- [ ] Checkout page loads  
- [ ] Fresh Namecheap backup taken  
- [ ] Rollback plan written (DNS back to Namecheap; keep Namecheap online 48h+)  
- [ ] DNS TTL lowered if possible (e.g. 300s)  

**STOP.** Ask human: “Approve DNS cutover to `162.35.185.121`?”

---

## Phase 6 — DNS cutover (only after approval)

1. Point A/AAAA (and www) to **162.35.185.121**  
2. Let’s Encrypt for `alldrivesauto.com` + `www`  
3. Force HTTPS; final `siteurl` / `home` check  
4. Monitor 30–60 minutes  

If broken: roll DNS back to Namecheap immediately and report.

---

## Phase 7 — Post-cutover

1. Keep Namecheap as fallback ~48–72 hours  
2. Confirm InterServer backup options; enable if available  
3. Optional later (separate prompt): **Meilisearch** + Woo search  

---

## Kickoff prompt (when secrets are ready)

On https://cursor.com/agents with repo `Trudyranks/ada-product-search`:

```text
Let the Cloud Agent handle the entire migration.
Read CLOUD_AGENT_BRIEF.md and execute Phases 0–7 for InterServer.
VPS IP is 162.35.185.121 (VPS#3516118). Secrets are in Cloud Agent Secrets.
Do not change public DNS until I explicitly approve Phase 6.
Log progress in docs/migration-log.md.
```

---

## Out of scope unless asked

- Paying for hosting  
- Mass product scraping/import during migration  
- Meilisearch (default: after cutover)  
- Phone/voice AI  
- Installing cPanel (not required)

---

## Success definition

Public https://alldrivesauto.com serves from InterServer VPS (`162.35.185.121`) with working shop/admin, SSL valid, Namecheap still available for rollback, and migration log in this repo.
