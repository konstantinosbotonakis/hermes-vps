# Hermes Agent on a VPS

This is a practical guide for installing Hermes Agent on a VPS and using it for simple automations.

It is written for people who are comfortable copying commands, but do not want a deep Linux or DevOps explanation.

The setup in this guide is intentionally simple:

- One Ubuntu VPS
- One hosted AI model provider
- Hermes Agent installed on the VPS
- Telegram as the main interface
- Scheduled automations running in the background

## Files in this guide

Read them in this order:

1. [01-before-you-start.md](01-before-you-start.md)
2. [02-create-and-connect-to-the-vps.md](02-create-and-connect-to-the-vps.md)
3. [03-install-hermes-agent.md](03-install-hermes-agent.md)
4. [04-connect-telegram.md](04-connect-telegram.md)
5. [05-create-automations.md](05-create-automations.md)
6. [06-use-cases.md](06-use-cases.md)
7. [07-troubleshooting.md](07-troubleshooting.md)
8. [08-security-notes.md](08-security-notes.md)

There is also a longer publishing version:

- [article-version.md](article-version.md)

## What Hermes Agent is useful for

Hermes Agent is useful when you want an AI assistant that is always available, not tied to your laptop, and able to run scheduled jobs.

A few examples:

- Send you a daily AI news summary.
- Check if your websites are online.
- Monitor a VPS and warn you if disk or memory usage is high.
- Summarise GitHub activity every morning.
- Watch competitor websites for important changes.
- Create a weekly business report.
- Answer messages from Telegram while running on your VPS.

## What this guide does not cover

This guide does not try to explain every Hermes feature.

It focuses on the setup most people should start with:

- VPS
- Hermes
- Telegram
- Cron automations

Once that works, you can add more advanced channels, more tools, or custom integrations later.

## Official Hermes links

- Website: https://hermes-agent.nousresearch.com/
- Documentation: https://hermes-agent.nousresearch.com/docs/
- Installation: https://hermes-agent.nousresearch.com/docs/getting-started/installation
- Quickstart: https://hermes-agent.nousresearch.com/docs/getting-started/quickstart
- Telegram guide: https://hermes-agent.nousresearch.com/docs/user-guide/messaging/telegram
- Cron automations: https://hermes-agent.nousresearch.com/docs/user-guide/features/cron

## Recommended starting setup

For most users, I would start with this:

- Ubuntu 24.04 LTS VPS
- 1 vCPU
- 2 GB RAM
- 20 GB disk
- Telegram bot access
- Hosted model provider, not a local model

You do not need a GPU for this setup.

The VPS runs Hermes. The AI model runs through a provider API.
