# Samba Docker Service

A minimal Samba file-sharing container built on `ubuntu:devel`, configured via `docker-compose`.

## Project Structure

```
samba/
├── Dockerfile
├── init                # Entrypoint script, launches smbd
├── docker-compose.yaml
├── config/
│   └── smb.conf         # Samba configuration (mounted into container)
└── data/                 # Shared files live here (mounted into container)
```

## What's Inside

- **Base image:** `ubuntu:devel`
- **Packages:** `samba`
- **User:** a Linux user `smb` is created with password `smb` (also used as the initial Samba password if configured — see [Security Notes](#security-notes))
- **Data directory:** `/data` (mode `777`) inside the container, mapped to `./data` on the host
- **Config:** `/etc/samba/` inside the container, mapped to `./config` on the host
- **Entrypoint:** runs `smbd` in the foreground via the `init` script

## Samba Share Configuration

Defined in `config/smb.conf`:

```ini
[sambashare]
    comment = Samba on Ubuntu
    path = /data/
    read only = no
    browsable = yes
```

This exposes `/data` as a share named `sambashare`, writable and browsable.

## Build

```bash
docker build -t samba:0.1 .
```

## Run

Networking is set to `host` mode in `docker-compose.yaml`, so Samba's ports are exposed directly on the host (no port mapping needed/possible to remap).

```bash
docker compose up -d
```

This will:
- Mount `./config` → `/etc/samba/` (Samba configuration)
- Mount `./data` → `/data/` (shared files)
- Restart automatically (`restart: always`)

## Accessing the Share

From another machine on the same network:

```bash
smbclient //<host-ip>/sambashare -U smb
```

Or mount it:

```bash
sudo mount -t cifs //<host-ip>/sambashare /mnt/point -o username=smb,password=smb
```

On Windows/macOS/Linux file managers, browse to `\\<host-ip>\sambashare` (Windows) or `smb://<host-ip>/sambashare` (macOS/Linux).
