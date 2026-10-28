# Deploy and Host Frappe Helpdesk on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/helpdesk?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=helpdesk)

[Frappe Helpdesk](https://frappe.io/helpdesk) is the open-source customer support desk from the makers of ERPNext: email tickets, SLAs with first-response and resolution targets, canned responses, assignment rules, a customer portal and a knowledge base, in a modern Vue agent UI. No per-agent fees. This template runs it the way `bench setup production` runs it on a server, with nightly backups and upgrades handled for you.

## About Hosting Frappe Helpdesk

The stack is three pieces: Helpdesk, a private MariaDB, and a Railway bucket for backups.

- **Everything Frappe runs, in one service.** nginx on the public port, gunicorn, the socket.io realtime server (live ticket updates for agents), background workers, the scheduler (SLA timers, email pulls), and Redis for the cache and job queue. They all need the same `sites/` folder, and a Railway volume attaches to one service, so they run together.
- **Locked down from the first second.** The site is created on first boot with a generated `Administrator` password, never `admin`/`admin`. MariaDB has no public proxy, and the realtime server only accepts connections from your own domain.
- **Nightly backups.** Every night at 03:00 UTC the database, the attachments and the site config go to the bundled bucket, one tar per weekday, so the last 7 days are always there. `erpnext-restore` puts one back, into this deployment or a new one.
- **Upgrades that migrate themselves.** The first boot of a new version backs up the database and runs `bench migrate` before serving the site.

## Common Use Cases

- A Zendesk or Freshdesk replacement for a support team, with no per-agent fees
- Turning a shared support@ inbox into tickets with SLAs, assignment and canned replies
- A customer portal and public knowledge base on your own domain

## Dependencies for Frappe Helpdesk Hosting

- MariaDB 11.8 (included, private network only)
- A Railway bucket for backups (included)

### Deployment Dependencies

- [Helpdesk documentation](https://docs.frappe.io/helpdesk)
- [Template source on GitHub](https://github.com/two-workspaces/erpnext)

### Implementation Details

**Sign in** at your service's Railway domain as `Administrator` with the `ERPNEXT_ADMIN_PASSWORD` value from the Helpdesk service's Variables tab. The first deploy builds the image and creates the site, so give it about five minutes. The agent UI is at `/helpdesk`; customers open tickets in the portal at the same address. Change the password afterwards; the variable is only read on first boot. Then set your time zone at `/app/system-settings`, since SLA due times follow it.

**Email.** Tickets come in by email: add your support mailbox under Helpdesk → Admin → Email Accounts, and incoming mail is pulled over IMAP on every plan. Railway only allows outbound SMTP on the Pro plan, so on other plans agents' replies can't be emailed to customers; use an HTTP mail relay, or upgrade.

**Memory.** About 400 MB for Helpdesk and 250 MB for MariaDB. On plans that give a service less than 2 GB, the template runs 1 web worker (4 threads) and 1 background worker. For a larger team, raise `ERPNEXT_WEB_WORKERS` and `ERPNEXT_BG_WORKERS`.

**Backups and restore.** Backups appear in the Backups bucket as `erpnext-Mon.tar` … `erpnext-Sun.tar` (the image is shared with our ERPNext template). For an extra one, run `erpnext-backup` in a `railway ssh` session on the Helpdesk service. To restore, run `erpnext-restore Mon` there, then restart the service. It brings over the site's encryption key, so saved mailbox passwords keep working in a new deployment too.

**Custom domain.** Add it in the service's Settings → Networking, then set `ERPNEXT_URL` to `https://your.domain` so links in ticket emails use it.

## Why Deploy Frappe Helpdesk on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying Frappe Helpdesk on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
