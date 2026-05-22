# How to Install Hermes Agent on a VPS

Hermes Agent is useful when you want an AI assistant that is always available, not just open in a browser tab.

The most practical way to run it is on a small VPS.

That gives you an assistant you can message from Telegram, and it can also run scheduled tasks in the background. For example, it can check your websites, send you daily summaries, monitor a server, or prepare a weekly report.

This guide is written for people who are not server administrators. You will still need to copy and paste a few commands, but the goal is to keep the setup simple.

## What we are building

The setup looks like this:

- A small Ubuntu VPS runs Hermes Agent.
- Hermes connects to an AI model provider.
- You talk to Hermes through Telegram.
- Hermes can run scheduled automations.
- The VPS keeps running even when your laptop is closed.

For a first setup, I would not try to connect everything.

Start with Telegram and automations.

Once that works, add more later.

## What you need before starting

You need:

- A VPS with Ubuntu.
- SSH access to the VPS.
- An AI provider API key.
- A Telegram account.
- Around 30 to 60 minutes.

Recommended VPS:

- Ubuntu 24.04 LTS.
- 1 vCPU minimum.
- 2 GB RAM preferred.
- 20 GB disk.
- No GPU required.

You do not need a GPU because the AI model can run through a hosted provider.

The VPS runs Hermes. The provider runs the model.

## Important model note

Use a model with at least 64K context.

This matters.

If you choose a small model with a small context window, Hermes may not work properly. It is better to make the setup work with a capable hosted model first, then optimise cost later.

## Step 1, connect to the VPS

Your VPS provider will give you an IP address.

Open Terminal and run:

```bash
ssh root@YOUR_SERVER_IP
```

Example:

```bash
ssh root@123.123.123.123
```

If the server asks whether to continue, type:

```text
yes
```

## Step 2, update the server

Run:

```bash
apt update && apt upgrade -y
```

Then install a few basic tools:

```bash
apt install -y curl git ufw nano
```

Enable a basic firewall:

```bash
ufw allow OpenSSH
ufw enable
ufw status
```

If asked to continue, type:

```text
y
```

## Step 3, optional, create a normal user

This is cleaner than running everything as root.

```bash
adduser hermes
usermod -aG sudo hermes
su - hermes
```

You should now be using the `hermes` user.

Check:

```bash
whoami
```

It should show:

```text
hermes
```

## Step 4, install Hermes Agent

Run the official installer:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

When it finishes, reload your shell:

```bash
source ~/.bashrc
```

Check the command:

```bash
hermes --help
```

If you see help text, Hermes is installed.

## Step 5, choose the model provider

Run:

```bash
hermes model
```

Choose your provider and paste your API key when asked.

For a beginner setup, use a hosted provider.

Examples:

- Nous Portal.
- OpenRouter.
- Anthropic.
- OpenAI.
- Z.AI.
- DeepSeek.
- Hugging Face.

Pick a model with at least 64K context.

## Step 6, test Hermes

Run:

```bash
hermes
```

Send a simple message:

```text
Tell me if this Hermes setup is working.
```

Then try:

```text
Check this VPS and tell me the disk usage, memory usage, and uptime.
```

If Hermes replies properly, the base setup works.

Stop the session with:

```text
Ctrl+C
```

## Step 7, create a Telegram bot

Open Telegram and search for:

```text
@BotFather
```

Send:

```text
/newbot
```

Choose a display name, for example:

```text
My Hermes Assistant
```

Choose a username ending in `bot`, for example:

```text
my_hermes_helper_bot
```

BotFather will give you a bot token.

Save it somewhere private.

## Step 8, get your Telegram user ID

Hermes should only accept messages from users you allow.

Search Telegram for:

```text
@userinfobot
```

or:

```text
@get_id_bot
```

Start the bot and copy your numeric user ID.

It will look like this:

```text
123456789
```

## Step 9, configure Telegram in Hermes

On the VPS, run:

```bash
hermes gateway setup
```

Choose Telegram.

Paste:

- The Telegram bot token.
- Your numeric Telegram user ID.

Then test the gateway:

```bash
hermes gateway
```

Open Telegram and send your Hermes bot:

```text
Hello. Are you working?
```

If it replies, Telegram is working.

Stop the foreground gateway with:

```text
Ctrl+C
```

## Step 10, run Hermes in the background

Install the gateway as a service:

```bash
sudo hermes gateway install --system
```

Start it:

```bash
hermes gateway start
```

Check it:

```bash
hermes gateway status
```

Hermes should now stay available after you close the terminal.

## Step 11, create your first automation

From Telegram, send this to your Hermes bot:

```text
Every weekday at 8am, search for the latest important AI agent and open source LLM news from the last 24 hours. Summarise the top 3 stories, include links, keep it under 300 words, and deliver it here in Telegram.
```

Then check the job list on the VPS:

```bash
hermes cron list
```

If you see the job, the automation was created.

## Useful automation examples

### Website uptime check

```text
Every 10 minutes, check whether https://example.com is online. If the site returns HTTP 200 within 3 seconds, reply with [SILENT]. If the site is down, slow, or returns an error, send me a Telegram alert with the status code, response time, and likely cause.
```

### VPS health check

```text
Every 6 hours, check disk usage, memory usage, CPU load, uptime, and Docker container status on this VPS. If everything looks healthy, reply with [SILENT]. If anything looks wrong, send me a Telegram alert with the exact issue and suggested next action.
```

### Daily AI news briefing

```text
Every weekday at 8am, search the web for important AI agent, LLM, and automation news from the last 24 hours. Find at least 5 sources. Summarise the top 3 stories in plain English, include links, and deliver the result to Telegram. Keep it under 300 words.
```

### GitHub daily summary

```text
Every weekday at 9am, check the GitHub repository OWNER/REPO. Summarise new pull requests, merged pull requests, open issues, failed workflows, and anything that needs attention. Format it as a short engineering standup update.
```

### Competitor monitoring

```text
Every Monday at 9am, check these competitor websites: [add URLs]. Look for meaningful product changes, pricing changes, new features, new blog posts, and new landing page messaging. Ignore small visual changes. Send me a concise report with links.
```

## Managing automations

List jobs:

```bash
hermes cron list
```

Run a job manually:

```bash
hermes cron run JOB_ID
```

Pause a job:

```bash
hermes cron pause JOB_ID
```

Resume a job:

```bash
hermes cron resume JOB_ID
```

Delete a job:

```bash
hermes cron remove JOB_ID
```

## Troubleshooting

If Hermes does not start:

```bash
hermes --help
```

If the model does not work:

```bash
hermes model
```

If Telegram does not reply:

```bash
hermes gateway status
hermes logs
```

If automations do not run:

```bash
hermes cron list
hermes cron status
hermes gateway status
```

The gateway must be running for cron jobs to fire.

## Basic security rules

Keep it simple:

- Do not make the bot public.
- Do not share your Telegram bot token.
- Do not share model API keys.
- Keep command approval enabled.
- Do not expose the dashboard publicly.
- Start with read-only automations.
- Review old automations monthly.

A good first setup is private, small, and boring.

That is what you want for an agent that runs on a server.
