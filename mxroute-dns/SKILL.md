---
name: mxroute-dns
description: |
  MXroute DNS record configuration helper. Use when the user needs help configuring DNS records
  for MXroute — MX records, SPF, DKIM, DMARC, CNAME for custom hostnames, domain verification,
  or diagnosing DNS-related email delivery issues.
---

# MXroute DNS Configuration

You are an expert on configuring DNS records for MXroute email hosting. Help the user set up correct DNS records.

## Important: Always ask the user for their server name first
The server name is found in the MXroute Control Panel (panel.mxroute.com) under the **DNS** section. It looks like `(servername).mxrouting.net` (e.g., `tuesday.mxrouting.net`, `fusion.mxrouting.net`).

## Complete DNS Record Set

For a domain `example.com` on server `tuesday.mxrouting.net`:

### MX Records (required — routes email to MXroute)
```
Type: MX    Name: @    Priority: 10    Value: tuesday.mxrouting.net
Type: MX    Name: @    Priority: 20    Value: tuesday-relay.mxrouting.net
```

### SPF Record (required — authorizes MXroute to send for your domain)
```
Type: TXT   Name: @    Value: v=spf1 include:mxroute.com -all
```

**Rules**:
- Only ONE SPF record per domain
- If you have other services sending email, merge them: `v=spf1 include:mxroute.com include:_spf.google.com -all`
- Use `-all` (hard fail) not `~all` (soft fail) for best deliverability

### DKIM Record (required — cryptographically signs your emails)
```
Type: TXT   Name: x._domainkey   Value: (copy from Control Panel → DNS → select domain)
```

**Notes**:
- Each domain has a unique DKIM key
- The key is long — copy it exactly
- Some DNS providers need quotes removed
- Selector is always `x`

### DMARC Record (strongly recommended — tells receivers how to handle auth failures)
```
Type: TXT   Name: _dmarc   Value: v=DMARC1; p=quarantine; rua=mailto:postmaster@example.com
```

**Policy options**:
- `p=none` — monitor only (good for initial setup)
- `p=quarantine` — send failures to spam (recommended)
- `p=reject` — reject failures entirely (strictest)

### Domain Verification (one-time, during domain addition)
```
Type: TXT   Name: @    Value: (verification key from Control Panel → DNS)
```
Can be removed after domain is successfully added.

### Custom Hostnames (optional — use your own domain for mail/webmail)
```
Type: CNAME   Name: mail      Value: tuesday.mxrouting.net
Type: CNAME   Name: webmail   Value: tuesday.mxrouting.net
```
After adding CNAMEs: Control Panel → SSL Certificates → select domain → verify → request certificate.

**Important**: Use CNAME, not A records. Server IPs may change.

### Domain Alias DNS
If `alias.com` is an alias of `primary.com`, configure the SAME MX, SPF, and DKIM records for `alias.com`.

## Verification

- Use the **DNS Tester** in Control Panel to validate MX, SPF, DKIM
- External tools: whatsmydns.net for propagation, mail-tester.com for full email test
- DNS propagation: typically 5-30 minutes, can take up to 48 hours

## Common DNS Mistakes

1. **Multiple SPF records** — only one TXT record starting with `v=spf1` allowed; merge them
2. **Wrong DKIM selector** — must be `x._domainkey`, not `default._domainkey` or others
3. **DKIM key truncated** — some DNS providers have character limits; use multiple strings if needed
4. **MX pointing to IP** — MX records must point to hostnames, not IP addresses
5. **Missing relay MX** — always add both priority 10 (primary) and priority 20 (relay)
6. **Using A record for custom hostname** — use CNAME so it follows server IP changes
7. **SPF with ~all instead of -all** — use hard fail (`-all`) for best results
8. **Forgetting DMARC** — without DMARC, receivers have no policy guidance for auth failures

## DNS Provider-Specific Notes

When helping users, ask which DNS provider they use. Common ones:
- **Cloudflare**: Proxy (orange cloud) must be OFF for MX/mail records; use DNS-only (grey cloud)
- **Namecheap**: Use "Advanced DNS" tab; TXT records go in the "Value" field
- **GoDaddy**: May need to remove default "parked" MX records first
- **Google Domains/Squarespace**: Straightforward TXT/MX record interface
- **Route53**: Use quoted strings for TXT records

## Quick Checklist for Users

Ask the user to confirm:
- [ ] Server name from Control Panel
- [ ] MX record (priority 10) pointing to `(server).mxrouting.net`
- [ ] MX record (priority 20) pointing to `(server)-relay.mxrouting.net`
- [ ] SPF: `v=spf1 include:mxroute.com -all` (single record, merged if needed)
- [ ] DKIM: `x._domainkey` TXT record with key from Control Panel
- [ ] DMARC: `_dmarc` TXT record with at least `v=DMARC1; p=quarantine;`
- [ ] No conflicting/duplicate records
- [ ] DNS Tester in Control Panel shows all green
- [ ] mail-tester.com score of 10/10
