+++
title = 'Hosting homelab services behind subdomains using NGINX'
date = 2025-08-03T11:30:45+01:00
draft = false
summary = 'Host local services behind subdomains using the NGINX reverse proxy.'
+++

## Introduction

If you've started homelabbing, you've probably got several services hosted on a machine with several varying ports, and access them using, e.g., `192.168.0.2:8001` for service A, `192.168.0.2:9001` for service B, and so on. While this may be OK if you only have a few service and only you use them, this can get fairly messy quite quickly.

Instead, why not host them behind a subdomain? For example, you can have your PhotoPrism instance hosted at `photos.example.com` and your Home Assistant instance hosted at `home.example.com`. This can be achieved using a reverse proxy such as NGINX.

Let's illustrate this with an example of how this works: User -> photos.example.com -> NGINX server -> 192.168.0.2:8001 -> PhotoPrism instance.

I'll discuss how to do this on a local network. NGINX is also very useful for hosting services on the public Internet, however I recommend not exposing any homelab services outside of the LAN.

## Prerequisites

- Basic knowlegde of Linux command line and networking.
- A homelab service or two already running. (I'll cover the Pi-hole Web interface).
- A domain and/or a local DNS server such as Pi-hole.
- Optional: An SSL certificate - see how to generate one without a public Web server using Let's Encrypt.

## Install NGINX

Even though I normally prefer using Docker to run homelab services, I find it simpler to use NGINX natively on my system. This avoids the hassle of having to configure each service to network with an NGINX container. 

Install NGINX using the installation instructions for your particular system. If you happen to have homelab services run across multiple machines, you likely want to install NGINX on each machine, and set up the DNS record and site configuration onto the NGINX server running on that machine. This avoids unnecessary network traffic, as well as prevents unencrypted network traffic (if using SSL termination on NGINX).

On Debian-based systems:

```bash
sudo apt update
sudo apt install nginx
```

On RedHat-based systems:

```bash
sudo dnf install nginx
```

## Configure NGINX

### Creating site configuration

In NGINX, there exists a site configuration 'server name' (e.g., `service-1.example.com`). You'll need to create one for each service you host. As these configurations are in a privileged directory, I recommend editing these configurations using a command-line text editor with sudo, e.g., `sudo vim <path>` or `sudo nano <path>`. This can be installed easily on most Linux systems. 

On Debian-based systems, these exist in `/etc/nginx/sites-available/`. Sites here are not actually live until you link them to the `sites-enabled` directory. Start by creating a site configuration in `sites-available` using, e.g., `sudo touch /etc/nginx/sites-available/pihole.example.com`. You can then link it to sites-enabled using `sudo ln -s /etc/nginx/sites-available/pihole.example.com /etc/nginx/sites-enabled/`. If you decide to remove the service but want to keep the site configuration, you can then run `sudo unlink /etc/nginx/sites-enabled/pihole.example.com`.

On RedHat-based systems, things are a bit simpler. Site configurations exist and are live in `/etc/nginx/conf.d/`. Create a site configuration here using, e.g., `sudo touch /etc/nginx/conf.d/pihole.example.com`.

### Editing site configuration

If documentation exists for how to configure NGINX for the particular service, I would recommend following that documentation. Occasionally some services need additional configuration, such as enabling Web sockets.

Below is an example configuration for Pi-hole. You can use the below as a base for any service - just make sure to adjust the `server_name` and the `proxy_pass` port.

If using HTTPS:

```nginx
server {
    server_name pihole.example.com;

    listen [::]:443 ssl;
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    client_max_body_size 500M;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8080/;
    }
}

server {
    listen 80;
    listen [::]:80;

    server_name pihole.example.com;

    if ($host = pihole.example.com) {
        return 301 https://$host$request_uri;
    }

    return 404;
}
```

If not using HTTPS:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name pihole.example.com;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8080/;
    }
}
```

### Enabling site configuration

With that all in place, you are ready to test and enable the site configuration.

```bash
sudo nginx -t
```

If you see any errors, you will need to fix them before proceeding. (If you proceed without fixing, NGINX will not start which will cause downtime for any other services running on the server.)) If everything looks good:

```bash
sudo nginx -s reload
```

NGINX is now ready to proxy requests to `pihole.example.com` to the Pi-hole service running on `http://127.0.0.1:8080/`.

## Configure DNS

You now need to configure the DNS settings for your domain so that `pihole.example.com` points to the IP address of the machine running NGINX.

### You will need to set the following records:

#### DNS records

| Domain    | IP |
| -------- | ------- |
| <machine_name>.example.com | <machine_ip> |

#### CNAME records

| Domain    | Target |
| -------- | ------- |
| pihole.example.com | <machine_name>.example.com |

### Public DNS

You can do this on your domain's public DNS, however note that public DNS is not private - someone could look up the IP address of `pihole.example.com` and get information about your local infrastructure which may or may not be a concern depending on your use case. It is also not best practice to have DNS records pointing directly to internal IP addresses.

If using this method, follow the documentation according to your domain's DNS provider.

### Local DNS

I would recommend using a local DNS server where you can make local DNS configuration that applies only to your local network - see my guide [Local DNS using Pihole with upstream DNS over HTTPS](/blog/pihole-with-doh/) for how to do this.

## Tips and caveats

1. Once your homelab service is proxied using a reverse proxy running on the same machine, you can bind the Docker host port to only the local interface. This way, the service will only be reachable from your subdomain/NGINX over the local network. You can do this by specifying the local IP address as part of the host port. For example, for Pi-hole, you could do `"127.0.0.1:8080:80/tcp"`
1. Many homelab services have require Web sockets. You can enable them by adding the following directives to your NGINX location block:
    ```nginx
    location / {
        ...
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        ...
    }
    ```
1. You can create an NGINX site configuration for each service/subdomain you wish to proxy.

## Further reading

- [Securing local services with Let's Encrypt](/blog/securing-local-services-with-lets-encrypt/).
- [Using Pi-hole for locally resolvable DNS for your internal services](/blog/pihole-with-doh/).
