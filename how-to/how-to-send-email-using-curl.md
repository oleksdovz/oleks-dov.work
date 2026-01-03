# How to Send Email Using curl in Your DevOps Workflows

Whether you're automating alerts, sending logs, or testing mail services, curl can be a powerful tool for sending email directly 
from the CLI—especially handy in CI/CD pipelines or monitoring scripts.

✅ Prerequisites
You'll need:

- An SMTP server (Gmail, Mailgun, SendGrid, etc.)
- curl installed
- Authentication credentials (username/password or API key)
⚠️ Use app-specific passwords or tokens if 2FA is enabled on your email provider.



## Option 1: Using curl with SMTP (Plain Auth)

Here’s how to send a basic email via SMTP

```bash
curl --url 'smtps://smtp.gmail.com:465' \
  --ssl-reqd \
  --mail-from 'your.email@gmail.com' \
  --mail-rcpt 'recipient@example.com' \
  --upload-file email.txt \
  --user 'your.email@gmail.com:your_app_password'
```

`email.txt` contents:
```
To: recipient@example.com
From: your.email@gmail.com
Subject: Test email from curl

This is the body of the email.

```

# Option 2: Using API-based Providers (e.g., Mailgun, SendGrid)

More secure and flexible than raw SMTP.

Example: Send Email with Mailgun API

```bash
curl -s --user 'api:YOUR_API_KEY' \
  https://api.mailgun.net/v3/YOUR_DOMAIN_NAME/messages \
  -F from='Excited DevOps <mailgun@YOUR_DOMAIN_NAME>' \
  -F to=recipient@example.com \
  -F subject='Hello from curl!' \
  -F text='This is an email sent via Mailgun and curl.'

```

Example: Send Email with SendGrid API
```
curl --request POST \
  --url https://api.sendgrid.com/v3/mail/send \
  --header 'Authorization: Bearer YOUR_SENDGRID_API_KEY' \
  --header 'Content-Type: application/json' \
  --data '{
    "personalizations": [{"to": [{"email": "recipient@example.com"}]}],
    "from": {"email": "your.email@example.com"},
    "subject": "Hello from SendGrid & curl",
    "content": [{"type": "text/plain", "value": "This email was sent using curl and SendGrid."}]
  }'

```


# Pro Tip
Automate email notifications in scripts:

```
#!/bin/bash
ALERT_MSG="Disk usage above threshold!"
echo -e "Subject: Alert!\n\n$ALERT_MSG" > alert.txt

curl --url 'smtps://smtp.example.com:465' \
  --ssl-reqd \
  --mail-from 'monitor@example.com' \
  --mail-rcpt 'devops@example.com' \
  --upload-file alert.txt \
  --user 'monitor@example.com:your_password'

```


# Security Tip

Don’t hardcode passwords in scripts — use environment variables or secret managers.
Always prefer API-based sending when possible (better logging, easier error handling, more secure).

# Wrap Up

With curl, you can integrate email notifications into almost any DevOps task — fast and scriptable. Whether for alerts, test messages, or automation, this tool is small but mighty.