# Local Dev Host Certificates

Simple, ready-to-use TLS certificates for local development with the `local.devhost.name` domain. Skip the hassle of generating local SSL certs and dealing with CORS headaches—run everything behind a local HTTPS reverse proxy.

This is a tool provided for free by the [Simply Hexagonal](https://simplyhexagonal.org) open source collective.

## Like this tool? ❤

Please consider:

- Supporting Simply Hexagonal through [Open Collective](https://opencollective.com/simplyhexagonal) 💖
- [Buying the maintainer a coffee](https://www.buymeacoffee.com/jeanlescure) ☕
- Supporting the maintainer on [Patreon](https://www.patreon.com/jeanlescure) 🏆
- Starring this repo on [Github](https://github.com/jeanlescure/short-unique-id) 🌟

## Quick start

1. Add the certificates to your project (creates a `certs` folder):

```
git clone https://github.com/simplyhexagonal/local-dev-host-certs.git certs
```

2. Start a proxy and sample services with Docker Compose (place this next to the `certs` folder and run `docker compose up`):

```yml
services:
  proxy:
    image: nginxproxy/nginx-proxy
    # If port error try: sudo sysctl net.ipv4.ip_unprivileged_port_start=80
    privileged: true
    ports:
      - 80:80
      - 443:443
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - ./certs:/etc/nginx/certs
    environment:
      - DEFAULT_HOST=local.devhost.name
  web:
    image: strm/helloworld-http
    ports:
      - 8080:80
    environment:
      - VIRTUAL_HOST=local.devhost.name
      - VIRTUAL_PATH=/
  api:
    image: thomaspoignant/hello-world-rest-json
    ports:
      - 9090:8080
    environment:
      - VIRTUAL_HOST=local.devhost.name
      - VIRTUAL_PATH=/api/
      - VIRTUAL_DEST=/
```

3. Open your app at [https://local.devhost.name](https://local.devhost.name) (HTTP works too).

## How it works

- The DNS for `local.devhost.name` resolves to `127.0.0.1`.
- In the example above `nginx-proxy` container reads certs from `/etc/nginx/certs` (mounted from your `./certs` folder) and routes requests using `VIRTUAL_HOST`/`VIRTUAL_PATH`.
- Prefer sub-path routing when you can—it often simplifies CORS, but for cases where subdomains are needed, we also provide a wildcard `local-*.devhost.name` to `127.0.0.1`. Examples:
  - `local-api.devhost.name`
  - `local-db.devhost.name`
  - `local-admin.devhost.name`

## Renewal dates

The certificates are generated using [letsencrypt](https://letsencrypt.org/) and expire every 90 days.

We provide a file named `.scheduled-renewal` that contains the date of next renewal with the format `yyyy-mm-dd`.

We renew the certificates a few days before expiry to avoid disruptions.

The easiest way to automatically check if you need to renew is by comparing the current `.scheduled-renewal` file in your certs directory against the latest one in the `local-dev-host-certs` repo, for example:

```bash
# Inside certs directory

# Download to latest certs to renewal-certs directory
git clone https://github.com/simplyhexagonal/local-dev-host-certs.git renewal-certs

# Exit on error
# TODO

# Compare `.scheduled-renewal` against `renewal-certs/.scheduled-renewal`
# If different, move files up (overwrite existing) and delete empty `renewal-certs` directory
# TODO
```

You can schedule this automatic process to happen 24 hours after the scheduled renewal date.
