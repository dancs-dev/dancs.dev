+++
title = 'Local DNS using Pihole with upstream DNS over HTTPS'
date = 2025-07-28T16:15:09+01:00
draft = false
summary = "Set up Pi-hole using Docker, configure it to use upstream DNS over HTTPS, and use local DNS to easily access local homelab services."
+++


## Introduction

When setting up my homelab, and convincing my family to use services I was hosting, one of the main points of friction was requiring them to enter an IP address and port. While the adblocking functionality of Pi-hole is useful for some devices and applications such as mobile phones, tools like [uBlock Origin](https://ublockorigin.com/) on [Firefox](https://www.firefox.com/en-US/) can be more powerful. The primary reason I use Pi-hole is for its ability to set local DNS records. This means, for example, I can set `home.example.com` to point to a local IP address running Home Assistant without having to make that DNS record public.

However, the main drawback to Pi-hole is its reliance on plain unencrypted upstream DNS and the lack of privacy and data integrity that provides[^dns-privacy]. There have been several new protocols developed to resolve these issues, including the DNS over HTTPS (DoH) protocol in [RFC 8484](https://www.rfc-editor.org/rfc/rfc8484). The Pi-hole documentation provides a few workarounds for using DoH, [including using the `cloudflared` tool](https://docs.pi-hole.net/guides/dns/cloudflared/). Despite this tool being developed by Cloudflare, you can choose any DoH provider you like.

## Prerequisites

- A device such as a Raspberry Pi that can run 24/7
- [Docker](https://docs.docker.com/engine/install/)

## Installing and configuring Pi-hole

Create a `docker-compose.yaml` in a suitable directory. Make sure you adjust any settings as required, in particular, the `WEBPASSWORD` setting.

```yaml
services:
  # Credit to https://mroach.com/2020/08/pi-hole-and-cloudflared-with-docker/#option-1-hidden-cloudflared
  # for the idea of using a hidden internal Docker network.
  cloudflared:
    container_name: cloudflared
    image: cloudflare/cloudflared
    restart: unless-stopped
    command: proxy-dns
    environment:
      # The upstream DoH provider to use.
      - "TUNNEL_DNS_UPSTREAM=https://9.9.9.9/dns-query"
      # Listen on an unprivileged port.
      - "TUNNEL_DNS_PORT=5053"
      # Bind on all interfaces.
      - "TUNNEL_DNS_ADDRESS=0.0.0.0"
    networks:
      internal:
        ipv4_address: 172.30.9.2
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      # Expose the Pi-Hole interface on the local machine on port 8080.
      # Remove '127.0.0.1:' if not accessing the admin interface on the local
      # machine (and you have not yet set up a reverse proxy).
      - "127.0.0.1:8080:80/tcp"
    networks:
      internal:
        ipv4_address: 172.30.9.3
    environment:
      TZ: 'Europe/London'
      WEBPASSWORD: "<CHANGE_ME>"
      CORS_HOSTS: "pihole.example.com"
    # Ensure configuration and data is persistent.
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    depends_on:
      - cloudflared

networks:
  internal:
    ipam:
      config:
        - subnet: 172.30.9.0/29
```

Now start the containers:

```sh
# Pull the latest images.
docker compose pull

# Stop/remove any old containers.
docker compose down

# Start the containers in the background.
docker compose up -d
```

### Setting Pi-hole to use the cloudflared container for upstream DNS

Once the containers have started, log in to Pi-hole on `http://127.0.0.1:8080` (or the address and port if changed from the configuration) and use the password you set in the `WEBPASSWORD` configuration.

Navigate to `System > Settings > DNS` and uncheck all the default upstream DNS servers. 

Then, expand the custom DNS servers section and add `172.30.9.2#5053` - this is the IP address of the cloudflared container and the port it is listening on. Pi-hole will send DNS requests to cloudflared, which in turn will make a DoH request to the upstream DoH provider.

## Local DNS

Now onto the really neat part about Pi-hole - the ability to set local DNS records.

Navigate to `System > Settings > Local DNS Records`.

Under local DNS records, I would recommend creating one record per machine/IP address. Then, under local CNAME records, you can create as many domains per target as needed. This is probably best illustrated with an example:

### Local DNS records

| Domain    | IP |
| -------- | ------- |
| rpi-1.example.com | 192.168.0.2 |
| rpi-2.example.com | 192.168.0.3 |

### Local CNAME records

| Domain    | Target |
| -------- | ------- |
| pihole.example.com | rpi-1.example.com |
| home.example.com | rpi-1.example.com |
| photos.example.com | rpi-2.example.com |

One of the benefits of this approach is that you can easily change the IP address of a machine without having to update many records. It also makes it easier to see what machine a domain is pointing to.

Even if you don't currently have any other homelab services, you can use this feature for easier access to the Pi-hole admin interface! Once you've set up a local DNS record pointing to the machine your Pi-hole is running on, and a CNAME record for your Pi-hole, see [Further reading](#further-reading) for how to reverse proxy to Pi-hole using NGINX.

## Setting your devices to use Pi-Hole DNS

You now need to set up your devices to use Pi-hole as their DNS server. The most straightforward way to ensure all of your devices use Pi-hole is to set it as your router's DNS. Unfortunately, not all routers support this. If your router does not support this, you will need to set each device to use Pi-hole. As there are so many different ways this could be done, I will not go into detail here. Instead, use your favourite search engine to find instructions for your specific operating system or device.


## Further reading

- [Putting your local services behind a subdomain using NGINX](/blog/host-homelab-services-behind-subdomain/).
- [Securing local services with Let's Encrypt](/blog/securing-local-services-with-lets-encrypt/).

[^dns-privacy]: Huston, Geoff (July 2019). "[DNS Privacy and the IETF](https://ipj.dreamhosters.com/wp-content/uploads/2019/07/ipj222.pdf
)". The Internet Protocol Journal.
