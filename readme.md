# VPN node services

> This config is for Linux server. If you need to run this on Windows machine, replace `/var/run/docker.sock` and `/etc/localtime` in `compose.yaml` file with analogs (not sure about localtime tho).

A complete configuration for VPN child nodes. Just clone, config, run, config some more and add to master node. Everything is as automated as possible.

Minimal server specs based on `trust me bruv`:
* CPU: 1-2 cores will be fine.
* RAM: 2Gb, ideally 3-4.
* Storage: 40-60Gb, or else remove log collection.

## Logging

Logs are collected in `logs` folder and rotated automatically. Traefik collects up to 5 rotated log files 50Mb each. 3x-UI logs used in extreme cases, and their rotation controlled by service itself.

## Backups

Mostly managed by hosting service, but in case of panel updates, we can backup `data` folder ourselves.

## SSL/TLS

Controlled by Traefik. Each certificate issued and renewed automatically, using Let\`s Encrypt\`s DNS-01 strategy and stored in `certs.json` file.

## Configuration

There\`s a bit, mostly specifying sensitive information.

### Certificates

Create file `certs.json` and apply `chmod 600`

### Environment

`.env` file contains database credentials and `CF_TOKEN` generated in cloudflare dashboard. Token settings:
* `Zone:Read`, `DNS:Edit` over domain.
* is in - list of node IPs + master node IP.

### Traefik

Fill `api.dashboardName` and `certificatesResolvers.lets.acme.email` fields in `traefik.yaml` file.

### Dynamic services

* Replace every instance of `vpn.docker.localhost` with actual domain name.
* Ideally, domain for subscription router should differ from vps itself.
* Generate BA credentials and fill `http.middlewares.dashboard_auth.basicAuth.users[0]` field.
* Come up with HostSNI that will be used for 3x-UI inbound and fill `tcp.routers.proxy_vless.rule` field.

### First login

After everything up and running, you can go to `example.com/router/dashboard/` and verify than traefik resolved every router.

Then you need to proceed to `example.com/proxy/dashboard`, enter default credentials (admin:admin) and configure 3x-UI:

* Change default login and password.
* Generate panel\`s API token.
* Change subscription\`s URI path, port and Reverse Proxy URI.
* Change subscription appearance, copy/paste it from another node or from master.
* (Optional) Add domain to Hosts tab. Use 443 as port value.

### In/Out-bounds

Create VLESS inbound binded to a port 1201, configure it to sniff. Create outbound, named `proxy`. Then route russian traffic through `direct` and `IPv4` and everything else through `proxy`.

Now we can connect node to master.

### Connection to master

Go to master server\`s proxy dashboard and open Nodes tab. Here we can add new node using domain, port and API token from previous steps. Base path must be set to `proxy/dashboard`.
If you feel fancy, add a new outbound just in case of future updates.

> Critical. Verify that a new host appeared in Hosts tab. If not, add it manually.

Everything else will be sended over to new node. At this point we all set.

## Running

Just `docker compose up` this bitch.
