# Cloud Agent brief â€” InterServer migration

**Mission:** Handle the **entire** WordPress/WooCommerce migration of https://alldrivesauto.com from Namecheap shared hosting to the **InterServer** VPS, end-to-end, with zero surprise DNS cutover.

Cloud Agents do **not** see local Cursor chats. This file is the source of truth in the repo.

**Repo:** `Trudyranks/ada-product-search` Â· branch `main`  
**Kickoff phrase:** *Let the Cloud Agent handle the entire migration.*

---

## Current status (2026-07-25) â€” VPS ready; Phase 1 done

| Item | Value |
|------|--------|
| Live site | https://alldrivesauto.com |
| Current host | Namecheap shared / cPanel (keep live until Phase 6 approved) |
| Namecheap account user | `Trudale1` (password in Cloud Agent Secrets; may need email 2FA) |
| New host | **InterServer Premium VPS** â€” paid & Active |
| Service ID | **VPS#3516118** |
| Hostname | `vps3516118` / PTR `vps3516118.trouble-free.net` |
| Public IP | **162.35.185.121** |
| Package | KVM Linux VPS Slice (**Ubuntu 8 Slices**) Â· **$24/mo** Â· NJ USA |
| Panel | https://my.interserver.net/view_vps?id=3516118 |
| VPS OS | **Ubuntu 24.04.4 LTS** (reinstalled clean) |
| Phase 1 | **DONE** â€” Nginx, PHP 8.3-FPM, MariaDB, UFW, Fail2ban, WP-CLI, Composer; web root `/var/www/alldrivesauto`; http://162.35.185.121/ returns 200 |
| Approx data | ~50 GB files + database |
| Stack | WordPress + WooCommerce (Mobex theme) + existing plugins |

### CRITICAL â€” transfer must NOT use the humanâ€™s PC

The human is on a **limited mobile/data plan**. **Do not** download the ~50 GB site to a laptop/desktop or stream large archives through the userâ€™s browser.

**Required copy path:** Namecheap (cPanel/SFTP/SSH) **â†’ directly â†’** InterServer VPS (`162.35.185.121`), using the **Cloud Agentâ€™s** network (or VPS pulling from Namecheap).

Preferred methods (pick what Namecheap allows):

1. On the VPS: `rsync` / `lftp` / `wget` / `scp` **pull** from Namecheap SFTP/SSH  
2. Or: create a full backup in Namecheap cPanel and have the **VPS download** the backup URL/archive  
3. DB: `mysqldump` on Namecheap (or phpMyAdmin export) streamed/pulled to the VPS â€” not saved on the userâ€™s phone/PC  

Log transfer progress in `docs/migration-log.md`. Resume if interrupted.

### SSH status

- `root@162.35.185.121` â€” **working** (password in Cloud Agent Secrets as `VPS_ROOT_PASSWORD`)  
- **Never commit passwords** to git.

---

## Secrets checklist (Cloud Agent Secrets / kickoff prompt)

Put these in Cursor **Cloud Agent Secrets** before kickoff (or paste once in the prompt). Do **not** commit to the repo.

| Secret key (suggested) | What it is | Status |
|------------------------|------------|--------|
| `VPS_IP` | `162.35.185.121` | Known â€” can hardcode or secret |
| `VPS_USER` | `root` | Known |
| `VPS_ROOT_PASSWORD` | Working root password (`AdaVps2026!SecureRoot` as of 2026-07-25) | **Ready** |
| `INTERSERVER_PANEL_URL` | https://my.interserver.net/view_vps?id=3516118 | Known |
| `INTERSERVER_ACCOUNT_PASSWORD` | my.interserver.net login | Available in operator secrets |
| `NAMECHEAP_USER` | Namecheap account `Trudale1` | Ready |
| `NAMECHEAP_PASSWORD` | Namecheap account password | Ready (in secrets) |
| `NAMECHEAP_CPANEL_URL` | https://business55.web-hosting.com:2083 | Ready |
| `NAMECHEAP_CPANEL_HOST` | `business55.web-hosting.com` | Ready |
| `NAMECHEAP_CPANEL_USER` | cPanel user **`alldrnmk`** (home `/home/alldrnmk`) | Ready |
| `NAMECHEAP_CPANEL_PASSWORD` | Try Namecheap account password; reset in cPanel if SFTP fails | Ready |
| `NAMECHEAP_SFTP_HOST` / port | Host `business55.web-hosting.com` â€” SSH/SFTP often port **21098** on Namecheap | Confirm in cPanel â†’ SSH Access |

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

## Phase 0 â€” Kickoff checks

1. Confirm SSH to `root@162.35.185.121` works (fix password via panel/reinstall if needed)  
2. Confirm Namecheap/cPanel or SFTP + MySQL dump access works  
3. Record VPS public IP, disk free space, Ubuntu version (`uname -a`, `df -h`, `free -h`)  
4. Create `docs/migration-log.md` with timestamps  

Stop and ask the human if any access is missing.

---

## Phase 1 â€” Harden & base stack on VPS

Install and configure (Ubuntu Server preferred):

- Firewall (UFW): SSH, HTTP, HTTPS only  
- Fail2ban (basic SSH protection)  
- Prefer **Nginx** + PHP-FPM  
- PHP 8.x (match Namecheap site; prefer 8.1+)  
- MySQL or MariaDB  
- Composer / WP-CLI  
- Certbot (Letâ€™s Encrypt) â€” after domain points, or temporary IP/staging first  
- Modest swap if needed during large import  

Create web root, e.g. `/var/www/alldrivesauto`.

---

## Phase 2 â€” Export from Namecheap (while live site stays up)

1. Full files backup: `public_html` (or WP root), including `wp-content/uploads`  
2. Full MySQL dump of the WordPress database  
3. Note PHP version, cron jobs, email setup, SSL, redirects  
4. Transfer archives to VPS (rsync/SCP) â€” ~50 GB; resume if interrupted  

Keep Namecheap serving production the whole time.

---

## Phase 3 â€” Restore on VPS (staging on IP)

1. Unpack files into web root  
2. Import database  
3. Create DB user/password; update `wp-config.php`  
4. Set ownership/permissions  
5. Nginx vhost for `http://162.35.185.121` and/or staging hostname  
6. WP-CLI `search-replace` only for staging URLs if needed (serialized-safe)  
7. Disable aggressive cache plugins until stable  
8. Smoke-test: home, shop, product, cart, checkout, admin, search  

---

## Phase 4 â€” Fix migration issues

Resolve until green: PHP extensions, permalinks, HTTPS mixed content, cron, memory/upload limits for WooCommerce, Mobex/plugin compatibility. Document each fix in `docs/migration-log.md`.

---

## Phase 5 â€” Pre-cutover checklist (agent prepares; human approves)

- [ ] Site works on VPS IP / staging URL  
- [ ] Admin works  
- [ ] Products/images load  
- [ ] Checkout page loads  
- [ ] Fresh Namecheap backup taken  
- [ ] Rollback plan written (DNS back to Namecheap; keep Namecheap online 48h+)  
- [ ] DNS TTL lowered if possible (e.g. 300s)  

**STOP.** Ask human: â€œApprove DNS cutover to `162.35.185.121`?â€

---

## Phase 6 â€” DNS cutover (only after approval)

1. Point A/AAAA (and www) to **162.35.185.121**  
2. Letâ€™s Encrypt for `alldrivesauto.com` + `www`  
3. Force HTTPS; final `siteurl` / `home` check  
4. Monitor 30â€“60 minutes  

If broken: roll DNS back to Namecheap immediately and report.

---

## Phase 7 â€” Post-cutover

1. Keep Namecheap as fallback ~48â€“72 hours  
2. Confirm InterServer backup options; enable if available  
3. Optional later (separate prompt): **Meilisearch** + Woo search  

---

## Kickoff prompt (when secrets are ready)

On https://cursor.com/agents with repo `Trudyranks/ada-product-search`:

```text
Let the Cloud Agent handle the entire migration.
Read CLOUD_AGENT_BRIEF.md and execute Phases 0â€“7 for InterServer.
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
