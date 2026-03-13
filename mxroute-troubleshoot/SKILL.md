---
name: mxroute-troubleshoot
description: |
  MXroute email troubleshooting guide. Use when the user has issues with MXroute email delivery,
  spam folder problems (Gmail, Microsoft/Hotmail/Outlook), authentication errors, common error
  messages, DNS configuration issues, or connection problems.
---

# MXroute Troubleshooting

You are an expert on diagnosing and resolving MXroute email issues. Walk the user through systematic troubleshooting using the reference below.

## Diagnostic Steps (Always Start Here)

1. **Test email config**: Send a test email to https://mail-tester.com — aim for 10/10 score
2. **Check DNS**: Use the built-in DNS Tester in Control Panel (panel.mxroute.com)
3. **Verify records**: MX, SPF (`v=spf1 include:mxroute.com -all`), DKIM (`x._domainkey`), DMARC
4. **Check for duplicates**: Ensure no duplicate SPF records exist
5. **Check propagation**: DNS changes can take up to 48 hours

## Common Error Messages

### 550 Authentication Required
- **Cause**: Sending email without proper SMTP authentication
- **Fix**: Ensure email client has SMTP authentication enabled with full email address as username and correct password

### Sender Verify Failed
- **Cause**: Server cannot verify the FROM address is valid
- **Fix**: Ensure the sending address exists on the MXroute server and matches the authenticated account

### 550 5.7.515 Access Denied - Domain Authentication (Microsoft)
- **Cause**: Microsoft rejecting due to failed SPF, DKIM, or DMARC
- **Fix**: Most commonly a missing or incorrect DKIM record. Check Control Panel → DNS → copy DKIM key → add as TXT record for `x._domainkey`

### No Such Recipient Here
- **Cause**: Sending to an address the server thinks should exist locally but doesn't
- **Fix**: Common during domain migrations. Verify the recipient account exists in Control Panel → Email Accounts

### Address Blocked by Admins
- **Cause**: Sending or recipient address blocked due to policy violations or abuse
- **Fix**: Contact MXroute support with details

## Emails Going to Spam — Microsoft (Hotmail/Outlook/Office 365)

Microsoft has notoriously strict filtering. This is industry-wide, not MXroute-specific.

### Steps to Fix
1. **Get a 10/10 on mail-tester.com** before doing anything else
2. **Verify DNS**: SPF, DKIM, and DMARC all properly configured
3. **Ask recipients to**:
   - Mark your email as "Not Junk" (use the button, don't just move it)
   - Add your sender address to their contacts
4. **Email content best practices**:
   - Good text-to-image ratio
   - Avoid spam trigger words
   - Natural person-to-person communication style
   - Natural sending patterns (don't send many emails at once)
5. **MXroute manages** server IP reputation via Microsoft's Smart Network Data Services

### If issues persist
Submit a support ticket with: mail-tester score, affected domain, email headers, and steps already taken.

## Emails Going to Spam — Gmail

### Steps to Fix
1. **Get 10/10 on mail-tester.com**
2. **Verify DNS**: SPF, DKIM, DMARC properly configured
3. **Train Gmail's filters** — ask recipients to:
   - Mark messages as "Not spam"
   - Add your address to contacts before receiving emails
   - Reply to/engage with your messages
4. **Monitor with Google Postmaster Tools**:
   - Track domain reputation
   - Check spam rates and delivery errors
   - Review authentication results
5. **Follow Gmail's requirements**:
   - Authenticate with SPF and DKIM
   - Avoid sudden volume spikes
   - Keep spam complaint rates below 0.1%
   - Maintain consistent sending patterns
6. **Content quality**:
   - Avoid spam trigger words and excessive promotional language
   - Remove inactive subscribers and invalid addresses
   - Build volume gradually for new domains

## Connection Issues

### Cannot Connect to Server
- Verify server hostname (found in Control Panel → DNS)
- Check ports: IMAP 993 (SSL), SMTP 465 (SSL), or 587/2525 (STARTTLS)
- ISP may be blocking ports — try alternative ports (2525 for SMTP)
- Verify internet connectivity
- Check correct date/time on device

### Authentication Failures
- Username must be the full email address
- Verify password is correct (reset in Control Panel → Email Accounts if needed)
- Ensure IMAP/SMTP is not disabled for the account

### SSL Certificate Warnings
- If using custom hostnames, ensure SSL certificate was requested in Control Panel
- CNAME records must be properly configured before requesting certificates
- Wait for DNS propagation if recently changed

## DNS Issues

### SPF Not Passing
- Must be exactly: `v=spf1 include:mxroute.com -all`
- Only ONE SPF record allowed per domain (merge if needed)
- Check for conflicting SPF records

### DKIM Not Passing
- Copy key exactly from Control Panel → DNS → select domain
- Record name must be `x._domainkey`
- Key is unique per domain
- Some DNS providers require removing quotes from the value

### DMARC Not Working
- Record name: `_dmarc`
- Minimum: `v=DMARC1; p=none;`
- Recommended: `v=DMARC1; p=quarantine; rua=mailto:postmaster@yourdomain.com`

### MX Records Not Resolving
- Verify correct server name from Control Panel
- Priority 10 for primary, 20 for relay
- Check propagation at whatsmydns.net

## Expert Spam Filtering Issues

- Enabled by default — blocks spam from suspicious IPs
- **Cannot disable** if forwarding to Gmail, Microsoft, Yahoo, AOL, T-Online
- If legitimate mail is being blocked, submit whitelist request at https://whitelistrequest.mxrouting.net
- Changes take effect within 5 minutes
- Toggle: Management Panel → Login to Panel → Spam Filters → Advanced

## Migration Issues

| Problem | Fix |
|---------|-----|
| Connection timeouts | Check firewall and IMAP access on source server |
| Missing emails | Verify folder structures match; use imapsync mapping options |
| Authentication failures | Confirm credentials; ensure IMAP enabled on source |
| Slow migration | Split large mailboxes into smaller batches |

## When to Contact Support

Open a ticket at management.mxroute.com with:
- Complete error message and timestamp
- mail-tester.com score
- Affected domain and email addresses
- Email headers (if delivery issue)
- Steps already attempted

Response time: typically 4-6 hours, within 24 hours.
