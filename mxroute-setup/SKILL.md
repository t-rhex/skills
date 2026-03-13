---
name: mxroute-setup
description: |
  MXroute email hosting setup and configuration guide. Use when the user needs help setting up
  MXroute email accounts, configuring email clients (iOS, Outlook, Thunderbird), adding domains,
  creating forwarders, setting up CalDAV/CardDAV, custom hostnames, or migrating to MXroute.
---

# MXroute Setup & Configuration

You are an expert on MXroute email hosting. Help the user set up and configure their MXroute service using the reference below.

## Two Panels

- **Management Panel** (management.mxroute.com): Subscriptions, invoices, payment methods, plan upgrades, support tickets, and login to Control Panel.
- **Control Panel** (panel.mxroute.com): Email accounts, forwarders, spam filtering, domain/DNS management, CalDAV/CardDAV, webmail access, SSL certificates.

## Quick Setup Steps

1. Log into Management Panel → click "Login to Panel" to access Control Panel
2. Navigate to **Domains** → Add your domain (requires DNS verification TXT record first)
3. Configure DNS records at your domain registrar (see DNS section)
4. Create email accounts under **Email Accounts** → Create Account (set username, password, quota)
5. Configure email clients using server settings below

## DNS Records Required

### MX Records
| Type | Name | Priority | Value |
|------|------|----------|-------|
| MX   | @    | 10       | `(your-server).mxrouting.net` |
| MX   | @    | 20       | `(your-server)-relay.mxrouting.net` |

Server name is found in Control Panel under **DNS** section.

### SPF Record
| Type | Name | Value |
|------|------|-------|
| TXT  | @    | `v=spf1 include:mxroute.com -all` |

### DKIM Record
| Type | Name | Value |
|------|------|-------|
| TXT  | `x._domainkey` | Found in Control Panel → DNS → select domain → copy DKIM key |

### DMARC Record
| Type | Name | Value |
|------|------|-------|
| TXT  | `_dmarc` | `v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com` |

## Email Client Settings

**Server hostname**: Same as your primary MX record (e.g., `tuesday.mxrouting.net`)
**Username**: Full email address
**Password**: Email account password

| Protocol | Port | Encryption |
|----------|------|------------|
| IMAP     | 993  | SSL/TLS    |
| IMAP     | 143  | STARTTLS   |
| POP3     | 995  | SSL/TLS    |
| POP3     | 110  | STARTTLS   |
| SMTP     | 465  | SSL/TLS    |
| SMTP     | 587  | STARTTLS   |
| SMTP     | 2525 | STARTTLS   |

### iOS Mail Setup
1. Settings → Mail → Accounts → Add Account → Other → Add Mail Account
2. Enter name, email, password
3. Select IMAP
4. Incoming: `(your-server).mxrouting.net`, port 993, SSL ON
5. Outgoing: `(your-server).mxrouting.net`, port 465, SSL ON
6. Authentication: Password
7. Enable Mail only (disable Contacts, Calendars, Notes)

### Outlook Setup
1. File → Add Account (Windows) / Tools → Accounts → + (Mac)
2. Advanced/Manual setup → IMAP
3. Incoming: port 993, SSL/TLS
4. Outgoing: port 465, SSL/TLS
5. Username: full email address

### Thunderbird Setup
1. Account Settings → Add Mail Account
2. Enter credentials → Manual config
3. IMAP: port 993, SSL/TLS
4. SMTP: port 465, SSL/TLS

## Domain Management

### Adding a Domain
1. Find domain verification key in Control Panel under **DNS**
2. Add TXT record at your DNS provider with the verification key
3. Wait for DNS propagation (5-30 minutes)
4. Control Panel → Domains → Add Domain → enter domain name
5. Configure MX, SPF, DKIM records
6. Optionally remove verification TXT record after success

### Domain Aliases
Domain aliases share ALL email accounts and settings with the primary domain.
1. Control Panel → Domain Pointers → Add Domain Pointer
2. Enter source domain (alias) and target domain (primary)
3. Configure MX, SPF, DKIM DNS records for the alias domain (same values as primary)

### Catch-All
Control Panel → Advanced → select domain → forward unknown recipients to a specific address or reject them.

## Email Forwarders
1. Control Panel → Forwarders → Create Forwarder
2. Set forwarder address and destination address(es)
3. Supports multiple destinations
4. Avoid creating forwarding chains (causes loops)
5. Test forwarders after creation

## CalDAV & CardDAV

**Server**: `dav.mxroute.com`
**Port**: 443 (HTTPS)
**Auth**: Full email address + password
**Requirement**: Domain MX records must point to MXroute

### Platform Setup
- **iOS/iPadOS**: Settings → Calendar/Contacts → Accounts → Add CalDAV/CardDAV Account → server: `dav.mxroute.com`
- **macOS**: Calendar/Contacts app → Add Account → Other → Manual → `dav.mxroute.com`
- **Android**: Install DAVx5 → Base URL: `https://dav.mxroute.com`
- **Thunderbird**: Calendar tab → New Calendar → On the Network → `https://dav.mxroute.com`
- **Windows**: No native support; use Thunderbird or eM Client

## Custom Hostnames

Create CNAME records:
| Type  | Name    | Value |
|-------|---------|-------|
| CNAME | mail    | `(your-server).mxrouting.net` |
| CNAME | webmail | `(your-server).mxrouting.net` |

Then: Control Panel → SSL Certificates → select domain → verify DNS → check subdomains → request certificate.

Use `mail.yourdomain.com` for IMAP/SMTP and `webmail.yourdomain.com` for Roundcube webmail.

**Important**: Use CNAME records, not A records. Server IPs may change.

## Migration to MXroute

MXroute does not perform migrations. Use **imapsync**:

```bash
imapsync \
  --host1 source.server.com --user1 source@example.com --password1 'sourcepassword' \
  --host2 mail.mxroute.com --user2 destination@example.com --password2 'destinationpassword'
```

Available as: web interface, Windows app, or CLI (Linux).

### Migration Steps
1. Create destination accounts on MXroute first
2. Test with a non-critical account
3. Run imapsync during low-usage hours
4. Verify emails transferred
5. Update DNS MX records to MXroute
6. Configure SPF, DKIM, DMARC
7. Reconfigure all devices
8. Keep old service running during transition

## Spam Filtering

**Expert Spam Filtering** (enabled by default):
- Blocks spam from suspicious IP ranges
- Requires SMTP auth from suspicious IPs
- Toggle: Management Panel → Login to Panel → Spam Filters → Advanced
- Changes take effect within 5 minutes
- Cannot disable if forwarding to Gmail, Microsoft, Yahoo, AOL, or T-Online
- Whitelist requests: https://whitelistrequest.mxrouting.net

## Service Limits
- **Outbound**: 400 emails per hour per email address
- **Marketing emails**: ABSOLUTELY NOT allowed — instant suspension
- **Allowed**: Business correspondence, transactional emails, personal email
- **No refunds** for policy violations
- **No SLA** — but high reliability
- **Support**: Ticket-based only, typically 4-6 hour response

## Important Notes
- DNS changes can take 24-48 hours to propagate (often faster)
- Use the built-in DNS Tester in Control Panel to verify records
- No true 2FA available for email protocols (IMAP/SMTP don't support it)
- Use strong, unique passwords
- Webmail clients: Crossbox and Roundcube (accessible via Control Panel)
