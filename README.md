# SEB Server – Installation Guide

A simple guide for installing [SEB Server](https://github.com/SafeExamBrowser/seb-server) (bundled setup with Screen Proctoring) with Moodle integration, based on the official [seb-server-setup](https://github.com/SafeExamBrowser/seb-server-setup) Docker setup.

## Guide

1. [Server Setup](docs/01-server-setup.md) — create a user, install packages, SSH config
2. [Docker Setup](docs/02-docker-setup.md) — docker-compose, nginx, SSL certificates, first start
3. [Moodle Setup](docs/03-moodle-setup.md) — plugin, web service user, permissions, access token
4. [SEB Server Setup (Web UI)](docs/04-seb-server-webui.md) — institution, users, LMS setup, templates
5. [Creating an Exam](docs/05-creating-an-exam.md) — import, prepare and export an exam
6. [Useful Notes](docs/06-useful-notes.md) — tips and troubleshooting

## Prerequisites

- A Linux server (Debian/Ubuntu) with root access
  - Recommended for up to ~150 concurrent users: 4–6 CPU cores, 8–16 GB RAM
- A DNS name pointing to the server (e.g. `seb.example.org`)
- An SSL certificate (+ intermediate/CA certificate) for that DNS name
- A Moodle instance with administrator access

## Overview

The setup consists of three parts:

1. **Server & Docker** — run SEB Server, Screen Proctoring and MariaDB as Docker containers behind an nginx reverse proxy.
2. **Moodle** — install the [quizaccess_sebserver](https://github.com/ethz-let/moodle-quizaccess_sebserver) plugin and create a web service user the SEB Server connects with.
3. **SEB Server Web UI** — connect the LMS and create the configuration and exam templates used for exams.

Start with [1. Server Setup](docs/01-server-setup.md).
