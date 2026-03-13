---
name: cloudflare-dns
description: |
  Cloudflare DNS configuration guide. Use when the user needs help managing DNS records in
  Cloudflare — adding domains, creating/editing DNS records (A, AAAA, CNAME, MX, TXT, SRV, etc.),
  configuring email records (SPF, DKIM, DMARC, MX), understanding proxy vs DNS-only, CNAME
  flattening, nameserver setup, or troubleshooting DNS issues on Cloudflare.
---

# Cloudflare DNS Configuration

You are an expert on Cloudflare DNS management. Help the user configure DNS records correctly.

## Adding a Domain to Cloudflare (Full Setup)

1. **Add domain** in Cloudflare dashboard — must be a top-level domain (e.g., `example.com`)
2. **Review DNS records** — Cloudflare auto-scans existing records; verify zone apex, subdomains, and email records
3. **Change nameservers** at your registrar to the two Cloudflare nameservers shown in the dashboard Overview
4. **Wait for propagation** — up to 24 hours; you'll get an email when active
5. **Re-enable DNSSEC** after nameservers propagate

**Warning**: Missing DNS records after migration causes `DNS_PROBE_FINISHED_NXDOMAIN` errors.

## Proxy Status (Orange Cloud vs Grey Cloud)

### Proxied (Orange Cloud)
- Traffic routes through Cloudflare's network
- DDoS protection, caching, WAF, optimization active
- Origin IP hidden from public
- TTL locked to Auto (300s)
- CNAME records are flattened
- **Only A, AAAA, and CNAME records can be proxied**

### DNS-Only (Grey Cloud)
- DNS resolves directly to origin IP
- No Cloudflare protection or caching
- Origin IP exposed
- Custom TTL allowed
- Required for non-HTTP services (mail, FTP, SSH, etc.)

### When to Use Each

| Use Case | Proxy Status |
|----------|-------------|
| Web traffic (HTTP/HTTPS) | Proxied (orange) |
| Mail server (`mail.` subdomain) | DNS-only (grey) |
| MX records | Always DNS-only |
| Email CNAME records (DKIM, etc.) | DNS-only (grey) |
| Third-party domain verification | DNS-only (grey) |
| SSH, FTP, game servers | DNS-only (grey) |
| API endpoints (want protection) | Proxied (orange) |

**Critical rule**: Mail-related A/AAAA/CNAME records MUST be DNS-only. Cloudflare does not proxy SMTP (port 25) unless you have Spectrum.

## Creating DNS Records (Dashboard)

1. Go to **DNS** > **Records** in your Cloudflare dashboard
2. Click **Add record**
3. Select record type, fill in fields
4. Set proxy status (if applicable)
5. Click **Save**

## DNS Record Types Reference

### A Record — Points domain to IPv4 address
```
Type: A    Name: @           Content: 203.0.113.1    Proxy: On/Off    TTL: Auto
Type: A    Name: mail        Content: 203.0.113.1    Proxy: OFF       TTL: Auto
Type: A    Name: subdomain   Content: 203.0.113.1    Proxy: On        TTL: Auto
```
- Cannot use a Cloudflare IP as the content
- For mail subdomains, proxy MUST be off

### AAAA Record — Points domain to IPv6 address
```
Type: AAAA    Name: @    Content: 2001:db8::1    Proxy: On/Off    TTL: Auto
```
- Same rules as A records

### CNAME Record — Alias one domain to another
```
Type: CNAME    Name: www       Content: example.com          Proxy: On/Off
Type: CNAME    Name: mail      Content: mail.provider.com    Proxy: OFF
Type: CNAME    Name: blog      Content: user.github.io       Proxy: On
```
- Proxied CNAMEs are flattened (resolved to IP in response)
- For domain verification CNAMEs, use DNS-only to avoid flattening issues
- CNAME at zone apex (`@`) works on Cloudflare via CNAME flattening

### MX Record — Routes email to mail servers
```
Type: MX    Name: @    Mail server: mail.example.com       Priority: 10
Type: MX    Name: @    Mail server: backup.example.com     Priority: 20
```
- Lower priority number = higher preference
- Always DNS-only (cannot be proxied)
- Must point to a hostname, not an IP

### TXT Record — Text data for verification, email auth, etc.
```
Type: TXT    Name: @              Content: "v=spf1 include:_spf.google.com -all"
Type: TXT    Name: _dmarc         Content: "v=DMARC1; p=quarantine; rua=mailto:..."
Type: TXT    Name: x._domainkey   Content: "v=DKIM1; k=rsa; p=MIIBIjAN..."
```
- Max 2,048 characters per record
- Max 8,192 characters combined for records with same name
- Used for SPF, DKIM, DMARC, domain verification, etc.

### SRV Record — Service location
```
Type: SRV    Name: _service._proto    Priority: 10    Weight: 5    Port: 443    Target: server.example.com
```
- Common for VOIP, XMPP, CalDAV/CardDAV autodiscovery
- Cannot be proxied

### CAA Record — Certificate authority authorization
```
Type: CAA    Name: @    Tag: issue    CA domain: letsencrypt.org
```
- Controls which CAs can issue certificates for your domain
- Important if using non-Cloudflare SSL certificates

### NS Record — Nameserver delegation
```
Type: NS    Name: subdomain    Nameserver: ns1.provider.com
```
- Used for subdomain delegation to external DNS
- Max 10 NS records per delegation; best practice is 7 or fewer

### PTR Record — Reverse DNS
```
Type: PTR    Name: (IP octet)    Content: hostname.example.com
```
- Used in reverse DNS zones

## Email DNS Configuration

### Common Email Providers

#### Google Workspace
```
MX    @    aspmx.l.google.com         Priority: 1
MX    @    alt1.aspmx.l.google.com    Priority: 5
MX    @    alt2.aspmx.l.google.com    Priority: 5
MX    @    alt3.aspmx.l.google.com    Priority: 10
MX    @    alt4.aspmx.l.google.com    Priority: 10
TXT   @    v=spf1 include:_spf.google.com -all
```
Plus DKIM from Google Admin console → `google._domainkey`

#### Microsoft 365
```
MX    @    (your-domain).mail.protection.outlook.com    Priority: 0
TXT   @    v=spf1 include:spf.protection.outlook.com -all
```
Plus CNAME records for autodiscover and DKIM selectors from Microsoft admin

#### MXroute
```
MX    @    (server).mxrouting.net           Priority: 10
MX    @    (server)-relay.mxrouting.net     Priority: 20
TXT   @    v=spf1 include:mxroute.com -all
TXT   x._domainkey    (DKIM key from MXroute Control Panel)
TXT   _dmarc    v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com
```

#### Cloudflare Email Routing (receive-only forwarding)
```
MX    @    route1.mx.cloudflare.net    Priority: 69
MX    @    route2.mx.cloudflare.net    Priority: 31
MX    @    route3.mx.cloudflare.net    Priority: 84
TXT   @    v=spf1 include:_spf.mx.cloudflare.net -all
```
Set up via **Email** > **Email Routing** in dashboard.

### SPF Records
```
TXT    @    v=spf1 include:provider.com -all
```
- **Only ONE SPF record per domain** — merge multiple providers:
  `v=spf1 include:mxroute.com include:_spf.google.com -all`
- `-all` = hard fail (recommended), `~all` = soft fail, `?all` = neutral
- Too many DNS lookups (>10) causes SPF permerror

### DKIM Records
```
TXT    selector._domainkey    v=DKIM1; k=rsa; p=MIIBIjAN...
```
- Selector name varies by provider (e.g., `x`, `google`, `selector1`)
- Must be DNS-only (grey cloud)
- Long keys may need splitting across multiple strings

### DMARC Records
```
TXT    _dmarc    v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com
```
- `p=none` — monitor only (start here)
- `p=quarantine` — send failures to spam
- `p=reject` — reject failures outright
- `rua=` — where to send aggregate reports
- `ruf=` — where to send forensic reports

## CNAME Flattening

- Cloudflare resolves CNAME chains and returns the final IP address
- Enables CNAME at zone apex (`@`) which normally isn't allowed in DNS
- Happens automatically for proxied CNAMEs
- Can cause issues with **domain verification** (verifier expects a CNAME response, gets an A response)
- **Fix**: Set the verification CNAME to DNS-only (grey cloud), or temporarily disable flattening

## Common Cloudflare DNS Issues

### Email not working after adding domain to Cloudflare
- MX records missing or wrong — re-add them
- Mail subdomain (`mail.example.com`) is proxied — set to DNS-only (grey cloud)
- SPF/DKIM records not migrated — add TXT records

### Website shows Cloudflare error after DNS change
- A/AAAA record pointing to wrong IP
- SSL mode mismatch (set to Full Strict if origin has valid cert)
- DNS not yet propagated — wait up to 24 hours

### Third-party verification failing
- CNAME flattening returning IP instead of CNAME — set record to DNS-only
- TXT record not propagated yet
- Record added to wrong name (e.g., `@` vs specific subdomain)

### "Too many redirects" error
- SSL/TLS mode set to Flexible but origin forces HTTPS — change to Full or Full (Strict)
- Conflicting redirect rules

### SPF permerror (too many lookups)
- SPF allows max 10 DNS lookups
- Flatten nested includes or use an SPF flattening service
- Remove unused `include:` directives

## Useful Tools

- **Cloudflare DNS Tester**: Dashboard → DNS → check record propagation
- **External**: `dig example.com MX`, `nslookup`, whatsmydns.net, mxtoolbox.com
- **Email testing**: mail-tester.com (send test email, get 10/10 score)

## Quick Checklist for Email Setup on Cloudflare

- [ ] MX records added (correct hostnames and priorities)
- [ ] SPF TXT record at `@` (single record, merged if multiple senders)
- [ ] DKIM TXT record(s) with correct selector name
- [ ] DMARC TXT record at `_dmarc`
- [ ] Mail subdomain A/CNAME record set to DNS-only (grey cloud)
- [ ] No conflicting/duplicate SPF records
- [ ] Test with mail-tester.com → aim for 10/10
