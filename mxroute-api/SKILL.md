---
name: mxroute-api
description: |
  MXroute SMTP API helper. Use when the user needs help sending emails programmatically via
  MXroute's SMTP API, including code examples in PHP, Python, Node.js, or troubleshooting
  API errors.
---

# MXroute SMTP API

You are an expert on MXroute's SMTP API. Help the user send emails programmatically.

## Endpoint

**URL**: `https://smtpapi.mxroute.com/`
**Method**: POST
**Content-Type**: `application/json`

## Required Fields

| Field      | Description                              | Example                        |
|------------|------------------------------------------|--------------------------------|
| `server`   | MXroute hostname                         | `tuesday.mxrouting.net`        |
| `username` | SMTP username (full email address)       | `user@example.com`             |
| `password` | Account password                         | `yourpassword`                 |
| `from`     | Sender email address                     | `user@example.com`             |
| `to`       | Recipient email address                  | `recipient@example.com`        |
| `subject`  | Email subject (plain text only)          | `Hello`                        |
| `body`     | Email body (HTML supported)              | `<h1>Hi</h1><p>Hello!</p>`    |

## Response Format

**Success**: `{"success": true, "message": "Email sent successfully."}`
**Failure**: `{"success": false, "message": "[error description]"}`

Common errors: invalid server, missing required fields, invalid JSON, authentication failures.

## Limitations

- **Single recipient per request** — to send to multiple recipients, make multiple requests
- **No file attachments** supported
- **Body size limit**: ~10MB
- **Subject**: Plain text only (no HTML)
- **HTTPS required** for all requests
- **Rate limiting**: 400 emails per hour per email address; implement exponential backoff for retries

## Security Best Practices

- NEVER hardcode credentials — use environment variables
- Always use HTTPS
- Validate email format and server hostnames before sending

## Code Examples

### Python
```python
import os
import requests

response = requests.post("https://smtpapi.mxroute.com/", json={
    "server": os.environ["MXROUTE_SERVER"],
    "username": os.environ["MXROUTE_USERNAME"],
    "password": os.environ["MXROUTE_PASSWORD"],
    "from": "sender@example.com",
    "to": "recipient@example.com",
    "subject": "Hello from MXroute",
    "body": "<h1>Hello!</h1><p>This is a test email.</p>"
})

result = response.json()
if result["success"]:
    print("Email sent successfully")
else:
    print(f"Error: {result['message']}")
```

### Node.js (using fetch)
```javascript
const payload = {
  server: process.env.MXROUTE_SERVER,
  username: process.env.MXROUTE_USERNAME,
  password: process.env.MXROUTE_PASSWORD,
  from: "sender@example.com",
  to: "recipient@example.com",
  subject: "Hello from MXroute",
  body: "<h1>Hello!</h1><p>This is a test email.</p>"
};

const response = await fetch("https://smtpapi.mxroute.com/", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify(payload)
});

const result = await response.json();
console.log(result.success ? "Sent!" : result.message);
```

### PHP
```php
<?php
$data = [
    "server"   => getenv("MXROUTE_SERVER"),
    "username" => getenv("MXROUTE_USERNAME"),
    "password" => getenv("MXROUTE_PASSWORD"),
    "from"     => "sender@example.com",
    "to"       => "recipient@example.com",
    "subject"  => "Hello from MXroute",
    "body"     => "<h1>Hello!</h1><p>This is a test email.</p>"
];

$ch = curl_init("https://smtpapi.mxroute.com/");
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, ["Content-Type: application/json"]);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

$response = curl_exec($ch);
curl_close($ch);

$result = json_decode($response, true);
echo $result["success"] ? "Sent!\n" : "Error: " . $result["message"] . "\n";
```

## Troubleshooting API Issues

| Error | Cause | Fix |
|-------|-------|-----|
| Authentication failure | Wrong credentials | Verify username (full email) and password; test login via webmail first |
| Invalid server | Wrong hostname | Check Control Panel, DNS for correct server name |
| Missing required fields | Incomplete JSON | Ensure all 7 fields are present |
| Invalid JSON | Malformed request body | Validate JSON syntax before sending |
| Rate limited | Exceeded 400/hour | Implement exponential backoff; spread sends across time |

## When NOT to Use This API

- **Marketing/promotional emails** — prohibited, causes account suspension
- **Bulk sending** — MXroute is not for mass email
- **Attachments** — not supported; use direct SMTP with a library instead
- **Multiple recipients** — make separate API calls per recipient
