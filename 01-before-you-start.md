# 01, Before You Start

Before installing Hermes Agent, it helps to understand what you are building.

You are not installing "ChatGPT on a server".

You are installing an agent that lives on a VPS, connects to an AI model provider, and can run tools, scheduled jobs, and messaging integrations.

The simplest useful setup looks like this:

- You send a message in Telegram.
- Telegram sends that message to Hermes running on your VPS.
- Hermes uses your selected AI model to understand the request.
- Hermes can run tools, check websites, read files, run safe terminal commands, or create scheduled jobs.
- Hermes sends the answer back to Telegram.

## What you need

You need:

- A VPS.
- SSH access to the VPS.
- Ubuntu Linux on the VPS.
- A model provider account and API key.
- A Telegram account.
- Around 30 to 60 minutes for the first setup.

## Recommended VPS size

For a basic Hermes setup, use:

- 1 vCPU minimum.
- 2 GB RAM preferred.
- 20 GB disk minimum.
- Ubuntu 24.04 LTS.

A smaller server may work, but 2 GB RAM gives you more breathing room.

You do not need a GPU if you use a hosted AI provider.

## Choose a model provider first

Hermes needs to connect to a model provider.

Examples:

- Nous Portal
- OpenRouter
- Anthropic
- OpenAI
- Z.AI
- DeepSeek
- Hugging Face
- GitHub Copilot
- Vercel AI Gateway
- Any compatible OpenAI-style endpoint, if supported by your setup

For a beginner, OpenRouter is often a practical choice because it gives access to many models through one API key.

For engineering and coding work, Anthropic, OpenAI, Z.AI, DeepSeek, and similar providers are also common choices.

## Important model requirement

Hermes requires a model with a large context window.

Use a model with at least 64K context.

If you pick a small model with a small context window, Hermes may fail to start properly or may behave badly with automations.

This is one of the most common early mistakes.

## What to keep ready

Before you start, keep these ready in a notes file:

- VPS IP address.
- VPS username.
- VPS password or SSH key.
- AI provider API key.
- Telegram bot token, you will create this later.
- Your Telegram numeric user ID, you will get this later.

Do not publish these values. Do not paste them into screenshots.

## A sensible first target

Do not start by trying to connect every channel.

Start with this:

- Install Hermes.
- Connect one model provider.
- Start Hermes from the command line.
- Connect Telegram.
- Create one simple automation.

Once this works, expand slowly.
