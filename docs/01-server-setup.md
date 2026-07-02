# 1. Server Setup

[← Back to overview](../README.md)

## Create a dedicated user

Log in as root (or via `sudo su`) and create a dedicated user for the SEB Server setup:

```bash
useradd sebserver -m
passwd sebserver
```

## Install APT packages

As root:

```bash
apt install git docker-compose default-mysql-client -y
```

## Adjust SSH config (optional)

To keep SSH sessions alive, add the following to `/etc/ssh/sshd_config` and restart the SSH service:

```
ClientAliveInterval 300
ClientAliveCountMax 3
```

---

Next: [2. Docker Setup →](02-docker-setup.md)
