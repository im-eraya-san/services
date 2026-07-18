# Samba

A Rocky Linux–based Samba file-sharing service, containerized with Docker and deployed via `docker-compose`. Part of the [services](../) collection of self-hosted Docker services.

## Project Structure

```
samba/
├── build/
│   ├── Dockerfile
│   └── init              # Entrypoint script, launches smbd
├── config/
│   └── smb.conf           # Samba configuration (mounted into container)
├── docker-compose.yaml
└── README.md
```

## What's Inside

- **Base image:** `rockylinux:8`
- **Packages:** `samba`, `samba-common`, `samba-client`
- **Data directory:** `/srv/` created in the image, owned by `nobody:nobody`, mode `755`
- **Default config:** the image's stock `/etc/samba/smb.conf` is backed up to `smb.conf.bak` so the mounted config takes over cleanly
- **Entrypoint:** runs `smbd` in the foreground via the `init` script

## Samba Share Configuration

Defined in `config/smb.conf`:

```ini
[global]
workgroup = WORKGROUP
server string = Samba Server %v
netbios name = rocky-8
security = user
map to guest = bad user
dns proxy = no
ntlm auth = true

[Public]
path = /data
browsable = yes
writable = yes
guest ok = yes
read only = no
```

This exposes `/data` as a share named `Public`, with **guest access enabled** — no username/password required to connect, read, or write.

## Build

```bash
docker build -t smb:0.1 -f build/Dockerfile build/
```

## Run

Networking is set to `host` mode in `docker-compose.yaml`, so Samba's ports are exposed directly on the host.

```bash
docker compose up -d
```

This mounts `./config` → `/etc/samba/` (Samba configuration) into the container.

## Accessing the Share

From another machine on the same network, no credentials needed:

```bash
smbclient //<host-ip>/Public -N
```

Or mount it:

```bash
sudo mount -t cifs //<host-ip>/Public /mnt/point -o guest
```

On Windows/macOS/Linux file managers, browse to `\\<host-ip>\Public` (Windows) or `smb://<host-ip>/Public` (macOS/Linux).

## Notes

- The share path is `/data`, but only `/etc/samba/` is currently mounted as a volume in `docker-compose.yaml` — `/data` is not mounted to the host, so anything written to the share won't persist across container recreation. Add a volume mount (e.g. `$PWD/data/:/data/`) if persistence is needed.
- Guest access is enabled (`guest ok = yes`, `map to guest = bad user`), meaning **anyone on the network can read/write this share without authentication**. Fine for trusted local networks; add `valid users` and a Samba password (`smbpasswd -a <user>`) if that's not acceptable for your environment.
