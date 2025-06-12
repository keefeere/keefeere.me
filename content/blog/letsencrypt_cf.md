---
title: "Setting Up Let's Encrypt with DNS Challenge Using Cloudflare on Synology"
date: 2025-03-21
description: "A step-by-step guide to obtaining Let's Encrypt certificates using
the DNS-01 challenge with Cloudflare DNS on Synology NAS."
tags: ["Let's Encrypt", "Cloudflare", "DNS Challenge", "SSL", "acme.sh", "Synology"]
---

Securing your Synology NAS with SSL/TLS certificates is essential. Since
Synology's built-in Let's Encrypt support does not support wildcard
certificates, we can use the `acme.sh` script with Cloudflare's DNS
challenge to obtain and renew certificates automatically.

## Prerequisites

Before starting, ensure you have:

- A Synology NAS running DSM 7 or later.
- A domain managed by Cloudflare.
- Cloudflare API token with DNS edit permissions.
- SSH access enabled on your Synology NAS.

## Step 1: Enable SSH and Connect to Your Synology NAS

1) Log in to your Synology DSM.
2) Go to **Control Panel** → **Terminal & SNMP**.
3) Enable **SSH Service** and set the port (default is 22).
4) Connect to your NAS via SSH:

   ```bash
   ssh admin@your-nas-ip
   ```

   Replace `admin` with your username and `your-nas-ip` with your NAS's IP address.

## Step 2: Create a Dedicated User for Certificate Management

For security, it's best to create a separate user for managing
certificates instead of using `root`.

1) Log in to your Synology DSM.
2) Go to **Control Panel** → **User & Group**.
3) Click **Create** to add a new user:

- Username: `mycertadmin`
- Password: Set a strong password.
- Assign the user to the `administrators` group.

## Step 3: Install acme.sh

Download and install `acme.sh` for the `acme` user:

```bash
curl https://get.acme.sh | sh
```

## Step 4: Configure Cloudflare API Token and Synology login data

1) Log in to [Cloudflare Dashboard](https://dash.cloudflare.com/).
2) Navigate to **My Profile** → **API Tokens**.
3) Click **Create Token** and use the **Edit zone DNS** template.
4) Choose **Specific Zone** and select your domain.
5) Copy the generated token.

Set the API token and username created previous steps in your environment:

```bash
export CF_Token="your_cloudflare_api_token"
export SYNO_USERNAME="mycertadmin"
export SYNO_PASSWORD="YOUR CERTADMIN PASSWORD"
export SYNO_CERTIFICATE="Let's Encrypt"
export SYNO_CREATE=1
```

## Step 5: Issue a Let's Encrypt Certificate

Run the following command to obtain a wildcard certificate:

```bash
acme.sh --issue --dns dns_cf -d example.com -d "*.example.com"
```

Replace `example.com` with your actual domain.

## Step 6: Install the Certificate in Synology DSM

Once issued, deploy the certificate in Synology DSM:

```bash
acme.sh deploy -d 'example.com' --deploy-hook synology_dsm
```

This deploys your certificate in Synology NAS.

## Step 7: Automate Certificate Renewal Using Synology Task Scheduler

1) Open **Control Panel** → **Task Scheduler**.
2) Click **Create** → **Scheduled Task** → **User-defined script**.
3) Name the task (e.g., `Renew Let's Encrypt Certificate`).
4) Set the user to `acme`.
5) Go to the **Schedule** tab and set it to run daily (e.g., every day at 3 AM).
6) In the **Task Settings** tab, under **Run command**, enter:

   ```bash
   sh /volume1/homes/user/acme.sh renew -d 'example.com:' --deploy-hook synology_dsm
   ```

7) Click **OK** and enable the task.

This ensures the certificate is renewed automatically
and integrated into Synology's system.

## Conclusion

By using `acme.sh` with Cloudflare DNS challenge, you
can secure your Synology NAS with Let's Encrypt wildcard
certificates. This setup ensures seamless renewal and
integration with Synology's services.
