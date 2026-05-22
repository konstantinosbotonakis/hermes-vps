# 07, Troubleshooting

This page covers the problems most people hit first.

## Problem, `hermes` command not found

Try:

```bash
source ~/.bashrc
```

Then:

```bash
hermes --help
```

If it still does not work, disconnect and reconnect to the VPS.

```bash
exit
ssh hermes@YOUR_SERVER_IP
```

If you installed as `root`, connect as `root`.

## Problem, Hermes starts but the model does not reply

Run:

```bash
hermes model
```

Check:

- The provider is correct.
- The API key is correct.
- The selected model still exists.
- The selected model has enough context.
- Your provider account has credits or billing enabled.

A model problem is usually not a VPS problem.

## Problem, Telegram bot does not reply

Run:

```bash
hermes gateway status
```

If it is not running:

```bash
hermes gateway start
```

Then check logs:

```bash
hermes logs
```

Common causes:

- Wrong Telegram bot token.
- Wrong numeric Telegram user ID.
- Gateway not installed.
- Gateway not running.
- Model provider not configured.
- Model provider API key is invalid.

## Problem, Telegram replies to nobody

This is usually an allowlist issue.

Run:

```bash
hermes gateway setup
```

Check that your numeric Telegram user ID is allowed.

Do not rely on your Telegram username.

Hermes needs the numeric user ID.

## Problem, automation did not run

Check the cron list:

```bash
hermes cron list
```

Check cron status:

```bash
hermes cron status
```

Check gateway status:

```bash
hermes gateway status
```

The gateway must be running for scheduled automations.

If the gateway is stopped, cron jobs will not fire.

## Problem, automation runs but gives poor output

Rewrite the automation prompt.

Poor automation prompt:

```text
Check my sites.
```

Better automation prompt:

```text
Every 30 minutes, check https://example.com and https://support.example.com. If both sites return HTTP 200 within 3 seconds, reply with [SILENT]. If either site is down, slow, or returns an error, send me a Telegram alert with the URL, status code, response time, and likely cause.
```

The automation should include:

- Exact schedule.
- Exact URLs or sources.
- Exact success condition.
- Exact failure condition.
- Exact output format.

## Problem, too many messages

Use `[SILENT]`.

Example:

```text
If everything is healthy, reply with [SILENT]. Only send me a Telegram message if something needs attention.
```

This is useful for monitoring jobs.

## Problem, too expensive

You have a few options:

- Use a cheaper hosted model for simple reports.
- Use no-LLM script automations for simple checks.
- Run less frequent jobs.
- Avoid asking every automation to search the web.
- Keep prompts short and specific.
- Use `[SILENT]` for normal results.

Do not optimise cost before the setup works.

Make it reliable first. Then reduce cost.

## Problem, server rebooted and Hermes stopped

Run:

```bash
hermes gateway status
```

If stopped:

```bash
hermes gateway start
```

If it keeps happening, reinstall the gateway service:

```bash
sudo hermes gateway install --system
hermes gateway start
```

## Problem, I need to see what happened

Run:

```bash
hermes logs
```

If that is not enough, check the Hermes log directory:

```bash
ls -lah ~/.hermes/logs/
```

Then inspect the latest logs:

```bash
tail -n 100 ~/.hermes/logs/agent.log
tail -n 100 ~/.hermes/logs/errors.log
```

## Quick recovery checklist

When something breaks, run these in order:

```bash
hermes --help
hermes model
hermes gateway status
hermes cron list
hermes cron status
hermes logs
```

This usually tells you where the issue is:

- Installation problem.
- Model problem.
- Telegram gateway problem.
- Cron problem.
- Runtime error.
