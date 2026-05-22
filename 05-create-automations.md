# 05, Create Automations

Automations are where Hermes becomes useful.

Instead of asking the same question every day, you ask Hermes to create a scheduled job.

For example:

- Check my websites every 10 minutes.
- Send me an AI news summary every morning.
- Check this VPS every 6 hours.
- Summarise GitHub activity every weekday.
- Tell me only if something is wrong.

## Important idea

A scheduled automation must explain the full task.

Bad prompt:

```text
Do my usual check every morning.
```

Better prompt:

```text
Every morning at 8am, check whether https://example.com is online. If it is working, reply with [SILENT]. If it is down, slow, or returning errors, send me a Telegram alert with the status code, response time, and suggested next step.
```

Hermes cron jobs run later in a fresh context.

Do not assume Hermes remembers what "usual" means.

## Step 1, Check that the gateway is running

Run:

```bash
hermes gateway status
```

Cron jobs are handled by the gateway.

If the gateway is not running, start it:

```bash
hermes gateway start
```

## Step 2, Create an automation from Telegram

Message your Telegram bot:

```text
Every weekday at 8am, search for the latest important AI agent and open source LLM news from the last 24 hours. Summarise the top 3 stories, include links, keep it under 300 words, and deliver it here in Telegram.
```

Hermes should understand that this is a scheduled task.

## Step 3, List automations

On the VPS, run:

```bash
hermes cron list
```

You should see your job in the list.

## Step 4, Run an automation manually

If you want to test a job without waiting for the schedule:

```bash
hermes cron run JOB_ID
```

Replace `JOB_ID` with the job ID from the list.

## Step 5, Pause an automation

```bash
hermes cron pause JOB_ID
```

## Step 6, Resume an automation

```bash
hermes cron resume JOB_ID
```

## Step 7, Delete an automation

```bash
hermes cron remove JOB_ID
```

## Useful automation pattern

Use this pattern when writing automations:

```text
Every [schedule], do [specific task]. Use [sources or commands]. If everything is normal, reply with [SILENT]. If something needs attention, send me a Telegram message with [exact details].
```

Example:

```text
Every 30 minutes, check https://example.com. If the site returns HTTP 200 within 3 seconds, reply with [SILENT]. If not, send me a Telegram alert with the status code, response time, and likely reason.
```

## Keep automations short and clear

A good automation has:

- A clear schedule.
- A clear source.
- A clear action.
- A clear output format.
- A clear condition for alerting.

Avoid asking one automation to do too much.

Instead of one huge job, create smaller jobs:

- Website uptime monitor.
- Weekly competitor report.
- Daily AI news summary.
- VPS health alert.
- GitHub pull request summary.

Small automations are easier to test and easier to fix.
