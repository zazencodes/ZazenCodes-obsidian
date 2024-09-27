---
date: 2024-09-10
tags:
    - code-snip
hubs:
    - "[[python]]"
---

# Test SMTP send email

```python
# Script: test_smtp_send_email.py
# Usage: python test_smtp_send_email.py --help

import smtplib
import email.message


def main(
    smtp_host,
    smtp_starttls,
    smtp_ssl,
    smtp_user,
    smtp_password,
    smtp_port,
    smtp_mail_from,
    smtp_mail_to,
):

    msg = email.message.EmailMessage()
    msg["From"] = smtp_mail_from
    msg["To"] = smtp_mail_to
    msg["Subject"] = "Test Email"
    msg.set_content("This is a test email.")

    # Create SMTP connection
    if smtp_ssl:
        with smtplib.SMTP_SSL(smtp_host, smtp_port) as server:
            server.login(smtp_user, smtp_password)
            server.send_message(msg)
    else:
        with smtplib.SMTP(smtp_host, smtp_port) as server:
            if smtp_starttls:
                server.starttls()
            server.login(smtp_user, smtp_password)
            server.send_message(msg)


if __name__ == "__main__":
    import argparse

    description = r"""
    Send a test email using SMTP, e.g.

    python
        --smtp_host your_smtp_host
        --smtp_user your_smtp_user
        --smtp_password your_smtp_password
        --smtp_mail_from your_email
        --smtp_mail_to recipient_email
    """
    parser = argparse.ArgumentParser(description=description)
    parser.add_argument("--smtp_host", required=True, help="SMTP host")
    parser.add_argument("--smtp_starttls", action="store_true", help="Use STARTTLS")
    parser.add_argument("--smtp_ssl", action="store_true", help="Use SSL")
    parser.add_argument("--smtp_user", required=True, help="SMTP username")
    parser.add_argument("--smtp_password", required=True, help="SMTP password")
    parser.add_argument("--smtp_port", type=int, default=587, help="SMTP port")
    parser.add_argument("--smtp_mail_from", required=True, help="Sender email address")
    parser.add_argument("--smtp_mail_to", required=True, help="Recipient email address")
    args = parser.parse_args()

    # Pass arguments to the main function
    main(
        args.smtp_host,
        args.smtp_starttls,
        args.smtp_ssl,
        args.smtp_user,
        args.smtp_password,
        args.smtp_port,
        args.smtp_mail_from,
        args.smtp_mail_to,
    )

```

