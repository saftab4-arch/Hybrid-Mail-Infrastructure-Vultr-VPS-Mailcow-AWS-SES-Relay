# Hybrid Mail Infrastructure: Vultr VPS + Mailcow + AWS SES Relay

## Project Overview

In this project, I built a production-style hybrid mail infrastructure using:

* Vultr VPS
* Ubuntu Linux
* Docker
* Mailcow
* AWS SES SMTP Relay
* Cloudflare DNS

The goal was to build a secure self-hosted mail platform capable of:

* Receiving inbound email locally on a VPS
* Sending outbound email securely through AWS SES
* Passing SPF, DKIM, and DMARC validation
* Achieving successful Gmail inbox delivery

This project simulates a real-world enterprise mail architecture where inbound and outbound email responsibilities are separated for better reliability and deliverability.

---

# Why This Project Matters

Modern organizations rely heavily on email infrastructure.

A properly configured mail environment requires:

* DNS engineering
* Linux administration
* Docker containerization
* SMTP routing knowledge
* Mail authentication
* TLS encryption
* Reputation management

This project demonstrates practical infrastructure engineering beyond basic cloud deployments.

---

# Why Vultr Instead of AWS EC2?

I intentionally used a Vultr VPS instead of AWS EC2 for this project.

## Reason

AWS commonly restricts outbound SMTP port 25 on EC2 instances to prevent abuse and spam.

While mailboxes and Mailcow can technically run on EC2, direct outbound SMTP delivery becomes difficult without additional approval processes.

Vultr VPS allows SMTP traffic more easily, making it better suited for realistic self-hosted mail server labs.

## Final Hybrid Design

* Inbound mail handled locally on Vultr VPS
* Outbound mail relayed through AWS SES

This creates a hybrid architecture with strong deliverability and enterprise-style separation of responsibilities.

---

# Technologies Used

| Technology     | Purpose                     |
| -------------- | --------------------------- |
| Vultr VPS      | Mail server hosting         |
| Ubuntu Linux   | Base operating system       |
| Docker         | Container runtime           |
| Mailcow        | Full mail platform          |
| Postfix        | SMTP mail transfer          |
| Dovecot        | Mailbox access              |
| SOGo           | Webmail interface           |
| AWS SES        | Outbound SMTP relay         |
| Cloudflare DNS | DNS management              |
| SPF            | Sender validation           |
| DKIM           | Cryptographic email signing |
| DMARC          | Anti-spoofing enforcement   |
| Reverse DNS    | Mail reputation validation  |

---

# Final Architecture

## Inbound Mail Flow

```text
Internet
    ↓
MX Record Lookup
    ↓
mail.basitcloudlab.com
    ↓
Vultr VPS Public IP
    ↓
Ubuntu Linux Server
    ↓
Docker Containers
    ↓
Mailcow Stack
    ↓
Postfix
    ↓
Mailbox Storage (Dovecot)
    ↓
SOGo Webmail
```

## Outbound Mail Flow

```text
SOGo Webmail
    ↓
Postfix (Mailcow)
    ↓
AWS SES SMTP Relay
    ↓
Gmail / Outlook / Yahoo
```

---

# DNS Records Configured

## A Record

```text
mail.basitcloudlab.com → VPS Public IP
```

Used to route mail hostname to the Vultr VPS.

---

## MX Record

```text
basitcloudlab.com → mail.basitcloudlab.com
```

Defines which server receives inbound mail for the domain.

---

## SPF Record

Final Production SPF:

```text
v=spf1 mx include:amazonses.com -all
```

Purpose:

* Authorizes Mailcow and AWS SES to send mail for the domain
* Rejects unauthorized senders

---

## DKIM

DKIM signing was configured using:

* Mailcow DKIM signing
* AWS SES DKIM verification

Purpose:

* Cryptographically validates outbound mail authenticity
* Prevents tampering/spoofing

---

## DMARC

Final Production DMARC:

```text
v=DMARC1; p=reject; rua=mailto:admin@basitcloudlab.com; adkim=s; aspf=s
```

Purpose:

* Reject spoofed emails
* Enforce strict SPF and DKIM alignment
* Generate DMARC reports

---

## Reverse DNS / PTR

Configured PTR:

```text
VPS IP → mail.basitcloudlab.com
```

Purpose:

* Improves sender reputation
* Helps Gmail and Outlook trust SMTP server
* Enables forward-confirmed reverse DNS

---

# Mailcow Services Running

| Service     | Purpose              |
| ----------- | -------------------- |
| Postfix     | SMTP mail handling   |
| Dovecot     | IMAP mailbox service |
| SOGo        | Webmail platform     |
| Rspamd      | Spam filtering       |
| Redis       | Caching              |
| MariaDB     | Database             |
| Nginx       | Reverse proxy        |
| DKIM signer | Email signing        |

---

# Step-by-Step Build Process

## 1. Deployed Vultr VPS

* Ubuntu 24.04 LTS
* 4 GB RAM
* Public IPv4

---

## 2. Updated Linux Server

```bash
apt update && apt upgrade -y
```

Installed required packages:

```bash
apt install -y curl git nano jq htop dnsutils ufw
```

---

## 3. Installed Docker

```bash
curl -fsSL https://get.docker.com | sh
```

Verified:

```bash
docker --version
docker compose version
```

---

## 4. Installed Mailcow

```bash
git clone https://github.com/mailcow/mailcow-dockerized
```

Generated configuration:

```bash
./generate_config.sh
```

Started containers:

```bash
docker compose pull
docker compose up -d
```

---

## 5. Configured DNS

Configured:

* A Record
* MX Record
* SPF
* DKIM
* DMARC
* Reverse DNS

---

## 6. Configured AWS SES Relay

Created:

* SES domain identity
* SES DKIM verification
* SES SMTP credentials

Configured Mailcow relay:

```text
[email-smtp.us-east-1.amazonaws.com]:587
```

Bound SES relay to:

```text
basitcloudlab.com
```

---

# Testing Performed

## Gmail Inbox Delivery

Verified successful Gmail inbox delivery.

Result:

```text
Delivered to Inbox Successfully
```

---

## SPF Validation

Result:

```text
SPF = PASS
```

---

## DKIM Validation

Result:

```text
DKIM = PASS
```

---

## DMARC Validation

Result:

```text
DMARC = PASS
```

---

## TLS Encryption

Verified encrypted SMTP delivery.

---

## SES Relay Validation

Verified outbound mail routed through AWS SES successfully.

---

# Key Lessons Learned

## Linux Administration

* Hostname configuration
* Package management
* Docker deployment
* Service validation
* SMTP troubleshooting

---

## DNS Engineering

* A records
* MX records
* TXT records
* CNAME records
* Reverse DNS
* DNS propagation

---

## Mail Authentication

* SPF authorization
* DKIM cryptographic signing
* DMARC enforcement
* Anti-spoofing protection

---

## Cloud & Infrastructure

* Hybrid mail architecture
* SES SMTP relay configuration
* Dockerized infrastructure
* Production-style validation

---

# Security Features Implemented

* SPF strict enforcement
* DKIM signing
* DMARC reject policy
* TLS encrypted transport
* Reverse DNS alignment
* Secure SMTP authentication
* AWS SES relay reputation

---

# Troubleshooting Commands Used

## Check Containers

```bash
docker ps
```

## Restart Postfix

```bash
docker compose restart postfix-mailcow
```

## DNS Validation

```bash
dig MX basitcloudlab.com
```

```bash
dig -x VPS_IP
```

```bash
dig CNAME DKIM_RECORD
```

---

# Resume Project Summary

## Hybrid Mail Infrastructure: Vultr VPS + Mailcow + AWS SES Relay

Built a production-style hybrid mail infrastructure using Mailcow on Docker with AWS SES SMTP relay integration.

Configured:

* SMTP mail routing
* SPF, DKIM, and DMARC authentication
* Reverse DNS
* TLS-secured delivery
* SES outbound relay
* Gmail inbox validation

Validated successful inbox delivery with SPF PASS, DKIM PASS, and DMARC PASS.

---



---

# Final Result

Successfully deployed and validated a secure hybrid mail infrastructure capable of:

* Receiving inbound mail locally
* Relaying outbound mail securely through AWS SES
* Passing enterprise-grade mail authentication checks
* Delivering email successfully to Gmail inboxes

---

# Project Status

```text
PROJECT COMPLETE
INFRASTRUCTURE CLEANED UP
```

