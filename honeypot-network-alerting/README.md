# 🍯 HoneyWatch — SSH/Telnet Honeypot with Real-Time Alerting

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![Docker](https://img.shields.io/badge/Docker-Compose-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

> ⚠️ **Ethical Use Notice**: This tool is intended for controlled lab environments and
> authorized security research only. Deploying honeypots on production networks without
> written authorization is illegal and unethical. The author assumes no liability.

## Overview

HoneyWatch is a low-interaction honeypot that simulates SSH and Telnet services to
capture attacker behaviors. It logs commands, payloads, and origin IPs, then delivers
real-time alerts via Slack/Discord and visualizes attack patterns on a live dashboard.

## Tech Stack

- Python 3.10+, Cowrie (SSH/Telnet honeypot)
- Redis or SQLite for event storage
- Grafana or a custom Flask dashboard for visualization
- Slack/Discord webhooks or SMTP for alerting
- Docker + Docker Compose for deployment

## Features

- 🪤 Simulates SSH (port 22), Telnet (port 23), and HTTP (port 80) services
- 📊 Live Grafana/Flask dashboard with attack frequency and GeoIP heat maps
- 🔔 Real-time Slack/Discord/email alerting on new connections
- 🌐 Automatic IP reputation lookup via AbuseIPDB API
- 🗃️ SQLite logging with exportable JSON/CSV reports
- 🐳 Fully containerized via Docker Compose

## Architecture

[Internet] ──► [Cowrie Honeypot Container]
                       │
                       ▼
              [SQLite Event Store] ──► [Flask API] ──► [Dashboard]
                       │
                       ▼
              [Alert Engine] ──► [Slack / Email / Webhook]

## Configuration

Edit `.env` to customize:

| Variable              | Description                        | Default     |
|-----------------------|------------------------------------|-------------|
| `HONEYPOT_SSH_PORT`   | Fake SSH port to expose            | `2222`      |
| `SLACK_WEBHOOK_URL`   | Slack incoming webhook URL         | _(optional)_|
| `ABUSEIPDB_API_KEY`   | AbuseIPDB API key for IP lookup    | _(optional)_|
| `ALERT_THRESHOLD`     | Min connections before alerting    | `1`         |

## Screenshots

>

## MITRE ATT&CK Coverage

This honeypot captures behaviors related to:
- T1110 — Brute Force
- T1021 — Remote Services
- T1059 — Command and Scripting Interpreter

## 🤝 Contributing

Contributions are welcome. Feel free to open an issue or submit a pull request to improve this project.

## 📄 License

This project is licensed under the MIT License.
