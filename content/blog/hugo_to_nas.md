---
title: >
  How to Make CI/CD for your static Hugo
  site and deploy it to your Synology NAS
date: "2025-02-03"
description: "Guide to setup CI/CD in GitHub for Synology NAS"
tags: ["Hugo", "Blog", "Static Site", "GitHub", "DevOps", "Synology"]
---

Setting up a CI/CD pipeline for your static Hugo site and deploying it to a Synology
NAS requires several steps, from creating an SSH key to configuring GitHub Actions
for automated deployment. This guide covers all the necessary details, including
handling SSH permissions, Web Station setup on Synology NAS, and setting up Let's
Encrypt SSL with DNS challenge on Cloudflare.

## Prerequisites

- A Synology NAS with Web Station installed.
- A GitHub repository containing your Hugo site.
- A domain name and an SSL certificate (Let's Encrypt recommended).
- A Cloudflare account for managing DNS records.

## Step 1: Set Up Your Hugo Site

If you haven't already, create a Hugo site:

```sh
hugo new site mysite
cd mysite
git init
```

Push it to GitHub:

```sh
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

### Handling Submodules

If you use a Hugo theme as a submodule, initialize and update it before pushing:

```sh
git submodule add <theme-repo-url> themes/<theme-name>
git submodule update --init --recursive
git push --recurse-submodules=on-demand
```

## Step 2: Generate an SSH Key and Configure Synology NAS

On your local machine, generate an SSH key:

```sh
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

Copy the contents of `~/.ssh/id_rsa.pub` and add it to your Synology NAS:

1. Log in to your NAS and go to **Control Panel > Terminal & SNMP**.
2. Enable SSH service.
3. Place the public key inside `/volume1/homes/<user>/.ssh/authorized_keys`
  and set the correct permissions:

   ```sh
   chmod 700 /volume1/homes/<user>/.ssh
   chmod 600 /volume1/homes/<user>/.ssh/authorized_keys
   ```

Test the connection:

```sh
ssh -p <your-port> <user>@<nas-ip>
```

## Step 3: Configure Web Station on Synology NAS

1. Install **Web Station** from Synology Package Center.
2. Set up a virtual host pointing to `/volume1/web/mysite`.
3. Ensure permissions allow web access (use `sudo` if necessary to modify files).

## Step 4: Set Up GitHub Actions for CI/CD

Create `.github/workflows/deploy.yml` in your repo:

```yaml
name: Deploy Hugo Site to Synology NAS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
      with:
        submodules: 'true'  # Ensures theme submodules are pulled

    - name: Install Hugo
      run: |
        wget https://github.com/gohugoio/hugo/releases/download/v0.142.0/hugo_extended_0.142.0_Linux-64bit.tar.gz
        tar -xzf hugo_extended_0.142.0_Linux-64bit.tar.gz
        sudo mv hugo /usr/local/bin/

    - name: Build Hugo Site
      run: hugo --minify

    - name: Deploy to Synology NAS
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        SSH_USER: ${{ secrets.SSH_USER }}
        SSH_HOST: ${{ secrets.SSH_HOST }}
        SSH_PORT: ${{ secrets.SSH_PORT }}
        SSH_TARGET_DIR: "/volume1/web/mysite"
      run: |
        echo "$SSH_PRIVATE_KEY" > id_rsa && chmod 600 id_rsa
        export RSYNC_RSH="ssh -p $SSH_PORT -i id_rsa"
        tar -czf site.tar.gz -C public .
        scp -P $SSH_PORT -O site.tar.gz $SSH_USER@$SSH_HOST:/volume1/homes/$SSH_USER/site.tar.gz
        ssh -p $SSH_PORT -i id_rsa $SSH_USER@$SSH_HOST "sudo tar -xzf \
            /volume1/homes/$SSH_USER/site.tar.gz -C $SSH_TARGET_DIR"
```

### Important Notes

- **Use `-O` in `scp`** to ensure proper permissions.
- **SSH paths are relative** to `/volume1/homes/<user>`
    by default.
- **Target directory `/volume1/web/mysite` requires sudo permissions**,
    so we extract the files using `sudo tar`.
- **Ensure GitHub Secrets are configured**:
   - `SSH_PRIVATE_KEY`: Private key for SSH authentication.
   - `SSH_USER`: Username on the NAS.
   - `SSH_HOST`: NAS IP or domain.
   - `SSH_PORT`: Custom SSH port.

To make your site accessible via a domain:

1. Purchase a domain and point it to your NAS IP.
2. Use Let's Encrypt with **DNS challenge** via Cloudflare API.

## Step 5: Set Up a Domain and SSL Certificate

To make your site accessible via a domain:

1. Purchase a domain and point it to your NAS IP.
2. Use **Let's Encrypt** for an SSL certificate in **Control Panel > Security > Certificate**.
3. Apply the certificate to Web Station.

### Automating SSL Renewal with `acme.sh`

If you use Let's Encrypt with a DNS challenge on Cloudflare, set up automatic renewal:

1. Install `acme.sh`:

   ```sh
   curl https://get.acme.sh | sh
   ```

2. Configure the Cloudflare API key for DNS challenge:

   ```sh
   export CF_Key="your-cloudflare-api-key"
   export CF_Email="your-cloudflare-email"
   ```

3. Issue the certificate:

   ```sh
   acme.sh --issue --dns dns_cf -d yourdomain.com --keylength ec-256
   ```

4. Deploy the certificate to Synology DSM:

   ```sh
   acme.sh --deploy -d yourdomain.com --deploy-hook synology_dsm
   ```

### Scheduling Automatic Renewal on Synology

To ensure your SSL certificate is renewed automatically,
create a scheduled task in Synology Task Scheduler:

1. Open **Control Panel > Task Scheduler**.
2. Create a new **User-defined script** task.
3. Set the schedule (e.g., daily).
4. In the task script, enter:

   ```sh
   sh /volume1/homes/<user>/acme.sh renew -d 'yourdomain.com' --deploy-hook synology_dsm
   ```

For detailed domain setup, refer to:

- [Synology Web Station Documentation](https://kb.synology.com/en-global/WebStation)
- [Let's Encrypt Guide](https://letsencrypt.org/getting-started/)

## Conclusion

With this setup, your Hugo site is automatically built and deployed to your
Synology NAS upon each push to the `main` branch. You now have a fully
automated CI/CD workflow with HTTPS security via Let's Encrypt and Cloudflare
DNS challenge.
