# 08, Security Notes

Hermes can be powerful because it can use tools and run commands.

That also means you should set it up carefully.

This does not need to be complicated. A few basic rules avoid most problems.

## Rule 1, Do not make the bot public

Only allow your own Telegram user ID, or the user IDs of people you trust.

Do not enable access for everyone unless you fully understand the risk.

A public agent with server access is not a normal chatbot. It can become a security problem.

## Rule 2, Keep your bot token private

Your Telegram bot token is a secret.

Do not put it in:

- Blog screenshots.
- GitHub repositories.
- Public support tickets.
- Shared documents.
- YouTube videos.
- Slack messages with large groups.

If the token leaks, revoke it in BotFather and create a new one.

## Rule 3, Keep API keys private

The same applies to model provider API keys.

If an API key leaks, someone else may use your account and spend your credits.

Store keys only inside the Hermes configuration or a secure password manager.

## Rule 4, Keep command approval enabled

Hermes can ask before running dangerous commands.

For non-technical users, keep manual approval enabled.

Avoid fully automatic command execution unless the server is disposable or heavily locked down.

A safe default is:

```yaml
approvals:
  mode: manual
```

Avoid this unless you know exactly why you are doing it:

```yaml
approvals:
  mode: off
```

## Rule 5, Do not expose the dashboard publicly

Hermes has a local dashboard.

That does not mean it should be open to the internet.

A safe way to run it is locally:

```bash
hermes dashboard
```

Avoid binding it to a public IP without proper protection.

If you need remote access, use one of these:

- SSH tunnel.
- VPN.
- Reverse proxy with authentication.
- Firewall rules allowing only your IP.

## Rule 6, Use a separate VPS if possible

If Hermes is experimental, run it on its own VPS.

Do not install it directly on your production eCommerce server unless you have a clear reason.

A separate VPS is cleaner because:

- Mistakes are isolated.
- Logs are easier to inspect.
- Permissions are simpler.
- You can rebuild the server if needed.
- You avoid mixing agent experiments with production workloads.

## Rule 7, Start with read-only automations

Your first automations should read and report.

Examples:

- Check uptime.
- Summarise news.
- Check server health.
- Report GitHub activity.
- Watch public pages.

Avoid early automations that:

- Delete files.
- Restart production services.
- Change DNS records.
- Modify customer data.
- Send customer emails automatically.
- Make purchases.
- Deploy code.

Add write actions later, after you trust the setup.

## Rule 8, Use clear alert rules

A bad monitoring job sends too many messages.

A good monitoring job only sends messages when action is needed.

Use this pattern:

```text
If everything is healthy, reply with [SILENT]. Only alert me if something needs attention.
```

## Rule 9, Review automations monthly

Every month, check:

```bash
hermes cron list
```

Remove automations you no longer need.

Old automations can become noisy, expensive, or misleading.

## Rule 10, Keep the server updated

Run this occasionally:

```bash
sudo apt update && sudo apt upgrade -y
```

Then reboot if needed:

```bash
sudo reboot
```

After reboot, check:

```bash
hermes gateway status
```

## Simple security checklist

Before you call the setup finished, check:

- Firewall is enabled.
- SSH works.
- Telegram bot is private.
- Only trusted Telegram user IDs are allowed.
- Gateway runs as a service.
- Dangerous commands require approval.
- Dashboard is not public.
- API keys are not in public files.
- Automations are specific and limited.
