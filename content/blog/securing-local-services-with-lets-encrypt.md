+++
title = "Securing local services with Let's Encrypt"
date = 2025-07-28T14:20:54+01:00
draft = false
summary = "Securing local services within a home network using Let's Encrypt certificates. Normally a public Web server is required, but it is possible to obtain certificates when the domain is not publicly reachable using the ACME DNS challenge."
+++

## Introduction

I have an ever-increasing number of services that I host in my homelab, such as Immich and Home Assistant. These services can only be accessed on the local network, so while I can reasonaly safely access them with a self-signed certificate, using a certificate signed by a trusted Certificate Authority will remove browser warnings and resolve potential compatibility issues.

However, how do you get a certificate for a domain that isn't publicly accessible? When setting up a certificate using Let's Encrypt, you'll need to prove that you control the domain[^lets-encrypt-how-it-works]. Tools like Cerbot can use the ACME protocol to automatically do this, normally by provisioning an HTTP resource under your domain. However, this won't work if you don't have a Web server that is publicly reachable.

The solution is the ACME DNS challenge and a DNS provider that provides an API to provision DNS records.

## Prerequisites

- A domain name that you own.
- A DNS provider that is supported by one of the [Certbot DNS plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins) (I'm using Cloudflare).

If your DNS provider is not supported, it is possible to manually provision the DNS record, but this will not allow for automatic renewals. Alternatively, a different ACME client may support your DNS provider, such as [acme.sh](https://github.com/acmesh-official/acme.sh/wiki/dnsapi), however this guide will focus on Certbot.

## Installing Certbot

These steps are based on the official Certbot installation instructions using pip on a Linux machine[^certbot-installation]. I recommend that you follow the official documentation for your specific operating system.

### Install Certbot dependencies

On Debian-based systems:

```sh
sudo apt update
sudo apt install python3 python3-dev python3-venv libaugeas-dev gcc
```

On Red Hat-based systems:

```sh
sudo dnf install python3 python-devel augeas-devel gcc
```

### Remove Certbot OS packages

If you've previously installed Certbot packages from your OS package manager, you should remove them before installing Certbot using pip.

### Set up a Python virtual environment

```sh
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
```

### Install Certbot

```sh
sudo /opt/certbot/bin/pip install certbot

# Link certbot to /usr/bin/, enabling use of the certbot command.
sudo ln -s /opt/certbot/bin/certbot /usr/bin/certbot
```

### Install Certbot DNS plugin

```sh
# Replace <PLUGIN> with your DNS provider, for example, certbot-dns-cloudflare.
sudo /opt/certbot/bin/pip install certbot-dns-<PLUGIN>
```

## Configure Certbot DNS plugin

These steps are based on the documentation for the Cloudflare plugin. If not using Cloudflare, follow [the Certbot documentation for your particular DNS provider](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins).

1. Go to your [Cloudflare dashboard](https://dash.cloudflare.com/?to=/:account/profile/api-tokens) and create a new API token.
1. Ensure that the token has `Zone:DNS:Edit` permissions. This allows the Certbot DNS plugin to make the necessary changes to your DNS records required by the ACME DNS challenge.

Once you have created your token, you should save the token to `/etc/letsencrypt/cloudflare.ini` with the following content:

```ini
dns_cloudflare_api_token = <your_token>
```

## Get a certificate

You are now ready to get a certificate for your domain. You can create a certificate for the domain itself, e.g., `example.com`, or particular subdomains, e.g., `photos.example.com`. You can also create a wildcard certificate, e.g., `*.example.com`, which can be used for all subdomains under your domain.

Run the following to create a wildcard certificate:

```sh
# Replace <PROVIDER> and <DOMAIN> with your DNS provider and your domain.
# You can add addition -d '<DOMAIN>' arguments to add additional domains to the
# certificate.
sudo certbot certonly --dns-<PROVIDER> --dns-<PROVIDER>-credentials \
/etc/letsencrypt/<PROVIDER>.ini -d '*.<DOMAIN>'

# For example, for Cloudflare and example.com:
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials \
/etc/letsencrypt/cloudflare.ini -d '*.example.com'
```

**Note**: the [Certificate Transparency](https://en.wikipedia.org/wiki/Certificate_Transparency) Internet security standard requires that all issued certificates be logged in public logs. These logs can be searched freely on websites such as [crt.sh](https://crt.sh). If you are obtaining a certificate for a specific subdomain rather than a wildcard, it is important to consider the potential exposure of insights into your internal network from these logs.

To ensure your certificate automatically renews, run the following line to add a cron job:

```sh
# Twice daily, with a random delay to reduce load on Let's Encrypt's servers,
# check and renew your certificates.
echo "0 0,12 * * * root /opt/certbot/bin/python -c \
'import random; import time; time.sleep(random.random() * 3600)' && \
sudo certbot renew -q" | sudo tee -a /etc/crontab > /dev/null
```

Test the renewal process by running:

```sh
sudo certbot renew --dry-run
```

## Conclusion

You should have a certificate that will automatically renew which you can use to secure your local services. Just point your DNS to your local IP (e.g. `photos.example.com` to `192.168.0.2`).

## Further reading

- [Putting your local services behind a subdomain using NGINX and your new certificate](/blog/host-homelab-services-behind-subdomain/).
- [Using Pi-hole for locally resolvable DNS for your internal services](/blog/pihole-with-doh/).

[^lets-encrypt-how-it-works]: https://letsencrypt.org/how-it-works/
[^certbot-installation]: https://certbot.eff.org/instructions?ws=other&os=pip&tab=wildcard
