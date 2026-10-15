---
layout: post
title: "My VPS is My Homelab: Building a Self-Hosted Stack That Actually Works"
description: "How I consolidated a scattered homelab onto a cheap VPS with Docker, Tailscale and Caddy, plus the exact commands to build it yourself."
date: 2026-05-11 09:00:00 -0700
published: true
image: '/assets/blog/vps-homelab-setup-tutorial.webp'
tags: [homelab, tutorial, self-hosting, vps, tailscale, docker]
---
Over the past year I've been building the foundation for a personal AI automation setup: a place where agents, workflows and services can run reliably without me depending on somebody else's platform. The VPS and networking setup in this post is what everything else sits on. The job search agent, the knowledge base, the daily briefings, the client monitoring and my GitHub repos all run on top of it. Getting this layer right is what made everything above it possible.

I've been homelabbing for years. Raspberry Pis running Pi-hole, a Synology NAS humming in the corner running way too many services for its little processor. At some point I decided to consolidate, and the answer wasn't a bigger NAS or a faster Pi. It was a cheap VPS and a smarter way of handling the network.

## The problem with the classic homelab

Most homelabs fail the same way: they're tied to your home internet. Changing IP addresses, ISP outages, NAT headaches and port forwarding rules you forgot you ever set up. It's all fragile in ways that don't matter... until they suddenly do.

I wanted my services reachable all the time, from any device, from anywhere. And I didn't want to spend a bunch of money doing it.

For years my homelab security plan was "open some ports and hope for the best." I had services bound to `0.0.0.0`, a UFW config I sort of remembered setting up once, and that low hum of anxiety you get from reading one too many breach postmortems. It worked the way a screen door works. Technically a door.

## The stack

The core is simple:

- **[RackNerd](https://racknerd.com) VPS**: a cheap Ubuntu 24.04 box that's always on.
- **[Tailscale](https://tailscale.com)**: a private network that connects all my devices to each other. Every machine I own (the VPS, my MacBook, the NAS, my Arch Linux desktop) is on the same private network all the time, with zero port forwarding.
- **[Docker](https://www.docker.com)**: every service runs in a container with its own compose file.
- **[Caddy](https://caddyserver.com)**: the reverse proxy. It handles HTTPS for every service on my Tailscale network on its own.

The big decision: **nothing is exposed to the public internet.** Caddy only listens on the Tailscale IP. SSH only listens on Tailscale. Every single service is Tailscale-only.

## What's running

A handful of containers that actually get used:

- **[n8n](https://n8n.io)**: workflow automation. It ties together [Gmail](https://mail.google.com), [Discord](https://discord.com), [Todoist](https://todoist.com) and a pile of APIs.
- **[FreshRSS](https://freshrss.org)**: my RSS reader for job boards and tech news.
- **[MainWP](https://mainwp.com)**: manages about 60 client WordPress sites from one dashboard.
- **[Uptime Kuma](https://uptime.kuma.pet)**: keeps an eye on those same sites.
- **[Glance](https://github.com/glanceapp/glance)**: my homelab dashboard. One tab, everything I need to see.
- **[Watchtower](https://containrrr.dev/watchtower/)**: updates the containers overnight.

## The Synology does storage, not compute

My Synology NAS stays in its lane: storage and media. Plex runs there because it needs to sit close to the media files. Immich runs there for the same reason.

The VPS gets the lightweight services, the NAS gets the storage-heavy ones, and they talk to each other over Tailscale when they need to.

## Why Tailscale changed everything

Before Tailscale I was juggling SSH keys stored in [1Password](https://1password.com). It worked, but it wasn't pretty, and it got worse with every machine I added. Tailscale replaced all of it with a private network that pretty much sets itself up.

Every device gets a stable IP. DNS is handled. I can reach the VPS from my phone, my laptop, my NAS, anywhere Tailscale is running. The whole thing feels like one private home network, even though the machines are spread across three different physical locations.

And the best part: no port 22, no port 80, no port 443 open to the internet. The only way in is through Tailscale's login, which the Tailscale team looks after, not me. The outside gremlins can knock all they want.

---

## Build it yourself

Here's the whole setup, start to finish.

**What you'll end up with:** A VPS running [Docker](https://docs.docker.com/get-docker/) containers behind Caddy, fully locked to a Tailscale network, with automatic TLS and zero port-forwarding required.

**Prerequisites:** A VPS ([Ubuntu](https://ubuntu.com/) 22.04+), a Tailscale account, basic Linux comfort.

---

## Step 1: Provision the VPS

Any cheap VPS will do. I use [RackNerd](https://www.racknerd.com/). My box is under $100 a year for 6 vCPU and 8GB RAM, which is more than this stack needs because my agents live there too. A smaller plan will handle everything in this post.

Once it's up, harden it before you do anything else. Yes, this is the boring part. Do it anyway.

```bash
# Update packages
apt update && apt upgrade -y

# Create a non-root user
adduser youruser
usermod -aG sudo youruser

# Set up SSH key auth (from your local machine)
ssh-copy-id youruser@your-vps-ip

# Harden SSH: drop a file in /etc/ssh/sshd_config.d/
cat > /etc/ssh/sshd_config.d/60-hardening.conf << 'EOF'
PasswordAuthentication no
PermitRootLogin no
X11Forwarding no
AllowTcpForwarding no
EOF

systemctl restart sshd
```

Next, install [fail2ban](https://www.fail2ban.org/). Three bad login attempts gets a 24-hour ban. That's long enough to annoy the bots, and short enough that you won't lock yourself out for good if you fat-finger your password on a new device.

```bash
apt install fail2ban -y
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
maxretry = 3
bantime = 86400
ignoreip = 127.0.0.1/8 100.64.0.0/10

[sshd]
enabled = true
EOF
systemctl enable --now fail2ban
```

The `100.64.0.0/10` in `ignoreip` is Tailscale's address range. Once you're connecting over Tailscale, you can't accidentally ban yourself again.

---

## Step 2: Install Tailscale

This is the step that turns a plain old VPS into something actually nice to live with.

```bash
curl -fsSL https://tailscale.com/install.sh | sh
tailscale up
```

Follow the link it prints to connect the VPS to your Tailscale account. Once it's connected, write down the VPS's Tailscale IP (something like `100.x.y.z`). That's the only IP your services will ever listen on. Not the public IP, not localhost, just the Tailscale IP.

Then install Tailscale on every device you want to reach your homelab from (laptop, phone, whatever) and sign them in to the same account.

**Enable MagicDNS** in the [Tailscale admin console](https://login.tailscale.com/admin/dns). This gives your VPS a stable hostname like `your-machine.your-tailnet.ts.net` that works from all of your devices. You'll use that hostname everywhere from here on.

---

## Step 3: Install Docker

```bash
curl -fsSL https://get.docker.com | sh
usermod -aG docker youruser
```

Log out and back in so the group change takes effect. Skip this and you'll be typing `sudo` in front of every Docker command and quietly cursing the tutorial that didn't warn you. Consider yourself warned.

---

## Step 4: Set Up Caddy as a Reverse Proxy

Caddy handles HTTPS for your Tailscale hostname on its own. No cert management, no Certbot renewals, no `let's encrypt rate limit exceeded` emails at 2am.

```bash
mkdir -p ~/dev/docker/caddy
cd ~/dev/docker/caddy
```

`caddy.yml`:
```yaml
services:
  caddy:
    image: caddy:latest
    container_name: caddy
    ports:
      - "YOUR_TAILSCALE_IP:80:80"     # Replace with your Tailscale IP
      - "YOUR_TAILSCALE_IP:443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    networks:
      - proxy-net
    restart: unless-stopped

volumes:
  caddy_data:
  caddy_config:

networks:
  proxy-net:
    name: proxy-net
```

`Caddyfile`:
```
your-machine.your-tailnet.ts.net:8080 {
    reverse_proxy n8n:5678
}

your-machine.your-tailnet.ts.net:8083 {
    reverse_proxy uptime-kuma:3001
}
```

Caddy gets the TLS certificates for your Tailscale hostname by itself. Adding a new service means adding a new block here and reloading.

Start it:
```bash
docker compose -f caddy.yml up -d
```

**The one line that matters:** in the compose file, `ports` is bound to `YOUR_TAILSCALE_IP`, not `0.0.0.0`. That one difference is the whole security model. `0.0.0.0` means "anyone on the internet." `YOUR_TAILSCALE_IP` means "only devices on my Tailscale network." Bind to `0.0.0.0` and you've put your services on the public internet and undone all of step 1.

---

## Step 5: Add Services

Each service gets its own folder and compose file. They all join `proxy-net` so Caddy can find them by container name. Here's [n8n](https://n8n.io/) as an example:

```bash
mkdir -p ~/dev/docker/n8n
```

`n8n.yml`:
```yaml
services:
  n8n:
    image: n8nio/n8n
    container_name: n8n
    hostname: n8n
    environment:
      N8N_HOST: your-machine.your-tailnet.ts.net
      N8N_PORT: 5678
      N8N_PROTOCOL: https
      WEBHOOK_URL: https://your-machine.your-tailnet.ts.net:8084/
      GENERIC_TIMEZONE: America/Los_Angeles
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - proxy-net
    restart: unless-stopped

volumes:
  n8n_data:

networks:
  proxy-net:
    external: true
```

Start it:
```bash
docker compose -f n8n.yml up -d
```

Add the corresponding block to your Caddyfile, then reload Caddy without downtime:
```bash
docker exec caddy caddy reload --config /etc/caddy/Caddyfile
```

Rinse and repeat for each service. Here's my stack at the moment (there's more, but a guy's gotta have some secrets):

| Service                                                | Port | Purpose             |
| ------------------------------------------------------ | ---- | ------------------- |
| [n8n](https://n8n.io/)                                 | 8084 | Workflow automation |
| [FreshRSS](https://freshrss.org/)                      | 8080 | RSS aggregator      |
| [Uptime Kuma](https://github.com/louislam/uptime-kuma) | 8083 | Site monitoring     |
| [Glance](https://github.com/glanceapp/glance)          | 8085 | Homelab dashboard   |

---

## Step 6: Lock Down UFW

**"YOU SHALL NOT PASS!"**
Binding to the Tailscale IP already keeps the outside world out, but UFW adds an explicit "no" on top:

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow in on tailscale0  # Allow all Tailscale traffic
ufw deny 8080               # Block public access to service ports
ufw deny 8083
ufw deny 8084
ufw deny 8085
ufw enable
```

You can also use [Tailscale's access controls](https://tailscale.com/kb/1018/acls/) to decide which of your devices can reach which services. That's worth a look once the basics are solid.

---

## Step 7: Add Watchtower for Auto-Updates

Running containers means running software that gets security fixes and updates. [Watchtower](https://containrrr.dev/watchtower/) pulls new images every night and cleans up the old ones:

```yaml
services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --schedule "0 0 3 * * *" --cleanup
    restart: unless-stopped
```

Updates at 3am, old layers cleaned up automatically. Set it and forget it.

---

## Step 8: Keep the Box Itself Patched

Watchtower only updates your containers. It doesn't touch the operating system underneath them, and that's the part that gets the scary security notices. The `apt upgrade` from Step 1 was a one-time thing, not a subscription.

Log in every week or two and run:

```bash
sudo apt update && sudo apt upgrade -y
```

If a kernel update comes through, you'll need a reboot (`sudo reboot`) before it actually counts. Ubuntu will leave a note at `/var/run/reboot-required` when that's the case.

If you'd rather not rely on remembering, Ubuntu's `unattended-upgrades` package installs security patches on its own:

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

It handles security fixes automatically, but it's not a reason to never log in again. Check in now and then anyway.

### Do your homework on your provider

Everything in this post happens inside the server. Your VPS provider usually offers protection outside it too, and it's worth half an hour to see what yours has. Depending on the provider, look for:

- **Two-factor authentication on the control panel.** If someone gets into your provider account, they can reset your root password or rebuild the server from the outside, and none of your SSH hardening will matter.
- **A network or cloud firewall.** It blocks traffic before it ever reaches your server, which is a nice second layer on top of UFW.
- **Snapshots or backups.** Some plans include them and some charge extra. Either way, know whether you have them *before* you need one.
- **DDoS protection, rescue mode or console access, and security notices.** Find out what's included and turn on alerts so you hear about problems from them first.

Every provider offers something different, so read through their docs and your control panel instead of assuming. This setup gets you a locked-down server. Keeping it locked down is an ongoing job, and it's yours.

---

## The Secret Sauce: tmux

Docker keeps your services up. But the stuff you run by hand, like a Claude Code session or a script you're babysitting, only lives as long as your SSH connection. Close the laptop and it's gone.

[tmux](https://github.com/tmux/tmux) is what turns this box from "a server I SSH into" into "a computer that's always on and always where I left it." It keeps terminal sessions running on the server whether you're connected or not.

```bash
sudo apt install tmux -y
tmux new-session -s main
```

Start your work inside it, press `Ctrl+B` then `D` to detach, and close the laptop. Later, from any device on your tailnet, SSH back in and pick up exactly where you were:

```bash
tmux attach -t main
```

That's the same trick that makes always-on agent setups like [OpenClaw](https://openclaw.ai) and [Hermes](https://hermes-agent.nousresearch.com) appealing: an agent that lives on a machine that never sleeps, so you can check in on it from anywhere. A cheap VPS, Tailscale and tmux get you most of that feeling with tools you already understand. I go deeper on running Claude Code this way in the [Claude + Obsidian post](/blog/claude-obsidian-workflow).

---

## Accessing Everything

Once it's all running, every service lives at `https://your-machine.your-tailnet.ts.net:PORT` from any device on your Tailscale network. There's no VPN to connect by hand, Tailscale just runs in the background. Pull out your phone on coffee shop WiFi and your homelab dashboard loads like it's sitting on your desk.

You now have your own little corner of the internet that nobody else can get into. Not bad for about $100 a year.

SSH access: `ssh youruser@your-machine.your-tailnet.ts.net` or `ssh youruser@YOUR_TAILSCALE_IP`

---

## Troubleshooting

**Service not reachable:** Check that the container is on `proxy-net` (`docker network inspect proxy-net`) and that your Caddyfile block has the right upstream hostname and port.

**TLS not working:** Make sure Tailscale MagicDNS is enabled in the admin console and that the hostname in your Caddyfile matches exactly. Caddy can't provision a cert for a hostname Tailscale doesn't recognize.

**Port exposed publicly:** Check your `ports` binding in the compose file. It must be `tailscale-ip:host-port:container-port`. If you see `host-port:container-port` without the IP prefix, it's bound to `0.0.0.0` and the whole internet can knock.

---

## What I'd do differently

The biggest one: I'd start with Tailscale on day one. The SSH key approach worked, but it didn't grow well, and adding each new device was a manual chore I kept putting off.

I'd also resist the urge to run everything. The most useful services aren't the flashiest ones. Uptime Kuma and n8n get used every day. The fun experiment containers mostly sit there collecting dust.

## What's next

An OptiPlex 7070 Micro running CachyOS is in the plan for a garage workstation. Whether that eventually turns into a little Kubernetes cluster across the VPS and the garage box is still an open question. The idea is tempting. The complexity, less so.

For now, though, the stack is boring in the best possible way. Everything just runs.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
